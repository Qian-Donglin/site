---
title: "EM Algorithm"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

この[Qiitaの記事](https://qiita.com/kenmatsu4/items/59ea3e5dfa3d4c161efb#4%E4%B8%80%E8%88%AC%E3%81%AEem%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0)や[このブログ](https://yamagensakam.hatenablog.com/entry/2020/01/06/015145)を参考に書いた。

## 対数尤度

基本的には尤度は非常に大きい、小さい値を扱うことを考えて、計算精度の問題上、対数で扱う。

データ$\mathbf{X}$の背後にあるパラメタ$\theta$を学習したい。しかし、隠れパラメタがある都合上上手くこのままでは難しい。

ここで、隠れパラメタ$\mathbf{Z}$だとする。以下のように周辺確率に分解でき、対数を取る事ができる。

$$
p(\mathbf{X} | \theta) = \int p(\mathbf{X}, \mathbf{Z} | \theta) d \mathbf{Z} \\\\ 
\log p(\mathbf{X} | \theta) = \log \int p(\mathbf{X}, \mathbf{Z} | \theta) d \mathbf{Z}
$$

しかし、$\log \int$は扱いづらい！(以下の参考資料では$\log \Sigma$となっている)。ここで、**いい感じの関数**$q(\mathbf{Z})$を導入することで、

$$
\int q(\mathbf{Z}) \frac{p(\mathbf{X}, \mathbf{Z} | \theta)}{q(\mathbf{Z})} d \mathbf{Z} = \mathbb{E} _{q(\mathbf{Z})} [ \frac{p(\mathbf{X}, \mathbf{Z} | \theta)}{q(\mathbf{Z})} ]
$$

**このように、$q(\mathbf{Z})$についての期待値**に変形できる。

そして、$\log$が**上に凸の関数**であるので、イェンゼンの不等式を用いると以下のように変形できる。汎関数$\mathcal{L}$という**変分下限**を得ることができる。

$$
\log \mathbb{E} [X] \leq \mathbb{E} [\log X] \\\\ 
\log \int q(\mathbf{Z}) \frac{p(\mathbf{X}, \mathbf{Z} | \theta)}{q(\mathbf{Z})} d \mathbf{Z} = \log \mathbb{E} _{q(\mathbf{Z})} [ \frac{p(\mathbf{X}, \mathbf{Z} | \theta)}{q(\mathbf{Z})} ] \\\\ 
\geq \mathbb{E} _{q(\mathbf{Z})} [ \log \frac{p(\mathbf{X}, \mathbf{Z} | \theta)}{q(\mathbf{Z})} ] = \int q(\mathbf{Z}) \log \frac{p(\mathbf{X}, \mathbf{Z} | \theta)}{q(\mathbf{Z})} d \mathbf{Z} = \mathcal{L}(q, \theta)
$$

変分下限と本来の$p(\mathbf{X} | \theta)$の差はKLダイバージェンスである$KL [ q(\mathbf{Z}) | p(\mathbf{Z} | \mathbf{X}, \theta) ]$が入る。下図参照。

$$
\log p(\mathbf{X} | \theta) = \mathcal{L} (q, \theta) + KL [ q(\mathbf{Z}) | p(\mathbf{Z} | \mathbf{X}, \theta) ]
$$

![](em-kldivergence.avif)

KLダイバージェンスは非負なので、$\log p(\mathbf{X} | \theta)$ではなく、$\mathcal{L} (q, \theta)$の最大化を代わりに行うことで結果的に$p(\mathbf{X} | \theta)$の最大化をしよう！(**下限を大きくするので必ず最善というわけではないが**)というのがEM Algorithm。

## EステップとMステップ

EステップはExpectation。**$\mathcal{L}(q, \theta)$の$\theta$を固定して、$q$を動かして最大化をする**。

MステップはMaximaization。**$\mathcal{L} (q, \theta)$の先ほど得た$q$を固定して、$\theta$を動かして最大化をする**。

E->M->E->M...と行い、収束したら終了。

### Eステップ

$$
\log p(\mathbf{X} | \theta) = \mathcal{L} (q, \theta) + KL [ q(\mathbf{Z}) | p(\mathbf{Z} | \mathbf{X}, \theta) ]
$$

この式において、$\theta$を固定しているので、左辺は定数となる。この時、$q$を動かして$\mathcal{L} (q, \theta)$を最大化するということは、$KL [ q(\mathbf{Z}) | p(\mathbf{Z} | \mathbf{X}, \theta) ]$の最小化と同じ意味であり、KLダイバージェンスが0ということは、解は$q(\mathbf{Z}) = p(\mathbf{Z} | \mathbf{X}, \theta)$である。

