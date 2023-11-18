---
title: "0-Survey-2020"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元論文のリンク](https://link.springer.com/article/10.1007/s10994-020-05877-5)

2020年のサーベイ論文。[弱教師学習本](../../../read_book/Machine%20Learning/ML%20from%20Weak%20Supervision/_index.md)よりコンパクトにまとまっている。

## Introduction

PU Learningの現実での応用例

1. パーソナライズされた広告のクリック。押されなかったからと言って興味がないわけではない。
2. 診断されてないからといって、必ずそれにかかっていないことはない。
3. 知識ベースという知識のタプルがある。学習してないタプルだからと言って、その組み合わせは嘘であると言えない。

PU Learningの鍵となるResearch Question

1. PUのデータから学習するという問題にどう帰着するか。
2. 学習アルゴリズムを容易に設計するときには、どのような仮定が必要か。
3. PUのデータからクラス事前確率$p(y = +1)$を推定できるか？それは有用か？
4. PUのデータからどうして上手くモデルを学習できるか。
5. モデルの性能をどう評価すればいいか。
6. 実世界ではどのような応用があるか。
7. 他の機械学習分野へどう影響するか。

## PU Learningの準備

### Positive Negativeによる二値分類

一番理想的なケースである。$\mathbf{x}$の周辺確率分布は以下のように得られる。$\pi = p(y = +1)$

$$
\mathbf{x} \sim f(\mathbf{x}) = \pi f _+(\mathbf{x}) + (1 - \pi) f _- (\mathbf{x})
$$

### Positive Unlabeledによる二値分類

一般的には、PUデータは$(\mathbf{x}, y, s)$とラベル$s$を拡張。1ならばラベル付きである。ラベルが付いてるものは全てPositiveであるという仮定なので、$p(y = +1 | s = +1) = 1$。

### 学習のメカニズム

Positiveのデータは$\mathbf{x} \sim p(\mathbf{x} | y = +1)$から得られる。各Positiveのデータについて、選ばれる確率$p(s = +1 | \mathbf{x}, y = +1)$はPropensity SScoreと呼ばれる。[2008 Elkan, Noto](../PU%20Learning/_index.md)にも指摘しているように、一様にPositiveのなかから選ぶのであれば$\mathbf{x}$に依らない定数倍が成立する。

$$
p(\mathbf{x} | s = +1) = p(\mathbf{x} | s = +1, y = +1) = \frac{p(s = +1 | \mathbf{x}, y = +1)}{p(s = +1 | y = +1)} p(\mathbf{x} | y = +1)
$$

### single-training-setシナリオとcase-controlシナリオ

基本的には、いずれのデータセットも分布からデータを得るときはi.i.d.である。

single-training-setシナリオでは、各Positiveのデータは上にもある$p(s = +1 | \mathbf{x}, y = +1)$というデータに依存するラベル付けされやすさという値をもってしてラベルが付く。これらはすべて**同じデータセット**から得られている。定式化するなら、以下の通り。周辺確率ごと定義できるというものである。

$$
\mathbf{x} \sim f(\mathbf{x}) \\\\ 
\sim \pi f _+ (\mathbf{x}) + (1 - \pi) f _- (\mathbf{x})
$$

case-controlシナリオでは、PositiveとUnlabeledは別々のデータセットから来ている。別々のデータセットであるため、選択バイアスが発生していることが多い。また、クラス事前確率$p(y = +1)$が不明な時も多い。定式化するときは以下のものとなる。これは**ラベルなしの時だけわかるが、ラベルありについては全く分からない**。

$$
\mathbf{x} | s = 0 \sim f(\mathbf{x}) \\\\ 
\sim \pi f _+ (\mathbf{x}) + (1 - \pi) f _- (\mathbf{x})
$$

この論文では**後者を論じる**。大は小を兼ねる！

### クラス事前確率とラベル付けられる確率の関係

[2008 Elkan, Notoら](../PU%20Learning/_index.md)の論文を見ると詳しく解説されている。

