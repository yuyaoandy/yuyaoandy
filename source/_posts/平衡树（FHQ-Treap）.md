---
title: 平衡树FHQ-Treap
date: 2026-07-29 15:26:09
tags: [算法竞赛]
---

本文主要讲解FHQ-Treap这种平衡树的写法和应用。
<!-- more -->
FHQ Treap（无旋 Treap）是一种非常好写好用，支持可持久化的平衡树。

Treap 就是 Tree 和 Heap（堆） 的合体，因此 Treap 上的每个节点有两个权值，其中一个权值 $value$（也就是我们插入的权值） 满足 Tree 的性质，另外一个权值 $weight$ 满足 Heap 的性质（也就是父节点的 $weight$ 严格大于左右儿子）。

对于一般的二叉搜索树而言，在特定的数据下会变成 $O(n^2)$ 的原因就是因为树高太高了。于是每次进行操作的时候就要“爬”整一个树，导致时间复杂度假掉，一般平衡树都是通过某些方式限制树高。

而 Treap 也是这样，一个随机的二叉堆的高度期望是 $O(\log n)$ 的。

所以如果如果我们给每次插入的节点都赋上一个随机权值，那么 Treap 的期望高度也就是 $O(\log n)$ 的。

问题就是如何在操作后使得 Treap 仍然满足这些性质。

FHQ Treap 采用的是 Split（按照 $value$ 划分成两半） 和 Merge （将 $value$ 值有大小关系的两棵树按照堆的方式合并成为一棵） 的方式。

我们先来看看怎么进行 Split 操作。

如图，假如我们要按照分为 $\le 4$ 和 $>4$ 两个部分 

![](https://cdn.luogu.com.cn/upload/image_hosting/si2q3myf.png)

图中的红色部分就是 $>4$ 的节点，此时如果直接强行断掉红圈内和红圈外部的边，那么这棵树将会四分五裂。

![](https://cdn.luogu.com.cn/upload/image_hosting/khmbyjga.png)

但是其实还是挺有规律的，如果我们连上图中蓝色和绿色的边，然后再断开虚线处的边，那么整棵树就会分成两半。

对于一个节点，如果它的权值 $\le k$，那么它的左子树内部的节点权值一定是 $\le k$ 的，只有在它的右子树中可能出现 $>k$ 的节点，如果权值 $>k$ 那么右子树权值一定 $>k$，只有左子树可能出现 $\le k$ 的节点。

这样我们就能够递归下去进行分裂。然后我们以连接蓝色的边为例。当一个节点的权值 $\le k$ 且它的右子树里面有权值 $\le k$ 的节点的时候，我们从这个节点向它右子树里面满足条件的最浅的这个节点连边，这样递归下去所有满足条件的点都被连上了边。（在实际的代码中其实挺简洁的）

Merge 就更加好理解，为了满足堆的性质，每次我们必须将当前的两个节点中 $weight$ 比较大的那一个作为根节点（当然小根堆也是可以的）。假设当前作为根节点的节点（也就是 $weight$ 比较大的节点） $value$ 值比较小（大的时候同理），由于要合并上去的另外一棵树中的 $value$ 值都要大于当前的这个节点，所以那一棵子树只能放在当前节点的右儿子中，于是我们将另外一棵树和当前节点的右子树合并，这样从两棵树的根节点一直递归下去直到为空的时候就结束了。

容易发现，Split 和 Merge 都是 $O(Height)$ 也就是 $O(\log n)$ 的。

插入元素 $k$ 时，我们将树分成 $\le k$ 和 $>k$ 的两个部分，然后将元素 $k$ 新建立一个节点再和两部分分别合并起来（这样显然满足 Merge 的前置条件）。

删除元素 $k$ 时，我们通过两次 split 操作精准分离出恰好等于 $k$ 的这一部分，并删除这一棵树的根节点（这时候等于 $k$ 的这一棵树就被拆成两棵了），然后将所有四棵树合并起来。

查询元素排名，分离出 $<k$ 的子树，查询大小即可。

查询 $k$ 大：和二叉搜索树一样比较左右儿子的 sz 。

查询前驱，后继：分成两棵树然后暴力找两棵树最左/右的节点。

```cpp
#include<bits/stdc++.h>
using namespace std;
mt19937 rnd(chrono::system_clock::now().time_since_epoch().count());
struct Treap{
	int tot=0,rt=0;
	struct node{
		int sn[2],w,v,sz;
	}A[2000000];
	inline void push_up(int k){A[k].sz=A[A[k].sn[0]].sz+A[A[k].sn[1]].sz+1;}
	inline int newnode(int val){++tot;A[tot].sz=1;A[tot].v=val;A[tot].w=(int)rnd();return tot;}
	inline void split(int k,int val,int &x,int &y){
		if(!k)return x=y=0,void();//push_down()
		if(A[k].v<=val)x=k,split(A[k].sn[1],val,A[k].sn[1],y);
		else y=k,split(A[k].sn[0],val,x,A[k].sn[0]);
		push_up(k);
	}
	inline int merge(int x,int y,bool flag=0){
		//push_down()
		if(!x||!y)return x+y;
		if(A[x].w<A[y].w)swap(x,y),flag^=1;
		A[x].sn[flag^1]=merge(A[x].sn[flag^1],y,flag);push_up(x);
		return x;
	}
	inline void insert(int k){
		int tA=0,tB=0,tC=newnode(k);
		split(rt,k,tA,tB);
		rt=merge(merge(tA,tC),tB);
	}
	inline void erase(int k){
		int tA=0,tB=0,tC=0,tD=0;
		split(rt,k,tA,tB);split(tA,k-1,tC,tD);
		tA=merge(A[tD].sn[0],A[tD].sn[1]);rt=merge(merge(tC,tA),tB);
	}
	inline int rank(int k){
		int tA=0,tB=0,ans=0;split(rt,k-1,tA,tB);
		ans=A[tA].sz+1;rt=merge(tA,tB);
		return ans;
	}
	inline int kth(int k){
		int nw=rt;
		if(k>A[rt].sz)return -1;
		while(nw){
			if(k<=A[A[nw].sn[0]].sz)nw=A[nw].sn[0];
			else if(k==A[A[nw].sn[0]].sz+1)return A[nw].v;
			else k-=A[A[nw].sn[0]].sz+1,nw=A[nw].sn[1];
		}return -1;
	}
	inline int pre(int k){
		int tA=0,tB=0,ans=0;split(rt,k-1,tA,tB);
		int nw=tA;while(A[nw].sn[1])nw=A[nw].sn[1];
		ans=A[nw].v;rt=merge(tA,tB);
		return ans;
	}
	inline int nxt(int k){
		int tA=0,tB=0,ans=0;split(rt,k,tA,tB);
		int nw=tB;while(A[nw].sn[0])nw=A[nw].sn[0];
		ans=A[nw].v,rt=merge(tA,tB);
		return ans;
	}
}Tr;
```

由于 split 和 merge 的次数较多，常数比较大。



---



FHQ Treap 可以轻松支持文艺平衡树和可持久化。



对于文艺平衡树，我们只需要把下标当成元素插入到平衡树中，

然后平衡树的中序遍历就是整个序列。

此时由于区间翻转的原因，我们的“平衡树”并不满足二叉搜索树的性质，因此我们在普通平衡树中采用的按照元素权值来进行 split 的操作就并不适用了（而且我们也没有必要按照权值进行分裂是吧），需要按照 $siz$ 进行分裂（按照 $siz$ 的顺序可以分裂出序列中下标 $\le k$ 的元素），而 Merge 其实和权值 $value$ 无关，所以不用改动。

只要 split 出当前这一段区间然后再打上一份翻转的 tag 即可。

```cpp
void Split(int Node,int Sum,int &x,int &y){
	if(Node&&a[Node].tag)Pushdown(Node);
	if(!Node)x=y=0;
	else if(a[a[Node].sn[0]].sz<=Sum)x=Node,Split(a[Node].sn[1],Sum-a[a[Node].sn[0]].sz-1,a[Node].sn[1],y),Update(Node);
	else y=Node,Split(a[Node].sn[0],Sum,x,a[Node].sn[0]),Update(Node);
}
```

将文艺平衡树中“按照下标来 split” 的想法拓展，就可以得到用 FHQ-Treap 来进行区间修改区间查询的做法：

修改时将需要修改的区间通过两次 split 分裂成 $[1,L-1],[L,R],[R+1,n]$ 然后给 $[L,R]$ 所在的区间打上 tag。

查询的时候也可以通过两次 split 精准定位需要查询的区间。





对于可持久化也很简单，只需要在 split 和 merge 的时候将那些被修改掉的节点复制一份即可。

**记得在可持久化的时候如果有 tag 的下传的话，那么在下传 tag 的时候要复制节点**

可持久化文艺平衡树：

```cpp
struct Persistent_Treap{
	int tot=0,rt=0;
	struct node{
		int sn[2],w,v,sz;
		bool  tg;
		ll sum;
	}A[16000000];
	inline void push_down(int k){
		if(!A[k].tg)return;
		swap(A[k].sn[0],A[k].sn[1]);
		if(A[k].sn[0])A[++tot]=A[A[k].sn[0]],A[A[k].sn[0]=tot].tg^=1;
		if(A[k].sn[1])A[++tot]=A[A[k].sn[1]],A[A[k].sn[1]=tot].tg^=1;
		A[k].tg=0;
	}
	inline void push_up(int k){
		A[k].sz=A[A[k].sn[0]].sz+A[A[k].sn[1]].sz+1;
		A[k].sum=A[A[k].sn[0]].sum+A[A[k].sn[1]].sum+A[k].v;
	}
	inline int newnode(int val){++tot;A[tot].sz=1;A[tot].sn[0]=A[tot].sn[1]=A[tot].tg=0;A[tot].v=A[tot].sum=val;A[tot].w=(int)rnd();return tot;}
	inline void split(int k,int val,int &x,int &y){
		if(!k)return x=y=0,void();
		push_down(k);
		int nwnode=++tot;
		A[nwnode]=A[k];
		if(A[A[k].sn[0]].sz<val)x=nwnode,split(A[k].sn[1],val-A[A[k].sn[0]].sz-1,A[nwnode].sn[1],y);
		else y=nwnode,split(A[k].sn[0],val,x,A[nwnode].sn[0]);
		push_up(nwnode);
	}
	inline int merge(int x,int y,bool flag=0){
		if(!x||!y)return x+y;
		if(A[x].w<A[y].w)swap(x,y),flag^=1;
		push_down(x);
		int nwnode=++tot;
		A[nwnode]=A[x];
		A[nwnode].sn[flag^1]=merge(A[x].sn[flag^1],y,flag);
		push_up(nwnode);
		return nwnode;
	}
	inline void insert(int k,int rt1,int &rt2,int pos){
		int tA=0,tB=0,tC=newnode(k);
		split(rt1,pos,tA,tB);
		rt2=merge(merge(tA,tC),tB);
	}
	inline void erase(int rt1,int &rt2,int pos){
		int tA=0,tB=0,tC=0,tD=0;
		split(rt1,pos,tA,tB);split(tA,pos-1,tC,tD);
		rt2=merge(tC,tB);
	}
	inline void rever(int rt1,int &rt2,int L,int R){
		int tA=0,tB=0,tC=0,tD=0;
		split(rt1,L-1,tA,tB);
		split(tB,R-L+1,tC,tD);A[tC].tg^=1;
		rt2=merge(merge(tA,tC),tD);
	}
	inline ll query(int L,int R,int rt){
		int tA=0,tB=0,tC=0,tD=0;ll ans=0;
		split(rt,L-1,tA,tB);
		split(tB,R-L+1,tC,tD);
		ans=A[tC].sum;
		return ans;
	}
}Tr;
```





平衡树维护序列的区间复制操作。



由于我们抠出要复制的区间后并不能够直接遍历这个区间复制给另外一个区间。

这个时候就需要用到可持久化的操作，我们新开一个版本的平衡树，然后在在新版本的线段树中直接将要复制的区间剪切并粘贴到原版中需要粘贴的地方，但是这样如果我们预先给出权重会导致树高出问题，一种比较好的实现方式是在合并的时候有 $\frac{szA}{szA+szB}$ 的概率将 $A$ 节点作为根，时间复杂度未知，但是卡不掉。

当节点数量过多的时候需要定期重构序列，一般每 $O(\frac{n}{\log n})$ 次操作重构一次。

具体操作可以看 P5350 序列。