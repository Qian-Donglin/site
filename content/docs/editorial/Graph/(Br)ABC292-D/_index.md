---
title: "(Br) abc292 D"
weight: 1
katex: true
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[問題リンク](https://atcoder.jp/contests/abc292/tasks/abc292_d)

# 問題概要

あるN頂点M辺無向グラフ(**多重辺、自己ループあり**)を与えられる。

全ての**連結成分の頂点数と辺の数**が同じかを判定せよ。

制約:  $ N, M < 2 \times 10^5 $

# 自分の解法

木は頂点数=辺の数+1であると知られているので、**木に1つだけ辺を追加したものであるか？の判定に帰着**。

グラフのまだ見てない頂点に関して、DFSしながら、すでに訪ねたことのある頂点にブチ当たった回数をカウントする。

基本的には、2回ならば(DFSの行きで1回ぶちあたり、親頂点に戻って来てまた掘り進めようとしたら前に掘った兄弟の子孫にぶちあたる)条件に該当しそう。

以下のように実装。

```cpp
int encountSame = 0;//visitedな頂点に当たった回数
int massSize = 1;//今の連結成分のサイズ

void dfs(int now, int bef) {
	for (int i = 0; i < E[now].size(); i++) {
		auto next = E[now][i];
		if (bef == next)continue;
		if (!visited[next]) {
			visited[next] = true;
			massSize++;
			dfs(next, now);
		}
		else {
			encountSame++;
		}
	}
}
```

しかし、今回の問題は**多重辺、自己ループ**を許容していたので、面倒な例外処理をすることに。

## 例外処理

サイズ1の場合条件を満たすのは、自己ループが1つだけの場合。これを上のアルゴリズムに適用させると、`encuntSame == 2, massSize == 1`となり、例外。

サイズ2の場合は、多重辺の場合(`2->3, 2->3`のように)、`encountSame == 1, massSize == 2`となる。理由は、例だと頂点2から3へ移動したとき、戻るときはさっき使った多重辺以外を使えばいいが、DFSでbefを保持してる以上、戻れないから(**サイズ2だとループ完成=直前に戻る**)

よって以下のように。

```cpp
for (int i = 0; i < N; i++) {
	if (!visited[i]) {
		massSize = 1;
		visited[i] = true;
		encountSame = 0;
		dfs(i, -1);
		if ((massSize != 2 && encountSame == 2) || (massSize == 2 && encountSame == 1))
			;
		else isOK = false;
	}
}
```

テストケースにこれが置いてあったので気づけたけどなかったら気づくのはかなり遅れたはず。この方法はよくない。

## Union-Find

**グラフの連結成分の個数といえば、同じラベルの要素の検索とサイズを高速取得できるUnion-Find木**。

今回の場合連結成分の頂点数と辺の数を調べたいわけだが、同じ連結成分について**親のIDに頂点数が何個、辺が何本と毎回確認することで事足りる**。

なお、実装する際に、いつもの無向辺隣接リストで行う場合、

- 辺の二重カウント防止に辺の始点or終点の頂点だけ見て、それの親頂点に加算。
- `2->3`で親が例えば2の場合、`3->2`の逆辺も持つので、始点見るなら`root(2)==2, root(3)==2`と親に2回足されるので、最後に割る2しておく。

```cpp
UnionFind uf(N);
for (int i = 0; i < M; i++) {
	int a, b;
	cin >> a >> b;
	a--, b--;
	E[a].push_back(b);
	E[b].push_back(a);
	uf.unite(a, b);
}

for (int i = 0; i < N; i++) {
	countNode[uf.getroot(i)]++;
}

for (int i = 0; i < N; i++) {
	for(auto to : E[i])
		countEdge[uf.getroot(to)]++;
}

bool isOK = true;
for (int i = 0; i < N; i++) {
	if (countNode[i] != countEdge[i] / 2)
		isOK = false;
}
```

[ACソース](https://atcoder.jp/contests/abc292/submissions/39545335)