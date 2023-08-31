---
title: "(G+) abc317 E"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

https://atcoder.jp/contests/abc317/tasks/abc317_e

$H \times W$のグリッドがある。各グリッドは壁は通路で、始点終点がある。グリッドには人が立っている可能性があり、人の向いてる方向は上下左右`^v<>`である。人は向いてる方向で他の人や壁に当たらない限り視線が通っている。

人の視線を完全に避けて始点から終点まで行く補数を求める。

制約: $H, W \in [1, 2000]$

# 解法

上手く視線が通る場所をすべて印をつけたい。しかし人はいっぱいいて**全てのマス付けると間に合わなさそう**。

**ん？？？？？**間に合わない？本当に？よく考えて。

各マスは上下左右から見られるが、視線がさえぎられる関係上、**各マスの上下左右はそれぞれたかだか1人までにしかカバーされない**。

つまり、各マスはたかだか4回までしか見られない！実は大丈夫じゃないか！

[ACコード](https://atcoder.jp/contests/abc317/tasks/abc317_e)

**間に合わなさそうだと思っても、それぞれはたかだかX回とちゃんと評価することが大事。グリッドに限った話ではないが。**

実装に関しては各方向でそれぞれ書くと大変なので、埋める部分は次のように書いた。

```cpp
//視線が通っている処に印をつける。
	for (int i = 0; i < H; i++) {
		for (int j = 0; j < W; j++) {
			if (field[i][j] == '>' || field[i][j] == 'v' || field[i][j] == '<' || field[i][j] == '^') {
				isNG[i][j] = true;
				//その方向に印をつけていく。
				int direction;
				if (field[i][j] == '>') direction = 0;
				if (field[i][j] == 'v') direction = 1;
				if (field[i][j] == '<') direction = 2;
				if (field[i][j] == '^') direction = 3;

				for (int delta = 1;; delta++) {
					int nh = i + vx[direction] * delta, nw = j + vy[direction] * delta;
					if (nh < 0 || nw < 0 || nh >= H || nw >= W)
						break;
					if (field[nh][nw] == '#' || field[nh][nw] == '>' || field[nh][nw] == 'v' || field[nh][nw] == '<' || field[nh][nw] == '^')
						break;
					isNG[nh][nw] = true;
				}
			}
		}
	}
```

# オーバーキルというか無駄

1マスごとに見られる回数の上限に気づかなかった場合。

1. 縦のi列目の出現する壁や人のindexをまとめる二次元vectorを用意。横も同様。
2. 上のものには最初は-1, 最後にはH or Wの盤兵を入れておく。
3. これらは単調増加なので、各人にフォーカスするとき、次にどこまで埋めればいいのかは二分探索で求められる。
4. 埋める作業に関しては二次元imos法で行えば最後にまとめてでできる。
5. こうして、同様に視線が通ったマスの全てを洗い出せる。

オーバーキルだしこれを書いてる途中に1マスごとに何回までしか見られないのかはちゃんと気付くべき。

