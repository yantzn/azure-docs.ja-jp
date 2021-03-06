---
title: チュートリアル - Azure VM での SAP HANA データベースのバックアップ
description: このチュートリアルでは、Azure VM 上で稼働している SAP HANA データベースを Azure Backup Recovery Services コンテナーにバックアップする方法について学習します。
ms.topic: tutorial
ms.date: 11/12/2019
ms.openlocfilehash: a622370fca3144aeb6a5d7c071c227b3c21cf135
ms.sourcegitcommit: e50a39eb97a0b52ce35fd7b1cf16c7a9091d5a2a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/21/2019
ms.locfileid: "74287189"
---
# <a name="tutorial-back-up-sap-hana-databases-in-an-azure-vm"></a>チュートリアル:Azure VM での SAP HANA データベースのバックアップ

このチュートリアルでは、Azure VM 上で稼働している SAP HANA データベースを Azure Backup Recovery Services コンテナーにバックアップする方法について説明します。 この記事では、以下を行う方法について説明します。

> [!div class="checklist"]
>
> * コンテナーを作成して構成する
> * データベースを検出する
> * バックアップを構成する

現在サポートされているすべてのシナリオについては、[こちら](sap-hana-backup-support-matrix.md#scenario-support)を参照してください。

## <a name="onboard-to-the-public-preview"></a>パブリック プレビューにオンボードする

以下のように、パブリック プレビューにオンボードします。

* ポータルで、[この記事に従って](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-register-provider-errors#solution-3---azure-portal)、Recovery Services サービス プロバイダーにご利用のサブスクリプション ID を登録します。

* PowerShell では、このコマンドレットを実行します。 "登録済み" として完了するはずです。

    ```powershell
    Register-AzProviderFeature -FeatureName "HanaBackup" –ProviderNamespace Microsoft.RecoveryServices
    ```

## <a name="prerequisites"></a>前提条件

バックアップを構成する前に、必ず、次の操作を行ってください。

1. SAP HANA データベースを実行している VM に、次のように zypper を使用して、公式の SLES パッケージ/メディアから ODBC ドライバー パッケージをインストールして有効にします。

```bash
sudo zypper update
sudo zypper install unixODBC
```

>[!NOTE]
> リポジトリを更新していない場合は、unixODBC のバージョンが 2.3.4 以降であることを確認してください。 unixODBC のバージョンを確認するには、root として `odbcinst -j` を実行します。
>

2. [以下の手順](#set-up-network-connectivity)で説明されているように、Azure に到達できるように、VM からインターネットへの接続を許可します。

3. HANA がルート ユーザーとしてインストールされている仮想マシンで事前登録スクリプトを実行します。 [このスクリプト](https://aka.ms/scriptforpermsonhana)を使用すると、[適切な権限](#setting-up-permissions)が設定されます。

## <a name="set-up-network-connectivity"></a>ネットワーク接続を設定する

すべての操作において、SAP HANA VM では、Azure パブリック IP アドレスへの接続を必要とします。 接続がないと、VM の操作 (データベースの検出、バックアップの構成、バックアップのスケジュール、復旧ポイントの復元など) を行うことはできません。 Azure データセンターの IP 範囲へのアクセスを許可することで、接続を確立します。

* Azure データセンターの [IP アドレス範囲](https://www.microsoft.com/download/details.aspx?id=41653)をダウンロードしてから、これらの IP アドレスへのアクセスを許可できます。
* ネットワーク セキュリティ グループ (NSG) を使用している場合は、Azure Cloud の[サービス タグ](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)を使用して、Azure パブリック IP アドレスをすべて許可することができます。 [Set-AzureNetworkSecurityRule コマンドレット](https://docs.microsoft.com/powershell/module/servicemanagement/azure/set-azurenetworksecurityrule?view=azuresmps-4.0.0)を使用して、NSG ルールを変更することができます。
* 転送は HTTPS 経由で行われるため、ポート 443 を許可リストに追加する必要があります。

## <a name="setting-up-permissions"></a>アクセス許可の設定

事前登録スクリプトは、次のアクションを実行します。

1. HANA システム内に AZUREWLBACKUPHANAUSER を作成し、次の必要なロールとアクセス許可を追加します。
   * DATABASE ADMIN: 復元中に新しい DB を作成します。
   * CATALOG READ: バックアップ カタログを読み取ります。
   * SAP_INTERNAL_HANA_SUPPORT いくつかのプライベート テーブルにアクセスします。
2. すべての操作 (データベース クエリ、復元操作、バックアップの構成および実行) を処理するためのキーを HANA プラグインの Hdbuserstore に追加します。

キーの作成を確認するには、SIDADM 資格情報を使用して HANA コンピューター上で HDBSQL コマンドを実行します。

```hdbsql
hdbuserstore list
```

このコマンド出力には、AZUREWLBACKUPHANAUSER として示されるユーザーと共に {SID}{DBNAME} キーが表示されます。

>[!NOTE]
> /usr/sap/{SID}/home/.hdb/ の下に固有の一連の SSFS ファイルがあることを確認してください。 このパスにはフォルダーが 1 つしか存在しません。
>

## <a name="create-a-recovery-service-vault"></a>Recovery Services コンテナーを作成する

Recovery Services コンテナーは、経時的に作成されたバックアップと復旧ポイントを格納するエンティティです。 Recovery Services コンテナーには、保護される仮想マシンに関連付けられたバックアップ ポリシーも含まれます。

Recovery Services コンテナーを作成するには、次の手順に従います。

1. [Azure Portal](https://portal.azure.com/) でサブスクリプションにサインインします。

2. 左側のメニューから、 **[すべてのサービス]** を選択します。

![[すべてのサービス] を選択する](./media/tutorial-backup-sap-hana-db/all-services.png)

3. **[すべてのサービス]** ダイアログ ボックスに、「**Recovery Services**」と入力します。 入力に従って、リソースの一覧がフィルター処理されます。 リソースの一覧から **[Recovery Services コンテナー]** を選択します。

![Recovery Services コンテナーを選択する](./media/tutorial-backup-sap-hana-db/recovery-services-vaults.png)

4. **[Recovery Services コンテナー]** ダッシュボードで **[追加]** を選択します。

![Recovery Services コンテナーを追加する](./media/tutorial-backup-sap-hana-db/add-vault.png)

**[Recovery Services コンテナー]** ダイアログ ボックスが開きます。 **[名前]、[サブスクリプション]、[リソース グループ]** 、および **[場所]** に値を入力します。

![Recovery Services コンテナーの作成](./media/tutorial-backup-sap-hana-db/create-vault.png)

* **Name**:この名前は、Recovery Services コンテナーを識別するために使用されるため、Azure サブスクリプションに対して一意である必要があります。 2 文字以上で、50 文字以下の名前を指定します。 名前の先頭にはアルファベットを使用する必要があります。また、名前に使用できるのはアルファベット、数字、ハイフンのみです。 このチュートリアルでは、**SAPHanaVault** という名前を使用しています。
* **サブスクリプション**:使用するサブスクリプションを選択します。 1 つのサブスクリプションのみのメンバーの場合は、その名前が表示されます。 どのサブスクリプションを使用すればよいかがわからない場合は、既定 (推奨) のサブスクリプションを使用してください。 職場または学校アカウントが複数の Azure サブスクリプションに関連付けられている場合に限り、複数の選択肢が存在します。 ここでは、**SAP HANA solution lab subscription** サブスクリプションを使用しています。
* **[リソース グループ]** :既存のリソース グループを使用するか、新しいリソース グループを作成します。 ここでは、**SAPHANADemo** を使用しています。<br>
サブスクリプションの使用可能なリソース グループの一覧を表示するには、 **[既存のものを使用]** を選択し、ドロップダウン リスト ボックスからリソースを選択します。 新しいリソース グループを作成するには、 **[新規作成]** を選択し、名前を入力します。 リソース グループの詳細については、「[Azure Resource Manager の概要](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)」を参照してください。
* **[場所]** :コンテナーの地理的リージョンを選択します。 このコンテナーは、SAP HANA を実行している仮想マシンと同じリージョンにある必要があります。 ここでは、 **[米国東部 2]** を使用しています。

5. **[確認および作成]** を選択します。

   ![[確認および作成] を選択する](./media/tutorial-backup-sap-hana-db/review-create.png)

これで、Recovery Services コンテナーが作成されました。

## <a name="discover-the-databases"></a>データベースを検出する

1. コンテナーの **[作業の開始]** で、 **[バックアップ]** をクリックします。 **[ワークロードはどこで実行されていますか?]** で、 **[Azure VM の SAP HANA]** を選択します。
2. **[検出の開始]** をクリックします。 これで、コンテナー リージョン内の保護されていない Linux VM の検出が開始されます。 保護する Azure VM が表示されます。
3. **[仮想マシンの選択]** で、リンクをクリックして、データベースの検出のために SAP HANA VM へのアクセス許可を Azure Backup サービスに提供するスクリプトをダウンロードします。
4. バックアップする SAP HANA データベースをホストしている VM 上でスクリプトを実行します。
5. VM 上でスクリプトを実行した後、 **[仮想マシンの選択]** で VM を選択します。 その後、 **[DB の検出]** をクリックします。
6. Azure Backup によって、VM 上のすべての SAP HANA データベースが検出されます。 検出中に、Azure Backup によって VM がコンテナーに登録され、VM に拡張機能がインストールされます。 エージェントはデータベースにインストールされません。

   ![データベースを検出する](./media/tutorial-backup-sap-hana-db/database-discovery.png)

## <a name="configure-backup"></a>バックアップの構成

バックアップするデータベースが検出されたら、バックアップを有効にしましょう。

1. **[バックアップの構成]** をクリックします。

![バックアップの構成](./media/tutorial-backup-sap-hana-db/configure-backup.png)

2. **[バックアップする項目の選択]** で、保護するデータベースを 1 つ以上選択し、 **[OK]** をクリックします。

![バックアップする項目を選択する](./media/tutorial-backup-sap-hana-db/select-items-to-backup.png)

3. **[バックアップ ポリシー] > [バックアップ ポリシーの選択]** で、次のセクションの手順に従って、データベースの新しいバックアップ ポリシーを作成します。

![バックアップ ポリシーを選択する](./media/tutorial-backup-sap-hana-db/backup-policy.png)

4. ポリシーを作成した後、 **[バックアップ] メニュー**の **[バックアップの有効化]** をクリックします。

   ![[バックアップの有効化] をクリックする](./media/tutorial-backup-sap-hana-db/enable-backup.png)

5. ポータルの **[通知]** 領域で、バックアップ構成の進行状況を追跡します。

## <a name="creating-a-backup-policy"></a>バックアップ ポリシーの作成

バックアップ ポリシーでは、バックアップが取得されるタイミングと、それらが保持される期間を定義します。

* ポリシーはコンテナー レベルで作成されます。
* 複数のコンテナーでは同じバックアップ ポリシーを使用できますが、各コンテナーにバックアップ ポリシーを適用する必要があります。

次のように、ポリシー設定を指定します。

1. **[ポリシー名]** に新しいポリシーの名前を入力します。 この場合は、「**SAPHANA**」と入力します。

![新しいポリシーの名前を入力する](./media/tutorial-backup-sap-hana-db/new-policy.png)

2. **[Full Backup policy]\(完全バックアップのポリシー\)** で、 **[バックアップ頻度]** を選択します。 **[Daily]\(毎日\)** または **[毎週]** を選択できます。 このチュートリアルでは、 **[Daily]\(毎日\)** バックアップを選択しています。

![バックアップ頻度を選択する](./media/tutorial-backup-sap-hana-db/backup-frequency.png)

3. **[リテンション期間]** で、完全バックアップのリテンション期間の設定を構成します。
   * 既定では、すべてのオプションが選択されています。 使用しない保持期間の制限をすべてクリアして、使用するものを設定します。
   * あらゆる種類のバックアップ (完全、差分、ログ) の最小保持期間は 7 日間です。
   * 復旧ポイントは、そのリテンション期間の範囲に基づいて、リテンション期間に対してタグ付けされます。 たとえば、日次での完全バックアップを選択した場合、日ごとにトリガーされる完全バックアップは 1 回だけです。
   * 特定の曜日のバックアップがタグ付けされ、週次でのリテンション期間と設定に基づいて保持されます。
   * 月次および年次のリテンション期間の範囲でも、同様の動作になります。
4. **[完全バックアップのポリシー]** メニューで、 **[OK]** をクリックして設定を受け入れます。
5. 次に、 **[差分バックアップ]** を選択して、差分ポリシーを追加します。
6. **差分バックアップのポリシー**で、 **[有効]** を選択して頻度とリテンション期間の制御を開きます。 ここでは、毎週**日曜日**の**午前 2:00** に実行され、**30 日間**保持される差分バックアップを有効にしました。

   ![差分バックアップのポリシー](./media/tutorial-backup-sap-hana-db/differential-backup-policy.png)

>[!NOTE]
>増分バックアップは現在、サポートされていません。
>

7. **[OK]** をクリックしてポリシーを保存し、 **[バックアップ ポリシー]** のメイン メニューに戻ります。
8. **[ログ バックアップ]** を選択し、トランザクション ログ バックアップ ポリシーを追加します。
   * **[ログ バックアップ]** は既定で **[有効]** に設定されています。 SAP HANA によってすべてのログ バックアップが管理されるため、これを無効にすることはできません。
   * ここでは、バックアップのスケジュールとして **2 時間**を設定し、保持期間として **15 日間**を設定しています。

    ![ログ バックアップのポリシー](./media/tutorial-backup-sap-hana-db/log-backup-policy.png)

>[!NOTE]
> ログ バックアップでは、1 回の完全バックアップが正常に完了した後にのみ、フローが開始されます。
>

9. **[OK]** をクリックしてポリシーを保存し、 **[バックアップ ポリシー]** のメイン メニューに戻ります。
10. バックアップ ポリシーの定義が完了した後、 **[OK]** をクリックします。

これで、SAP HANA データベースのバックアップが正常に構成されました。

## <a name="next-steps"></a>次の手順

* [Azure VM で稼働している SAP HANA データベースでオンデマンド バックアップを実行する](backup-azure-sap-hana-database.md#run-an-on-demand-backup)方法を学習する
* [Azure VM で稼働している SAP HANA データベースを復元する](sap-hana-db-restore.md)方法を学習する
* [Azure Backup を使用してバックアップされている SAP HANA データベースを管理する](sap-hana-db-manage.md)方法を学習する
* [SAP HANA データベースのバックアップ時に発生する一般的な問題のトラブルシューティング](backup-azure-sap-hana-database-troubleshoot.md)方法を学習する
