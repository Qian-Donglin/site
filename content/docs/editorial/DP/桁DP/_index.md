---
title: "桁 Dp"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 実装の雛型

以下の部分だけで、$O(\log N) \times 20$かかる。ここに色々なフラグとかが付く。
具体的にはvを0から9と10通り試しているが、dpテーブルにはその次元は冗長である。

条件部分の`j | (v < digit)`が意味してるのは、

1. j == 0の未満フラグがない時、キープするには`v == digit`でなければならない。論理和を取るので、`v != digit`である。そして、前提条件で`j == 0 && v > digit`を排除してるので、`v < digit`にした。
2. j == 1の未満フラグがあるとき、もうすでにどの数字でもよい。遷移も1に移り続ける。

```cpp
dp.resize(N.size() + 1, vector<ll>(2, 0));
	//dp[i][j] := 上からi(1-idx)桁目で、jは未満なら1、イコールなら0

	dp[0][0] = 1;
	for (int i = 0; i < N.size(); i++) {
		int digit = N[i] - '0';
		for (int j = 0; j < 2; j++) {
			for (int v = 0; v <= 9; v++) {
				if (j == 0 && v > digit)break;
				
				dp[i + 1][j | (v < digit)] += dp[i][j];
			}
		}
	}
ans = dp[N.size()][0] + dp[N.size()][1];
//これは「0」も含む、N以下の値の全て。
```