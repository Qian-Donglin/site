---
title: "Lecture 1"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

real graphでは、完全グラフみたいな密なグラフは少ない。その中で、辺の次数が高い頂点の集団=密な部分の**Core**と、それ以外のほぼ木、一本線のような**Fringe**に分けられる。Coreの部分はExpander Graphである。

**Expanding Graph**は以下の性質を持つ。

1. つながりが密で、カットするときに取り除くべき辺が多い。
2. ランダムウォークの収束が素早い(これなに)
3. グラフの遷移行列の固有値が小さい。

では、そんなExpander GraphすなわちCore部分を見つけるのはどうすればよいか？NP-hardです……。
固有値(何の行列の？隣接行列？)が複素平面でプロットしたとき、減点の周りにクラスタ化されている場合、expander graphになることはわかる。

expander graphは密に結合されてるので分割統治法+DPの相性が悪い。逆にFringeは分割統治法+DPで計算しやすい(バラバラなんで)が、ランダムウォークの収束が遅い。

