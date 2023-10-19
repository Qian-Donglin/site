---
title: "Lecture 2"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

大前提: **graphがsparseだとやりやすい。だがexpander graphはsparseのクセにdenseな完全グラフみたいなのと似た性質持っており、よろしくない**。

Real Graph(CoreとFringeを持つ)を$W: 2 \times \sqrt{n} \times \sqrt{n}$のグリッド(メモリかな？)に格納するには？

自然な解決策として、グラフの各頂点を$W$の連結したグラフにマッピングする(なにこれ)。ただコストが重いので、グラフの概形をキープしたまま縮約する= Graph Minorを考えることができる。

完全グラフ、(そしてその傾向がある)expander graphは、いずれも独立した連結成分の数は$2\sqrt{n}$未満である。ちなみに$n$って何かはわからない...

Fringe部のほぼ木のようなグラフのwidthは$2 \sqrt{n}$のオーダーらしい(なにこれ？)

結論、Expander Graphを$W: 2 \times \sqrt{n} \times \sqrt{n}$のグリッドに格納するのは難しい。

# グラフの分割

あまりに大きいグラフの場合、メモリに乗らなくて困ることになる。頂点で分割すると一部を載せることができるが、乗っている頂点と乗っていない頂点との情報交換コストは最低限に抑えたいよね。OSのページフォールトと同じ。上手いことメモリに載せた部分グラフはそのままで完結したい。