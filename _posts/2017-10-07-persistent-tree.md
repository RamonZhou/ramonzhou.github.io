---
layout: post
title: '主席树学习'
date: 2017-10-09
author: Deluxurous
cover: '/assets/img/data-structure.png'
categories: OI
tags: Data_Structure Persistent
latest: 2017-10-13
---

> 蒟蒻学主席树

注：本文含有公式，显示可能较慢。

## 引入

思考这样一个问题:

> 给定 $N$ 个正整数构成的序列，将对于指定的闭区间查询其区间内的第 $K$ 小值。
>
> 题目链接：[Luogu 3834](https://www.luogu.org/problem/show?pid=3834)

可以想到一种树套树的做法（我不会写不会写不会写不会写不会写不会写

但这个做法不是现在要讲的。我们这样考虑：对于数列的每一个前缀 $[1,n]$ 建立一颗权值线段树。线段树的每个节点存储的是在这个前缀里权值在 $[l,r]$ 内的点的数量。这样在搜索的时候，对于查询的区间 $[Q_l,Q_r]$ ，就可以用 $[1,Q_r]$ 区间的答案减去 $[1,Q_l-1]$ 区间的答案从而得到所查询的区间的答案。而具体查询第 $K$ 大可以用二叉搜索的思想。对于一个**权值**区间 $[Q_l,Q_r]$ ，我们想查询这个区间中的第 $k$ 小，可以先计算区间 $[Q_l,Q_{mid}]$ 中有多少个数，从而得知这个第 $k$ 小是在左半个区间还是在右半个区间中，然后递归搜索。这样时间复杂度是 $O(n \log n)$ ，空间复杂度是 $O(n^2)$ 的。

那么问题来了。空间爆炸怎么办？

这就要用到玄妙的主席树。可以发现对于前缀 $[1,x]$ 建立的一颗线段树，它只比前缀 $[1,x-1]$ 的线段树多增加了一个值，也就是其实两颗线段树之间的不同的节点数量只有 $\log n$ 个。所以每次可以只新建 $\log n$ 个节点。那么空间复杂度就降到了 $O(n \log n)$ 啦，是不是妙妙啊。

虽然这是个**可持久化线段树**，但是它的作用就是支持查询历史版本，也就是会保存每次修改。那么在这里，每颗线段树就等于是在前一颗的基础上做了一次修改，所以可以用主席树来降低空间复杂度。

## 实现细节

1. 值域一般较大，需要离散化。

2. 查询答案时可以同时传递两个线段树的节点位置，这样比较方便。

具体代码实现：

<pre class="line-numbers"><code class="language-cpp">
    #include <bits/stdc++.h>
    using namespace std;
    const int MAXN=200010, TOTN=MAXN*18;

    // a数组保存权值，ver数组保存每颗线段树的根节点位置
    int a[MAXN],ver[MAXN];
    int n,m,clk,tot,cnt;
    struct temp_data {
        int x,y;
    } ord[MAXN];
    bool cmp(temp_data a,temp_data b) {
        return a.x<b.x;
    }

    // 使用动态开点，需要保存每个节点的左孩子和右孩子
    struct tree_node {
        int v,l,r;
    } seg[TOTN];

    /*
      build(): 建一颗空线段树
      rt: 当前节点
      l: 当前权值区间左端点
      r: 当前权值区间右端点
    */
    inline void build(int& rt,int l,int r) {
        rt=++clk;
        if(l==r) return;
        int mid=(l+r)>>1;
        build(seg[rt].l,l,mid);
        build(seg[rt].r,mid+1,r);
    }

    /*
      insert(): 插入一个权值
      v: 插入的权值
      l: 当前权值区间左端点
      r: 当前权值区间右端点
      rt: 当前正在新建的线段树节点位置
      last: 前一个版本的线段树节点位置
    */
    inline void insert(int v,int l,int r,int& rt,int last) {
        rt=++clk;
        seg[rt].l=seg[last].l;
        seg[rt].r=seg[last].r;
        seg[rt].v=seg[last].v+1;
        if(l==r) return;
        int mid=(l+r)>>1;
        if(v<=mid) insert(v,l,mid,seg[rt].l,seg[last].l);
        else insert(v,mid+1,r,seg[rt].r,seg[last].r);
    }

    /*
      query(): 查询第k小
      k: 查询的名次
      l: 当前权值区间左端点
      r: 当前权值区间右端点
      x: 前缀[1,R]的线段树节点
      y: 前缀[1,L-1]的线段树节点
    */
    inline int query(int k,int l,int r,int x,int y) {
        if(l==r) return l;
        int now=seg[seg[y].l].v-seg[seg[x].l].v, mid=(l+r)>>1;
        if(k<=now) return query(k,l,mid,seg[x].l,seg[y].l);
        else return query(k-now,mid+1,r,seg[x].r,seg[y].r);
    }

    int main() {
        scanf("%d%d",&n,&m);
        for(int i=1;i<=n;i++) {
            scanf("%d",&a[i]);
            ord[i]=(temp_data){a[i],i};
        }
        clk=0;

        // 离散化：
        sort(ord+1,ord+1+n,cmp);
        tot=cnt=0;
        for(int i=1;i<=n;i++) {
            if(i==1||ord[i].x!=ord[i-1].x) {
                a[ord[i].y]=++tot;
                ord[tot]=ord[i];
            } else {
                a[ord[i].y]=tot;
            }
        }
        build(ver[0],1,tot); // 先建一颗空线段树
        for(int i=1;i<=n;i++) {
            insert(a[i],1,tot,ver[i],ver[i-1]);
            // 每次在前一颗线段树上插入一个数，得到一颗新树
        }
        for(int i=1,par1,par2,par3;i<=m;i++) {
            scanf("%d%d%d",&par1,&par2,&par3);
            printf("%d\n",ord[query(par3,1,tot,ver[par1-1],ver[par2])].x);
            // 输出答案时记得把离散后的值映射回来。
        }
        return 0;
    }
</code></pre>

## 另一个例题

> 给定一棵N个节点的树，每个点有一个权值，对于M个询问，你需要回答两个节点间路径上第 $K$ 小的点权。强制在线。
>
> 题目链接：[Luogu 2633](https://www.luogu.org/problem/show?pid=2633) [BZOJ 2588](http://www.lydsy.com/JudgeOnline/problem.php?id=2588)

还是前缀查分的思想。我们可以维护每个点到根节点路径上的答案，那么对于一条路径 $u \leftrightarrow v$ 上的答案，即为 $S_{u}+S_{v}-
S_{lca}-S_{fa[lca]}$ 的值。（其中 $S_x$ 即为维护节点 $x$ 到根节点路径上信息的线段树）

## 带修改主席树

那如果还要在线修改这些权值怎么办呢？

考虑这个问题：

>给定一个含有n个数的序列，程序需要支持动态修改某个点的权值，同时查询区间第 $K$ 小。
>
>题目链接：[Luogu 2617](https://www.luogu.org/problem/show?pid=2617) [BZOJ 1901](http://www.lydsy.com/JudgeOnline/problem.php?id=1901)

由于每颗线段树保存的是数列前缀的信息，所以如果修改了某个节点 $a$ 的权值，只会影响前缀 $[1,a]$ 及以后的线段树，也就是说，每次修改权值只是对这些线段树的**后缀**添加了修改。那么这些前缀后缀的修改，当然应该由树状数组来完成。所以我们可以把这 $n$ 棵线段树以树状数组的方式储存（所以每颗线段树可能不止保存一个前缀咯），每次对点 $a$ 的修改可以化费 $\log n$ 的时间应用到 $a$~$n$ 的线段树上。那么时间复杂度就是 $O(n \log ^2 n)$ 的。

## 实现细节

1. 这道题目中修改权值是以直接替换的方式出现的，所以可以在读入时保存所有会出现的权值，一起离散化。否则如果只离散原数组的话，修改后的权值若不存在原数组中，就会难以完成修改。

2. 应用修改时先将原权值减去，再插入新权值。

3. 在查询时，由于我们使用树状数组，不能直接将两棵线段树直接相减，但还是应当将两个前缀相减，所以用树状数组求前缀和的方式，记录下组成两个前缀的线段树（下面的代码中使用f和g数组），然后每次用一些线段树的和减掉另一些线段树的和就可以了。这些线段树最多有 $\log n$ 级别个，所以复杂度可以接受。

具体看代码吧

<pre class="line-numbers"><code class="language-cpp">
    #include <bits/stdc++.h>
    using namespace std;
    const int MAXN=10010, TOTN=MAXN*500;
    struct oper {
        int ty,l,r,v;
    } Q[MAXN];
    struct treenode {
        int v,l,r;
    } seg[TOTN];
    int a[MAXN],f[MAXN],g[MAXN],ver[MAXN],ord[MAXN<<1];
    int n,m,tot,clk=0,topf,topg;
    char op[3];

    /*
      update(): 更新线段树信息
      v: 要修改的权值
      l: 当前权值区间左端点
      r: 当前权值区间右端点
      rt: 当前正在新建的线段树节点位置
      last: 前一个版本的线段树节点位置
      delta: 修改的值（本题为+1或-1）
    */
    inline void update(int val,int l,int r,int& rt,int last,int delta) {
        rt=++clk;
        seg[rt].v=seg[last].v+delta;
        seg[rt].l=seg[last].l;
        seg[rt].r=seg[last].r;
        if(l==r) return;
        int mid=(l+r)>>1;
        if(val<=mid) update(val,l,mid,seg[rt].l,seg[last].l,delta);
        else update(val,mid+1,r,seg[rt].r,seg[last].r,delta);
    }

    /*
      insert(): 插入或删除权值
      idx: 改动的点在原数组中的位置
      delta: 修改的值（本题为+1或-1）
    */
    inline void insert(int idx,int delta) {
        int pos=lower_bound(ord+1,ord+1+tot,a[idx])-ord; // 得到离散值
        for(int i=idx;i<=n;i+=i&(-i)) {
            // 用树状数组对一个后缀进行修改
            update(pos,1,tot,ver[i],ver[i],delta);
        }
    }

    /*
      query(): 查询区间第k小
      k: 查询的名次
      l: 当前权值区间左端点
      r: 当前权值区间右端点
    */
    inline int query(int rnk,int l,int r) {
        if(l==r) return l;
        int now=0;

        // 用f中线段树减去g中线段树
        for(int i=1;i<=topg;i++) {
            now+=seg[seg[g[i]].l].v;
        }
        for(int i=1;i<=topf;i++) {
            now-=seg[seg[f[i]].l].v;
        }

        int mid=(l+r)>>1;
        // 更新线段树节点
        if(rnk<=now) {
            for(int i=1;i<=topf;i++) {
                f[i]=seg[f[i]].l;
            }
            for(int i=1;i<=topg;i++) {
                g[i]=seg[g[i]].l;
            }
            return query(rnk,l,mid);
        } else {
            for(int i=1;i<=topf;i++) {
                f[i]=seg[f[i]].r;
            }
            for(int i=1;i<=topg;i++) {
                g[i]=seg[g[i]].r;
            }
            return query(rnk-now,mid+1,r);
        }
    }

    int main() {
        scanf("%d%d",&n,&m);
        for(int i=1;i<=n;i++) {
            scanf("%d",&a[i]);
            ord[i]=a[i];
        }
        tot=n;
        for(int i=1;i<=m;i++) {
            scanf("%s%d%d",op,&Q[i].l,&Q[i].r);
            if(op[0]=='Q') {
                Q[i].ty=1;
                scanf("%d",&Q[i].v);
            } else {
                Q[i].ty=2;
                ord[++tot]=Q[i].r;
            }
        }

        // 离散并建树：
        sort(ord+1,ord+1+tot);
        tot=unique(ord+1,ord+1+tot)-ord-1;
        for(int i=1;i<=n;i++) {
            insert(i,1);
        }

        // 处理操作
        for(int i=1;i<=m;i++) {
            if(Q[i].ty==1) {
                // 取出构成两个前缀的线段树
                topf=topg=0;
                for(int j=Q[i].l-1;j;j-=j&(-j)) {
                    f[++topf]=ver[j];
                }
                for(int j=Q[i].r;j;j-=j&(-j)) {
                    g[++topg]=ver[j];
                }
                printf("%d\n",ord[query(Q[i].v,1,tot)]);
            } else {
                insert(Q[i].l,-1); // 先删除原数
                a[Q[i].l]=Q[i].r;
                insert(Q[i].l,1); // 再插入新数
            }
        }
        return 0;
    }
</code></pre>

<br/>

差不多讲完了……感觉我好菜啊