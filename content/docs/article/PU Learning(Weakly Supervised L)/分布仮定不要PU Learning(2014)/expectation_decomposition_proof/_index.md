---
title: "Expectation Decomposition Proof"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元記事](../_index.md)

以下のことを証明する。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [l _{01} (y f(\mathbf{x}))]
= p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] + \mathbb{E} _{p(\mathbf{x})} [ \tilde{l} (f(\mathbf{x})) ]
$$

損失関数は以下のように定義されているので、これをまず代入する。

$$
\tilde{l}(y f(\mathbf{x})) = \frac{2}{y + 3} l _{01} (y f(\mathbf{x})) \\\\ 
\tilde{l} _{\eta} (y f(\mathbf{x})) = \frac{2}{y + 3} l _{\eta} (y f(\mathbf{x})) \\\\ 
\mathbb{E} _{p(\mathbf{x}, y)} [l _{01} (y f(\mathbf{x}))] \\\\ 
= \frac{1}{2} p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ l _{01} (f(\mathbf{x})) ] + \mathbb{E} _{p(\mathbf{x})} [ \tilde{l} (f(\mathbf{x})) ] \\\\ 
= \frac{1}{2} p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ l _{01} (f(\mathbf{x})) ] + \frac{1}{2} p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ l _{01} (f(\mathbf{x})) ] + p(y = -1) \mathbb{E} _{p(\mathbf{x} | y = -1)} [ l _{01} (-f(\mathbf{x})) ] 
$$

これを整理すると、以下の式となり示せた。

$$
p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ l _{01} (f(\mathbf{x})) ] + p(y = -1) \mathbb{E} _{p(\mathbf{x} | y = -1)} [ l _{01} (-f(\mathbf{x})) ] \\\\ 
= \mathbb{E} _{p(\mathbf{x})} [ l _{01} (y f(\mathbf{x})) ]
$$