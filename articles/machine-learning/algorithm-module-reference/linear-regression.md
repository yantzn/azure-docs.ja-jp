---
title: Linear regression (線形回帰):モジュール リファレンス
titleSuffix: Azure Machine Learning
description: Azure Machine Learning で線形回帰モジュールを使用して、パイプラインで使用するために線形回帰モデルを作成する方法について学習します。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 10/22/2019
ms.openlocfilehash: 688bf923c07d9417b002b7cab6e3c0a0c8d20dae
ms.sourcegitcommit: c22327552d62f88aeaa321189f9b9a631525027c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/04/2019
ms.locfileid: "73497749"
---
# <a name="linear-regression-module"></a>Linear Regression (線形回帰) モジュール
この記事では Azure Machine Learning デザイナー (プレビュー) のモジュールについて説明します。

このモジュールを使用して、パイプラインで使用するために、線形回帰モデルを作成します。  線形回帰では、1 つまたは複数の独立変数と数値の結果、または属性変数との間の線形関係を確立しようとします。 

このモデルを使用して線形回帰 モデルを定義し、ラベル付けされたデータセットを使用してモデルをトレーニングします。 その後、トレーニング済みのモデルは、予測に使用できます。

## <a name="about-linear-regression"></a>線形回帰について

線形回帰は一般的な統計メソッドであり、機械学習に導入され、直線に合わせて誤差を測定するための多くの新しいメソッドと共に強化されています。 ほとんどの基本的な意味で、回帰は数値目標の予測を指します。 線形回帰は、引き続き基本的な予測タスクにシンプルなモデルが必要なときに適した選択です。 線形回帰は、複雑さが欠けている高次元のスパース データのセットで適切に機能する傾向もあります。

Azure Machine Learning では、線形回帰に加えて、さまざまな回帰モデルをサポートします。 しかし、"回帰" という用語は大まかに解釈される場合があり、他のツールで提供される一部の種類の回帰はサポートされません。

+ 従来の回帰に関する問題は、1 つの独立変数と従属変数に関係します。 これは "*単純回帰*" と呼ばれます。  このモジュールでは、単純回帰をサポートします。

+ "*複数の線形回帰*" は、1 つの従属変数に関与する、2 つ以上の独立変数に関係します。 複数の入力が 1 つの数値の結果を予測するために使用される問題は、"*多変量線形回帰*" とも呼ばれます。

    **Linear Regression (線形回帰)** モジュールでは、その他のほとんどの回帰モジュールでできるように、これらの問題を解決することができます。

+ "*複数ラベル回帰*" は、1 つのモデル内の複数の従属変数を予測するタスクです。 たとえば、複数ラベルのロジスティック回帰で、サンプルを複数の異なるラベルに割り当てることができます。 (これは 1 つのクラス変数内に複数のレベルを予測するタスクとは異なります。)

    この種類の回帰は、Azure Machine Learning ではサポートされません。 複数の変数を予測するには、予測したい出力ごとに個別の学習者を作成します。

長年にわたって、統計学者は回帰にますます高度なメソッドを開発しています。 これは線形回帰にも当てはまります。 このモジュールでは、誤差を測定し、回帰直線に適合させるために 2 つのメソッドをサポートします。通常の最小二乗法メソッドと勾配降下です。

- **勾配降下**は、モデル トレーニング プロセスの各ステップで誤差の量を最小限にするメソッドです。 勾配降下には多くの変数があり、さまざまな学習に関する問題の最適化は広く学習されています。 **[Solution method]\(ソリューション メソッド\)** にこのオプションを選ぶと、ステップ サイズ、学習速度などを制御するために、さまざまなパラメーターを設定できます。 このオプションでは、統合パラメーターの一括処理の使用もサポートします。

- **通常の最小二乗法**は、線形回帰で最も一般的に使用される技術です。 たとえば、Microsoft Excel の分析ツールで使用されるメソッドは最小二乗法です。

    通常の最小二乗法は、実績値から予測された線までの距離の平方の合計として誤差を計算し、二乗エラーを最小限にすることでモデルを適合させる損失関数を示します。 このメソッドは、入力と従属変数との間の強力な線形関係を前提とします。

## <a name="configure-linear-regression"></a>線形回帰を構成する

このモジュールでは、次の異なるオプションと共に、回帰モデルを調整するために 2 つのメソッドをサポートします。

+ [オンライン勾配降下を使用して回帰モデルを作成する](#bkmk_GradientDescent)

    勾配降下は、より複雑、または変数の数を指定されたトレーニング データが少なすぎるモデルに対して優れた損失関数です。



+ [通常の最小二乗法を使用して回帰モデルを調整する](#bkmk_OrdinaryLeastSquares)

    小さなデータセットには、通常の最小二乗法を選択するのが最適です。 これによって、Excel に同様の結果が適用されます。

## <a name="bkmk_OrdinaryLeastSquares"></a>通常の最小二乗法を使用して回帰モデルを作成する

1. インターフェイスから**Linear Regression Model (線形回帰モデル)** モジュールをパイプラインに追加します。

    **[Machine Learning]** カテゴリでこのモジュールを見つけることができます。 **[Initialize Model]\(モデルの初期化\)** を展開し、 **[Regression]\(回帰\)** を展開して、**Linear Regression Model (線形回帰モデル)** モジュールを自分のパイプラインにドラッグします。

2. **[プロパティ]** ウィンドウの **[Solution method]\(ソリューション メソッド\)** ドロップダウン リストで、 **[Ordinary Least Squares]\(通常の最小二乗法\)** を選択します。 このオプションでは、回帰直線を見つけるために使用する計算メソッドを指定します。

3. **[L2 regularization weight]\(L2 正規化の重み\)** に、L2 正規化に対する重みとして使用する値を入力します。 オーバーフィットを避けるために、ゼロ以外の値を使用することをお勧めします。

     正規化がモデルの調整にどのように影響するかの詳細については、「[機械学習向けの L1および L2 正規化](https://msdn.microsoft.com/magazine/dn904675.aspx)」を参照してください。

4. 切片の用語を表示する場合、 **[Include intercept term]\(切片の用語を含める\)** オプションを選択します。

    回帰式を確認する必要がない場合、このオプションの選択を解除します。

5. **[Random number seed]\(乱数シード\)** には、必要に応じて、モデルによって使用される乱数ジェネレーターにシードを設定する値を入力できます。

    同じパイプラインにおけるさまざまな実行を超えて同じ結果を保持する必要がある場合、シード値を使用すると便利です。 それ以外の場合、既定はシステム クロックからの値を使用します。


7. [Train Model (モデルのトレーニング)](./train-model.md) モジュールを自分のパイプラインに追加して、ラベル付けされたデータセットに接続します。

8. パイプラインを実行します。

## <a name="results-for-ordinary-least-squares-model"></a>通常の最小二乗法モデルの結果

トレーニングの完了後、次の作業を行います。

+ モデルのパラメーターを表示するには、トレーナーの出力を右クリックして、 **[Visualize]\(可視化\)** を選択します。

+ 予測するには、新しい値のデータセットと共に、トレーニング済みのモデルを [Score Model (モデルのスコア付け) ](./score-model.md)モジュールに接続します。 


## <a name="bkmk_GradientDescent"></a>オンライン勾配降下を使用して回帰モデルを作成する

1. インターフェイスから**Linear Regression Model (線形回帰モデル)** モジュールをパイプラインに追加します。

    **[Machine Learning]** カテゴリでこのモジュールを見つけることができます。 **[Initialize Model]\(モデルの初期化\)** を展開し、 **[Regression]\(回帰\)** を展開して、**Linear Regression Model (線形回帰モデル)** モジュールを自分のパイプラインにドラッグします

2. **[プロパティ]** ウィンドウの **[Solution method]\(ソリューション メソッド\)** ドロップダウン リストで、回帰直線を見つけるために使用する計算メソッドとして、 **[Online Gradient Descent]\(オンライン勾配降下\)** を選びます。

3. **[Create trainer mode]\(トレーナー モードの作成\)** では、定義済みのパラメーターのセットでモデルをトレーニングするかどうか、またはパラメーターの一括処理を使用することでモデルを最適化する必要があるかどうかを示します。

    + **Single Parameter (単一パラメーター)** : 線形回帰ネットワークの構成方法が既にわかっている場合は、特定の値のセットを引数として渡すことができます。

   
4. **[Learning rate]\(学習速度\)** では、確率勾配降下オプティマイザーに初期学習速度を指定します。

5. **[Number of training epochs]\(トレーニング エポックの数\)** には、アルゴリズムが例を通して繰り返す必要がある回数を示す値を入力します。 少数の例を含むデータセットでは、この数値は収束に達するために大きくなります。

6. **Normalize features (フィーチャーの正規化)** : モデルをトレーニングするために使用された数値データを既に正規化している場合、このオプションの選択を解除できます。 既定では、モジュールによって、0 と 1 の間の範囲へのすべての数値入力が正規化されます。

    > [!NOTE]
    > 
    > スコア付けに使用される新しいデータに同じ正規化メソッドを提供することを忘れないでください。

7. **[L2 regularization weight]\(L2 正規化の重み\)** に、L2 正規化に対する重みとして使用する値を入力します。 オーバーフィットを避けるために、ゼロ以外の値を使用することをお勧めします。

    正規化がモデルの調整にどのように影響するかの詳細については、「[機械学習向けの L1および L2 正規化](https://msdn.microsoft.com/magazine/dn904675.aspx)」を参照してください。


9. イテレーションが進むときに学習速度を下げる必要がある場合、 **[Decrease learning rate]\(学習速度の低下\)** のオプションを選択します。  

10. **[Random number seed]\(乱数シード\)** には、必要に応じて、モデルによって使用される乱数ジェネレーターにシードを設定する値を入力できます。 同じパイプラインにおけるさまざまな実行を超えて同じ結果を保持する必要がある場合、シード値を使用すると便利です。


12. ラベル付けされたデータセットと、トレーニング モジュールの 1 つを追加します。

    統合パラメーターの一括処理を使用していない場合、[Train Model (モデルのトレーニング)](train-model.md) モジュールを使用します。

13. パイプラインを実行します。

## <a name="results-for-online-gradient-descent"></a>オンライン勾配降下の結果

トレーニングの完了後、次の作業を行います。

+ 予測するには、新しい入力データと一緒に、トレーニング済みのモデルを [Score Model (モデルのスコア付け)](./score-model.md) モジュールに接続します。


## <a name="next-steps"></a>次の手順

Azure Machine Learning で[使用できる一連のモジュール](module-reference.md)を参照してください。 