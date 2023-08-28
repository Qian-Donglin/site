---
title: "(C) abc293 E(平方分割)"
weight: 1
katex: true
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[問題文](https://atcoder.jp/contests/abc293/tasks/abc293_e)

これは平方分割の解法であり、別解として

- 行列累乗
- int128と数論で計算。

がある。

# 概要

$$\Sigma_{i=0}^{X-1} A^i$$を$M$で割ったあまりを求めよ。

制約: 
$$
A, M \in [1, 10^9], X \in [1, 10^{12}]
$$

# 解法

明らかに全部計算するのは間に合わないが、$\sqrt{X}$回の加算なら間に合うところから出発する。

繰り返し二乗法で使われているように、$a^{20}=(a^{10})^2=((a^5)^2)^2$で、**一度まとまって計算したものをまた他と累乗させると手間が減る事を利用**。

今回の場合、$[0, \lfloor \sqrt{X} \rfloor)$までにiを実際に増加させて加算させてみる。これは$O(\sqrt{X})$。

$$
B = 1 + A^1 + A^2 + \cdots + A^{\lfloor \sqrt{X} \rfloor - 1}
$$

この時、

$$
BA^{\lfloor \sqrt{X} \rfloor} = A^{0 + \lfloor \sqrt{X} \rfloor} + A^{1 + \lfloor \sqrt{X} \rfloor} + \cdots + A^{2\lfloor \sqrt{X} \rfloor - 1}
$$

となる。よって、$BA^{\lfloor \sqrt{X} \rfloor}, BA^{2\lfloor \sqrt{X} \rfloor} \cdots$をそれぞれ$O(1)$で求められる。
これも$O(\sqrt{X})$。

**肩の指数が大きい方から取った何個かが残るかもしれない**ので、それについても考える。

[Atcoderの解説](https://atcoder.jp/contests/abc293/editorial/5976)では、残ったものは1つ1つ足し集めればよい。これはたかだか$O(\sqrt{X})$しかないので、全体でこのアルゴリズムは$O(\sqrt{X})$に重めの定数倍がついている。**これで通るはずだが、重い定数倍があるからか自分は通らなかった**。

## 残ったものの集計の高速化

残ったものは肩の指数が連続していて、$A^{k \lfloor \sqrt{X} \rfloor}, A^{k \lfloor \sqrt{X} \rfloor + 1} \cdots$のような値。これに関しては、**ステップ1の集計の時に一緒にこちらも集めて、最後に倍率を乗じて足せばよい**。

**何個余るかに関しては、明らかに$X \div \lfloor \sqrt{X} \rfloor$の余りの数だけ**なので、実装も容易。

# 実装について

いろいろと境界条件がこんがらがるが、**半開区間で処理、具体例でシミュレーション**である程度対策できる。

[ACコード](https://atcoder.jp/contests/abc293/submissions/39989476)