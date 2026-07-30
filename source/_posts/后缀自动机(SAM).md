---
title: 后缀自动机（SAM）
date: 2026-07-29 15:26:09
tags: [算法竞赛]
---

本文系统讲解后缀自动机（SAM）的构建、性质、经典例题及广义SAM、线段树合并、LCT等高级应用。

<!-- more -->

### SAM

发现有一些和子串相关的问题比较难解决。

而 SAM 就是一种解决字符串子串问题的利器。

SAM 是一个 **DAG**，这个 DAG 的每一个节点都表示了匹配的状态。

而节点之间的边则表示转移，也就是在原来基础上加上字符后的匹配状态。

~~好吧，到现在你还是完全不能想象 SAM 长什么样。~~

在构建 SAM 之前，首先有一些定义。

设 $endpos(t)$ 为 $s$ 的子串 $t$ 在 $s$ 中出现的结束位置集合。

简单举个例子：

```cpp
s:abaababa
t:aba
```

 发现 $s_{[1,3]}=s_{[4,6]}=s_{[6,8]}=t$。

所以 $endpos("aba")=\{3,6,8\}$。

根据 $endpos$ 集合的不同，我们将 $endpos$ 集合相同的子串划分为一个等价类。

根据定义，可以知道对于 $s$ 的两个非空子串 $u$，$v$  （$|u|<|v|$）。

若 $endpos(u)=endpos(v)$ 则 $u$ 在 $s$ 中的每次出现都是以 $v$ 的后缀的形式存在的。

（这个比较显然）

**而如果有两子串 $u$，$v$ （$|u|<|v|$），则要么 $endpos(u)\cap endpos(v) = \varnothing$**

**要么 $endpos(v)\subseteq endpos(u)$。**

这个比较重要，假设 $endpos(u)$ 和 $endpos(v)$ 的交不为空集。

可知 $u$ 和 $v$ 存在一个位置，使得它们在这个位置上同时结束。

于是 $u$ 就为 $v$ 的后缀，即 $endpos(v)\subseteq endpos(u)$。

根据上面的这些性质，容易得出，一个 $endpos$ 等价类中的所有字符串按长度排序，

设其中长度最短的为 $l$，有 $d$ 个字符串，则这个等价类中的字符串长度刚好覆盖 $[l,l+d-1]$。



发现 $endpos$ 等价类有一些优美的性质，可以当做 SAM 的节点，但是这还不够。

需要一个叫后缀链接（$link$）的东西。

对于一个 $endpos$ 等价类，设其中最长的串为 $u$，则等价类中的其他串都是 $u$ 的后缀。

而 $u$ 的有几个后缀，它们是不和 $u$ 在同一个等价类里的。

设这个最长的后缀所对应的节点为 $P$，$u$ 所对应的节点为 $Q$，则 $link(Q)=P$。

因为根据上面的定义 $endpos(Q)\subsetneq endpos(link(Q))$。

所以显然，这样 $link$ 连出来的图是内向森林。

为了方便，不妨加入一个空串作为初始状态 $t_0$，并将相应的 $link$ 连向它。

这样，所有的 $link$ 就会构成一棵树了。

![](https://cdn.luogu.com.cn/upload/image_hosting/5jebym7g.png)

上图就是对 $ababc$ 构建 SAM 的结果，右图就是 $link$ 所构成的树（parent 树）

可以发现左图中任意一个状态在末尾加上字符，就会转移到一个新的状态上（或者不存在这个状态）



但是怎么让构建 SAM 抽象为一个算法呢？

接下来讲解一种 **动态加入每一个字符** 的做法。

设 $c$ 为当前字符， $lst$ 为加入 $c$ 之前，上一个字符更新所得到的状态。

（特别的，没有加入字符时的 $lst$ 为 $t_0=1$）

然后的过程就有点复杂，这里就不给证明为什么是对的了，只讲述步骤，感性理解。

1. 加入字符 $c$ 时，加入一个新的状态 $t$，$maxlen_t=maxlen_{lst}+1$ 其中 $maxlen$ 表示的是等价类中最长的字符串的长度，新的状态 $t$ 表示的是加入 $c$ 后的整串的等价类，$lst$ 表示的是加入前的整串的等价类 。

（因为是在字符串末尾加入的字符 $c$，所以长度是在之前的长度基础上 $+1$）

2. 从状态 $lst$ 开始，一直往上跳，如果该点没有找到字符为 $c$ 的转移，那么就添加一个，直到有了转移为止。

3. 找到第一个有 $c$ 转移的点 $p$，设从 $p$ 通过 $c$ 转移到 $q$，因为如果不处理从这个点向后转移会产生“歧义”。（特别地，如果没有找到，直接将 $link$ 连向 $0$）

   如果 $maxlen_p+1=maxlen_q$ 也就是已经存在这个等价类了，那么只需要将 $link(t)$ 赋值为 $q$。

   否则需要建一个新的状态，设为 $nw$，对于复制 $q$ 的后缀链接和转移（因为此时 $q$ 等价类一定包含当前的等价类，所以后缀链接和转移都能复制）。
   
   然后将 $maxlen_{nw}$ 赋值为 $maxlen_p+1$，将 $link(t),link(q)$ 赋值为 $nw$。   

4. 最终我们从状态 $p$ 沿着 $link$ 向上走，将所有这些指向 $q$ 的转移都指向 $nw$（因为出现了这个新的等价类，而这些点可以转移到当前等价类）
5. 这样就可以构造出点数边数都为 $O(n)$ 的 SAM 了。

```cpp
inline void insert(int c){
	int u=++tot,v=lst;
    s[u]=1;// 表示结束位置
    //
	T[u].len=T[v].len+1;
	while(v&&!T[v].sn[c])T[v].sn[c]=u,v=T[v].fa;
	if(!v)T[u].fa=1;
	else{
		int w=T[v].sn[c];
		if(T[v].len+1==T[w].len)T[u].fa=w;
		else{
			int nw=++tot;
			memcpy(T[nw].sn,T[w].sn,104);
			T[nw].fa=T[w].fa;T[nw].len=T[v].len+1;
			while(v&&T[v].sn[c]==w)T[v].sn[c]=nw,v=T[v].fa;
			T[w].fa=T[u].fa=nw;
		}
	}
	lst=u;
}
```

经过 `T[u].sn[c]` 相当于到达 $u$ 中**最长字符串**后面加上字符 $c$ 形成字符串所在的等价类。

 `T[u].fa` （也就是 link）就相当于在 $u$ 中**最短**的字符串去掉第一个字符形成的字符串所在的等价类。

#### 例题 1.1

[【模板】后缀自动机（SAM）](https://www.luogu.com.cn/problem/P3804)

这道题就是对 SAM 的基本理解与运用。

对于每个等价类，显然只有最长的那个可能产生贡献。

这样只要计算出每个等价类所出现的次数就可以了。

这个只用将 $link$ 所形成的树建出来，然后统计每个节点子树中结束位置的数量即可。

对于一个节点 $u$，它子树里的节点 $v$ 满足 $endpos(v) \subsetneq endpos(u)$。

等价类 $u$ 里的所有字符串都是等价类 $v$ 中字符串的前缀。

对于 $v$ 是结束位置的情况，$u$ 中的字符串 即为字符串 $[1,结束位置]$ 的一个后缀，也就是出现了一次。

所以出现次数总和就是节点子树中结束位置的数量。

直接取 $\max$ 即可。

```cpp
inline int solve(int x){
	int sz=(s[x])?(1):(0);
	for(int S:vec[x])
		sz+=solve(S);
	if(sz>1)ans=max(1ll*sz*T[x].len,ans);
	return sz;
}
```



#### 例题 1.2

[Cyclical Quest](https://www.luogu.com.cn/problem/CF235C)

这题也是对 SAM 的基本理解。

设 $sz_u$ 为节点 $u$ 子树中结束位置的数量。

则一个字符串 $T$ 在原串 $S$ 中的出现次数就是沿着 SAM 的转移边走，走到的节点的 $sz$。

但是这道题要求我们去求所有的循环同构的出现次数。

比较套路地，先将字符串复制一份接在后面。

然后和单个字符串匹配一样匹配，但是不同的是，如果出现了匹配不上的情况，可以跳 $link$。

前面说过跳 $link$ 相当于从前面减少若干字符，于是上面这个操作实际上就是一个双指针。

每次将右端点 $R$ 向右推一格，如果不能匹配（左端点太远了），

那么就通过跳 $link$ 的方式右移左端点。

直到 $R-L+1=|T|$，也就是匹配上了一个循环同构，这时候加上 $sz$ 即可。

注意到本题中，相同的循环同构只算一次，需要在 SAM 上打标记，保证只算一次。

```cpp
		for(int i=1;i<2*l;++i){
			int c=t[(i>l)?(i-l):(i)]-'a';
			while(nw&&!T[nw].sn[c])nw=T[nw].fa,len=T[nw].len;
			if(T[nw].sn[c])len++,nw=T[nw].sn[c];
			else nw=1,len=0;
			if(len==l){
				if(!vis[nw])ans+=T[nw].sz,vis[nw]=1,Clr.push_back(nw);// 标记
				if(T[T[nw].fa].len+1==l)nw=T[nw].fa;
				len--;
			}
		}
```

#### 例题 1.3

[[BJOI2020] 封印](https://www.luogu.com.cn/problem/P6640)

感觉是非常板子的一道题，对 $s$ 建后缀自动机，

通过和上面一题类似的方法可以求出从每个位置开始的最长公共子串长度 $len_i$。

然后这就和 SAM 没关系了。

要求的答案是 $\max_{i=l}^r\min(len_i,i-l+1)$。

在线不是很好做，考虑离线，将 $len_i>i-l+1$ 单独弄一个堆维护。

对于 $len_i\le i-l+1$ 的可以放进树状数组来做。

#### 例题 1.4

[Paper task](https://www.luogu.com.cn/problem/CF653F)

给定一个长度为 $n$ 的括号串 $s$，问有多少种本质不同的合法括号子串。 

首先建出 SAM，这样就可以算出本质不同的括号串数量了。

但是这些括号串不一定合法。

把 `(` 当做 $1$，`)` 当做 $-1$，做一遍前缀和 $f_i$。

$s_{[l,r]}$ 合法的条件是 $f_r=f_{l-1}$ （左右括号数相等）且 $\min_{i=l}^r f_i \ge f_r$（否则会有位置右括号没得匹配）

对于每一个 $f_i$ 都开一个 `vector` 把对应 $f$ 值的位置放进去。

这样就可以满足 $f_r=f_{l-1}$ 的限制。

对于 $\min_{i=l}^r f_i \ge f_r$ ，可以用 ST 表处理区间最大值。

然后可以二分找到满足条件的最靠前的 $l$。

```cpp
	int posL=T[x].st-T[x].len,posR=T[x].st;
	int t=f[posR]+N;
	int nwL=lower_bound(vec[t].begin(),vec[t].end(),posL)-vec[t].begin(),nwR=lower_bound(vec[t].begin(),vec[t].end(),posR-T[T[x].fa].len)-vec[t].begin()-1,mn=nwR+1,z=nwR;
	while(nwL<=nwR){
		int mid=nwL+nwR>>1;
		if(query(vec[t][mid],posR)>=t-N){
			mn=min(mn,mid);nwR=mid-1;
		}else nwL=mid+1;
	}
```

[Yet Another LCP Problem](https://www.luogu.com.cn/problem/CF1073G)

两个后缀在**原串**后缀树上的 $\text{lca}$ 的节点的最长长度等于他们的 $\text{lcp}$ 长度。

然后就是板子了，我们直接建出反串的 SAM，然后询问就相当于给定两个点集询问任取两点的 $\text{lca}$ 深度之和。

建出虚树直接做即可。

### 广义 SAM

SAM 非常有用，但是有时候会有多串问题，于是我们需要将 SAM 拓展。

一种比较主流的写法是将所有的串都放在一个 Trie 上，通过 BFS/DFS 来构建 SAM。

将所有的字符串插入一个 Trie 中，字典树上一个节点到根的路径代表的是一个前缀。

这其中有些地方有一点像后缀自动机，比如 `T[u].sn[c]` 都表示在状态 $u$ 表示的字符串后面加入一个 字符 $c$，所形成的字符串对应的状态。



因此可以通过一些转化将字典树改造为广义 SAM。

这里具体讲解一下怎么改造。

单串构建 SAM 中我们选取的是上一个字符对应的节点为 `lst`，在字典树上对应的就是父亲节点在 SAM 里的对应节点。

为了能够保证每个节点的父亲节点都先于它被插入了，

我们用 BFS 的方法，一层一层遍历 Trie 并将这些节点插入，当然也可以用 DFS，只是在一些情况下复杂度会伪。

插入的大部分内容和普通 SAM 的构建差不多，只是受到原来的 Trie 树的结构的影响，要做出一定改变。（可以对照上面的 SAM）

1. 找到有字符 $c$ 转移的节点的这个过程中，要从 $lst$ 的父亲节点开始。
   - 考虑到在原来的 SAM 中状态 $lst$ 是上一次加入的节点，此时还没有任何转移边，所以一定会在 $lst$ 添加一条通过 $c$ 到达当前节点的转移
   - 而现在我们是在一个字典树的基础上改造，开始已经有了字典树，也就是有了 $lst$ 到 $u$ 的转移，不用添加。
2. 与普通的 SAM 不同，新建节点复制转移的时候，要判断转移到节点的 $len$ 为 $0$ 的情况。
   - 因为已经有 Trie 树了，所以一些转移可能指向压根没加入的节点（$len=0$），不能将这些节点复制了。

代码：

```cpp
struct GSAM{
	static const int N=1000000;
	typedef long long ll;
	int tot=1;
	struct node{
		int len,fa,sn[26];
	}T[N*2+5];
	int insert(int lst,int c){
		int v=lst,u=T[v].sn[c];
		T[u].len=T[v].len+1;
		int p=T[v].fa;
		while(p&&!T[p].sn[c])T[p].sn[c]=u,p=T[p].fa;
		if(!p){T[u].fa=1;return u;}
		int q=T[p].sn[c];
		if(T[p].len+1==T[q].len){T[u].fa=q;return u;}
		int w=++tot;
		for(int i=0;i<26;++i)T[w].sn[i]=T[T[q].sn[i]].len?T[q].sn[i]:0;
		T[w].len=T[p].len+1;
		while(p&&T[p].sn[c]==q)T[p].sn[c]=w,p=T[p].fa;
		T[w].fa=T[q].fa;T[q].fa=T[u].fa=w;return u;
	}
	inline void Tinsert(char *s,int len){
		for(int i=1,nw=1,c;i<=len;++i){
			c=s[i]-'a';
			if(!T[nw].sn[c])
				T[nw].sn[c]=++tot;
			nw=T[nw].sn[c];
		}
	}
	inline void Build(){
		queue<pair<int,int> > q;
		for(int i=0;i<26;++i)
			if(T[1].sn[i])
				q.push(make_pair(i,1));
		while(!q.empty()){
			pair<int,int> t=q.front();q.pop();
			int lst=insert(t.second,t.first);
			for(int i=0;i<26;++i)
				if(T[lst].sn[i])
					q.push(make_pair(i,lst));
		}
	}
	inline ll CountDiffSubs(){
		ll sum=0;
		for(int i=2;i<=tot;++i)
			sum+=T[i].len-T[T[i].fa].len;
		return sum;//本质不同子串数量
	}
}S;
```

#### 例题 2.1

[Little Elephant and Strings](https://www.luogu.com.cn/problem/CF204E)

发现这个题在只有单串的时候就是 SAM 板子题。

现在有很多串，自然想到建立广义 SAM 解决。

要求出现次数 $\ge k$ ，则我们需要快速求出广义 SAM 的每个节点出现在多少个不同的串中。

可以枚举每个串的每一个前缀所对应的节点，则从它开始到根的所有节点都出现在了该串中。

此时，这个节点的出现次数 $+1$，但是要注意对于一个串不能重复加同一个节点。

这里可以通过在 parent 树上倍增加差分的方法做到 $O(|S|\log |S|)$ 的复杂度。

但是其实直接暴力进行树链加时间复杂度也是对的，是 $O(|S|\sqrt{|S|})$ 的，而且常数不大。

```cpp
	for(int i=1;i<=n;++i){
		int nw=1;
		for(char j:st[i]){
			nw=S.T[nw].sn[j-'a'];
			int tnw=nw;
			while(nw&&tag[nw]!=i)
				tag[nw]=i,sm[nw]++,nw=S.T[nw].fa;
			nw=tnw;
		}
	}
```

1. 考虑长度 $\ge \sqrt{|S|}$ 的串，由于不能重复加，所以这个串最多将自动机上的所有节点都加一遍。

   这样的串最多只有 $\sqrt{|S|}$ 个，对于一个串的复杂度最多是 $O(|S|)$ 的，总复杂度 $O(|S|\sqrt{|S|})$。

2. 考虑长度 $\le \sqrt{|S|}$ 的串，设其长度为 $|T|$。

   跳 parent 树每次都至少会使节点代表的字符串长度 $-1$。

   而 $|T|$ 的所有前缀的长度总和为 $\frac{|T|\times(|T|+1)}{2}$ 也就是 $O(|T|^2)$ 的。

   而长度为 $|T|$ 的串只有 $\frac{|S|}{|T|}$ 个，算算发现就是对的。

   于是这一部分复杂度是 $O(|T||S|)$ 也就是最多是 $O(|S|\sqrt{|S|})$ 的。

这也是一个比较常见的 trick。



题目要求我们对每一个串进行求值，

这让我们想到了 Cyclical Quest  这一题，用指针去扫右端点，合法左端点通过跳 parent 树得到。

可以预处理出每个节点祖先中 $\ge k$ 节点的最大长度，然后就可以直接求。

```cpp
	for(int i=1;i<=n;++i){
		int len=st[i].length(),nw=1;
		long long ans=0;
		for(int j=0;j<len;++j){
			char c=st[i][j]-'a';
			while(nw&&!S.T[nw].sn[c])nw=S.T[nw].fa;
			if(!S.T[nw].sn[c])nw=1;
			else nw=S.T[nw].sn[c];
            // 双指针
			int cnw=up[nw];// 祖先中>=k 节点最深，也就是长度最大的节点
			ans+=S.T[cnw].len;
		}
		cout<<ans<<' ';
	}
```

#### 例题 2.2

[[CmdOI2019] 口头禅](https://www.luogu.com.cn/problem/P5576)

有很多串 $s_1,s_2,\cdots,s_n$。

每次询问给定区间 $l,r$，问对于 $l,r$ 区间的所有串的最长公共子串长度。

考虑将询问离线挂在右端点，然后从左向右移动右端点。

这样可以对于每一个等价类维护一个 $lst$，表示每个等价类上一次没有出现的位置。

则若询问的左端点在 $lst$ 后面，则这个等价类里的串是公共子串，有贡献。

每次加入一个串，暴力跳 parent 树，并在数据结构 $lst+1$ 位置加入节点的贡献。

那些没有遍历到的点的 $lst$ 就为 $i$。

但是不能暴力修改这些点，我们可以对那些遍历到的点打一个 tag。

等到下一次跳到这个点时如果没有 tag，那么这个点在上一个串中没有出现，说明 $lst=i-1$。

然后移动右端点的时候需要将数据结构清空，这个也可以用打 tag 的技巧解决。

如果这个数据结构用线段树，那么时间复杂度是 $O(n\sqrt{n}\log n)$，无法通过。

用 $O(1)-O(\sqrt{n})$ 的分块做到 $O(n\sqrt{n})$，精细实现可以通过。

### SAM 其他技巧应用

- ### 线段树合并

这个玩意常常用来维护 SAM 中节点的 endpos 集合，当然，也可以维护一些其他信息。

维护这个 endpos 有什么用呢？

#### 例题 3.1

[[HEOI2016/TJOI2016] 字符串  ](https://www.luogu.com.cn/problem/P4094)

一道比较经典的题目。

注意到本题让我们求最长公共前缀，可以将整个串翻转，将问题转化为后缀问题。

先用线段树合并来预处理出每个节点的 $\text{endpos}$  集合。

这里需要注意，由于我们需要查询每个节点的信息，

线段树合并时要新开节点，不能覆盖以前的节点。

求出 $\text{endpos}$  之后，考虑哪一些串会是询问区间的后缀。

显然，如果我们找到了询问区间在 $\text{SAM}$  上的对应节点，

那么只有它在 $\text{parent}$ 树上的祖先才会可能是后缀（跳 $\text{parent}$ 树相当于变为原来后缀）

而通过线段树合并得到的 $\text{endpos}$  ，我们又可以知道一个节点所代表的串在原串中的出现位置。

利用这个，我们可以轻松知道该串出现在原串一个区间中的最长后缀。

具体地，设要求的区间是 $[l,r]$。

则设 $maxpos$ 为该节点 $\text{endpos}$ 集合中最大的小于等于 $r$ 的数。

则出现的最长后缀为 $\min{(maxpos-l+1,len)}$，其中 $len$ 表示该等价类中最长字符串长度。

发现随着在 $\text{parent}$ 树上向上跳，$endpos$ 集合会增大，因此对于任意的 $r$，$maxpos$ 都会单调不降。

因此 $maxpos-l+1$ 单调不降。

而 $len$ 又单调不增，如果将这两个画成函数，那么在交点出会取到  $\min{(maxpos-l+1,len)}$ 最大值

我们只需要找到最浅的节点，使得 $len\ge maxpos-l+1$。

设这个点是 $x$，我们将 $x$ 和 $link_x$ 所对应的最长后缀带进去算取 $\min$ 即可。

至于如何找到这个节点，可以通过倍增配合在线段树上查询做到 $O(n\log^2 n)$。

- ### LCT

LCT 常常用来维护 parent 树上的信息。

### 例题 3.2

[SubString](https://www.luogu.com.cn/problem/P5212)

强制在线，加字符，询问一个字符串出现次数。

由于强制在线，不能把树提前建出来，只能动态建树。

发现询问出现次数需要知道每个节点的 size。

然后加入一个节点相当于使一个节点到根节点路径上所有节点的 size $+1$。

直接用 LCT 做链加即可。（口胡，没写

#### 例题 3.3

[区间本质不同子串个数](https://www.luogu.com.cn/problem/P6292)

仍然考虑离线。

有一个经典的统计区间出现过的数的数量的 Trick：

对于每一个数维护这个数上次出现的位置 $lst_i$。

则新加入一个数 $a_{pos}$ 之后， $[lst_{a_{pos}}+1,pos]$ 中出现不同数的数量都会 $+1$。

可以转化为在 $lst_{a_{pos}}$ 单点 $-1$ 减去前面算过的贡献，在 $pos$ 单点 $+1$。

然后 $lst_{a_{pos}}$ 就会变为 $pos$。

询问的时候进行区间求和即可。

对于 SAM 的每一个等价类，这也是类似的。

维护这个等价类上一次出现的位置。

则等价类中长度为 $i$ 的字符串上一次出现的位置是 $[lst-i+1,lst]$。

这里只讨论在 $lst$  的位置 $-1$ 减去算过的贡献的情况，在 $pos$ 位置 $+1$ 的情况同理。

可以发现一个等价类中字符串开始的位置是一段连续的区间。

设一个等价类中最长的字符串是 $longest$，最短的为 $shortest$。

那么相当于在 $[lst-|longest|+1,lst-|shortest|+1]$ 区间 $-1$。

询问的时候同样可以区间查询。

每次右端点向右移动一格，加入一个字符。

则这个字符在 parent 树对应的节点到根节点路径上的节点，

对应的等价类 $lst$ 都会被修改为 $pos$ 并产生贡献。

直接暴力跳 parent 树时间复杂度错误。

发现 $lst$ 修改的过程和 LCT 链染色很像。

我们用 LCT 来进行这个染色的操作，这样 LCT 的每个 Splay 中都有一堆等价类。

因为 LCT 的 Splay 中的节点都成祖孙关系，这在 SAM 上又正好有很好的性质。

一个 LCT 的 Splay 中 $lst$ 都相等并且字符串长度都是连续的，和单个等价类差不多。

套用 LCT 的复杂度分析，这样最多进行 $O(n\log n)$ 次修改。

用线段树/树状数组维护区间加，区间求和，时间复杂度 $O(n\log ^2 n+q\log n)$。

- ### 点分治

#### 例题 3.4

[Ж-function](https://www.luogu.com.cn/problem/CF1098F)

*3500，很吓人。

lcp 不好做，考虑翻转原串，建反串的 SAM，这样两个节点的 lcp 长度就是 parent 树上两点的 LCA 的 $len$，与两个串的长度取 $\min$。

这个问题很像区间 LCA，如果你看过 【数据结构】线段树结构相关的问题 的话，就会发现这道题和 ZJOI 的那道题比较相似，应该也可以通过与其中第一种方法类似的方法上数据结构解决。

关于 LCA 有一个有趣的性质：LCA 是 $x$ 到 $y$ 路径上深度最浅的节点，于是这样就转化为了路径查询问题。

然后先咕咕咕着，之后再补。

























































































































































































































































