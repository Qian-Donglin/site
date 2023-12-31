---
title: "Self Supervised Learning"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# データ同士を比較する自己教師あり学習のsurvey

自分で生成するデータにラベルを付けてそのままで学習する。対照学習は自己教師ありの1つの分野。

- バックボーン あらかじめ事前学習した特徴抽出機を使って、その特徴で入力。エンコーダが一例。
- 対照学習(Contrastive Learning)　近いものは近い表現、遠いものは遠い表現と考える。
  - 画像分類、物体検出、音声処理などで使われる。
    - 画像の虫食い復元、白黒復元、ジグソーパズル復元など
  - 元の画像から加工して、**かさまし**してあげる。かさましした画像は**近い**と判定する。=Pretext Task
  - 上の条件を満たせるように、学習を進める(**人力を要しない！**)

- Pretext Task 画像の加工の例は以下の通り
  - ノイズをかけたり、ぼかしたり、明度を調節したり、色変換したり。
  - 反転したり。
- Frame Order based 動画でxフレーム目とx+δフレーム目って似てますよね？これで似てる画像を作り出せる。
  - 同じ人の動画か違う人の動画とか
- Future Prediction
  - 未来予測ができるような、特徴量を抽出できるエンコーダを訓練したい。
  - RNNを使う。LSTM使ってもよさそう。
- View Prediction 同じ動画を複数の視点から撮影されたのを検出できるようにすること。

正しいpretext taskを選ぼう！いい感じの変換を選ぶのは大事！

自然言語処理においての自己教師あり学習
- 文中から単語を抜いて、その穴埋めをするように学習する=center prediction
- 文中に一部単語を残して、隣を予測する。例えば10単語の文で何単語予測する=neighbor prediction
  - w0 ? ? ? T ? ? ? w8 w9
- 次の単語を予測する = next prediction
- 段落をかき乱して、それの並べ替え=sentence permutation

Batchサイズはかなり大事だとわかった。でかいと大事。

主なアーキテクチャは4つある。
なお、1 Batch当たりの画像が大きくなればpair wiseの比較ではなく、複数枚一気に比較してもOK。

- End to End
  - 2つの画像があったときに、エンコーダ(同一でも違うものでもOK)に入れて表現q, kをまず得る。qとkの近さをなんかシラノmeasure(cos類似度とか)で測量する。近い=+1, 遠い=-1にする。
  - Negative Sample(違うと判定する画像)の量が多いことが重要であるため、今までの各batchのデータもNegative例として扱う。
    - たまたま同じものでもNegative例として扱う。これは自己教師あり学習だからもうしゃあない。
  - エンコーダは表現1, 表現2があり、2は1よりもっと次元を落とした。2で近い、遠いを学習して、出来上がったエンコーダで本番の学習をするときは、表現1(次元を落としてない表現)を使う。
  - 結果的にGPUで殴った結果一番性能が良かった。下の3つよりも。
- Memory Bank
  - 今までのepochで生成した画像を貯めて、それを今の画像との近い遠い比較を行う。
  - あまりに昔の画像は消したほうがよさそう。
  - End to Endの方が性能よさそう。やっぱGPUパワーで殴るのが一番。
- Momentum Encoder
  - 2つのエンコーダを同時に学習させる。いる...?
  - パラメタの移動平均を取るだけで、誤差逆伝播なしに更新できて楽！
- Clustering
  - 表現2をそのまま使うのではなく、所属する画像のクラスを考えてクラスタリングした。
    - 先ほどの同じカテゴリの画像なのにNegative例と扱われてしまうのはまずいのでは？について答えた形。

Downstream Task=最終的に何の分類をやるのか
- 分類
- 外形を長方形でくくりだす
- 外形を性格にくくりだす。
- 複数の物体の識別をする。

