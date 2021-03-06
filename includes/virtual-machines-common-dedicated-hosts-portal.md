---
title: インクルード ファイル
description: インクルード ファイル
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 07/25/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 262880997c6b065dc5293a18d9a07c52ac836f37
ms.sourcegitcommit: f4d8f4e48c49bd3bc15ee7e5a77bee3164a5ae1b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/04/2019
ms.locfileid: "73591050"
---
> [!IMPORTANT]
> 現在、専用ホストはパブリック プレビュー段階にあります。
> このプレビュー バージョンはサービス レベル アグリーメントなしで提供されています。運用環境のワークロードに使用することはお勧めできません。 特定の機能はサポート対象ではなく、機能が制限されることがあります。 詳しくは、[Microsoft Azure プレビューの追加使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)に関するページをご覧ください。
>
> **プレビューに関する既知の制限事項**
> - 仮想マシン スケール セットは、現在、専用ホストではサポートされていません。
> - プレビューの初期リリースでは、次の VM シリーズがサポートされています: DSv3 と ESv3。 


## <a name="create-a-host-group"></a>ホスト グループを作成する

**ホスト グループ**は、専用ホストのコレクションを表す新しいリソースです。 リージョンと可用性ゾーンにホスト グループを作成し、それにホストを追加します。 高可用性を計画する場合は、追加のオプションがあります。 専用ホストでは、次のいずれかまたは両方のオプションを使用できます。 
- 複数の可用性ゾーンにまたがります。 この場合は、使用する各ゾーンにホスト グループを用意する必要があります。
- 物理ラックにマップされる複数の障害ドメインにまたがります。 
 
どちらの場合も、ホスト グループに対して障害ドメイン数を指定する必要があります。 グループ内で障害ドメインをまたがりたくない場合は、障害ドメインの数を 1 にします。 

可用性ゾーンと障害ドメインの両方を使用することもできます。 

この例では、1 つの可用性ゾーンと 2 つの障害ドメインを使用してホスト グループを作成します。 


1. Azure [portal](https://portal.azure.com) を開きます。
1. 左上隅にある **[リソースの作成]** を選択します。
1. **[ホストグループ]** を検索し、結果から **[ホスト グループ (プレビュー)]** を選択します。

    ![ホスト グループの検索結果。](./media/virtual-machines-common-dedicated-hosts-portal/host-group.png)
1. **[ホスト グループ (プレビュー)]** ページで、 **[作成]** を選択します。
1. 使用するサブスクリプションを選択し、 **[新規作成]** を選択して新しいリソース グループを作成します。
1. **[名前]** として「*myDedicatedHostsRG*」と入力し、 **[OK]** を選択します。
1. **[Host group name]\(ホスト グループ名\)** に「*myHostGroup*」と入力します。
1. **[場所]** で、 **[米国東部]** を選択します。
1. **[可用性ゾーン]** で、 **[1]** を選択します。
1. **[Fault domain count]\(障害ドメインの数\)** で、 **[2]** を選択します。
1. **[確認および作成]** を選択して、検証が行われるのを待ちます。

    ![ホスト グループの設定](./media/virtual-machines-common-dedicated-hosts-portal/host-group-settings.png)
1. "**検証に成功しました**" というメッセージが表示されたら、 **[作成]** を選択してホスト グループを作成します。

ホスト グループの作成には少しだけ時間がかかります。

## <a name="create-a-dedicated-host"></a>専用ホストを作成する

次に、ホスト グループに専用ホストを作成します。 ホストの名前に加えて、ホストの SKU を指定する必要があります。 ホスト SKU では、専用ホストに対してサポートされている VM シリーズとハードウェアの世代がキャプチャされます。  プレビュー期間中は、次のホスト SKU 値がサポートされます: DSv3_Type1 と ESv3_Type1。

ホスト SKU の詳細と価格については、「[Azure 専用ホストの価格](https://aka.ms/ADHPricing)」を参照してください。

ホスト グループの障害ドメイン数を設定した場合は、ホストの障害ドメインを指定するように求められます。  

1. 左上隅にある **[リソースの作成]** を選択します。
1. "**専用ホスト**" を検索し、結果から **[Dedicated hosts (preview)]\(専用ホスト (プレビュー)\)** を選択します。

    ![ホスト グループの検索結果。](./media/virtual-machines-common-dedicated-hosts-portal/host.png)
1. **[Dedicated hosts (preview)]\(専用ホスト (プレビュー)\)** ページで、 **[作成]** を選択します。
1. 使用するサブスクリプションを選択します。
1. *[リソース グループ]* として **myDedicatedHostsRG** を選択します。
1. **[インスタンスの詳細]** で、 **[名前]** に「*myHost*」と入力し、場所として *[米国東部]* を選択します。
1. **[ハードウェア プロファイル]** で、 **[Size family]\(サイズ ファミリ\)** として *[Standard Es3 family - Type 1]\(Standard Es3 ファミリ - Type 1\)* を選択し、 **[ホスト グループ]** として *myHostGrup* を選択し、 **[障害ドメイン]** として *[1]* を選択します。 他のフィールドについては既定値を使用します。
1. 終わったら、 **[確認および作成]** を選択して、検証が行われるのを待ちます。

    ![ホストの設定](./media/virtual-machines-common-dedicated-hosts-portal/host-settings.png)
1. "**検証に成功しました**" というメッセージが表示されたら、 **[作成]** を選択してホストを作成します。


