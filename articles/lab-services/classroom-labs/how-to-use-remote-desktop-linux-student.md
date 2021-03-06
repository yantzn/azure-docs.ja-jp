---
title: Azure Lab Services で Linux 用リモート デスクトップを使用する | Microsoft Docs
description: Azure Lab Services のラボで Linux 仮想マシン用リモート デスクトップを使用する方法について学習します。
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/20/2019
ms.author: spelluru
ms.openlocfilehash: 0f45af2730b05998fc82212c63778c89bb16b6ef
ms.sourcegitcommit: f4d8f4e48c49bd3bc15ee7e5a77bee3164a5ae1b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/04/2019
ms.locfileid: "73585093"
---
# <a name="use-remote-desktop-for-linux-virtual-machines-in-a-classroom-lab-of-azure-lab-services"></a>Azure Lab Services のクラスルーム ラボで Linux 仮想マシン用リモート デスクトップを使用する
この記事では、学生がラボで RDP または SSH を使用して Linux 仮想マシン (VM) に接続する方法を示します。 

インストラクターは、学生がクラスルーム ラボの VM に接続する前に、リモート デスクトップ接続機能を有効にする必要があります。 インストラクターがリモート デスクトップ接続機能を有効にする方法については、[Linux 仮想マシン用リモート デスクトップの有効化](how-to-enable-remote-desktop-linux.md)に関するページを参照してください。

> [!IMPORTANT] 
> **リモート デスクトップ接続**を有効にすると、Linux マシンの **RDP** ポートのみが開きます。 インストラクターは、最初に SSH を使用して Linux マシンに接続し、RDP と GUI のパッケージをインストールします。これで、後で RDP を使用して Linux マシンに接続できるようになります。 

## <a name="connect-to-the-student-vm"></a>学生の VM に接続する
ラボ所有者 (教師または教授) がマシンにインストールされた RDP と GUI のパッケージを使用してテンプレート VM を**発行**すると、学生が学生の Linux VM に RDP 接続できるようになります。 手順は次のようになります。 

1. 学生がラボのポータルに直接 (`https://labs.azure.com`)、または登録リンク (`https://labs.azure.com/register/<registrationCode>`) を使用してサインインすると、学生がアクセスできる各ラボのタイルが表示されます。 
2. VM が停止状態の場合は、タイルでボタンを切り替えて VM を起動します。 
3. **[接続]** を選択します。 VM に接続するための 2 つのオプションが表示されます。**SSH** と**リモート デスクトップ**。

    ![学生用 VM - 接続オプション](../media/how-to-enable-remote-desktop-linux/student-vm-connect-options.png)

## <a name="connect-using-ssh-or-rdp"></a>SSH または RDP を使用して接続する
**[SSH]** オプションを選択した場合は、次の **[仮想マシンに接続する]** ダイアログ ボックスが表示されます。  

![SSH 接続文字列](../media/how-to-enable-remote-desktop-linux/ssh-connection-string.png)

テキスト ボックスの横にある **[コピー]** ボタンを選択して、クリップボードにコピーします。 SSH 接続文字列を保存します。 仮想マシンに接続するには、SSH ターミナル ([Putty](https://www.putty.org/) など) からこの接続文字列を使用します。

**[RDP]** オプションを選択した場合は、RDP ファイルがお使いのコンピューターにダウンロードされます。 それを保存してから開いて、マシンに接続します。 

## <a name="next-steps"></a>次の手順
クラスルーム ラボで Linux VM 用リモート デスクトップ接続機能を有効にする方法については、[Linux 仮想マシン用リモート デスクトップの有効化](how-to-enable-remote-desktop-linux.md)に関するページを参照してください。 

