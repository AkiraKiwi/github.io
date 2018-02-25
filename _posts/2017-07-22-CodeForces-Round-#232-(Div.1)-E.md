---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-419599.jpg
navigation: True
title: "CodeForces Round #232 (Div. 1) E"
date:  2017-07-22 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
--- 

## Problem Information

source: [It's from the Round #232 (Div. 1) of the codeforces](http://codeforces.com/contest/396/problem/C)

## Description

给出一棵以 \\(1\\) 为根的有根树，有两种操作：<br>

*  \\( 1\\ v\\ x\\ k\\) : 给节点 \\(v\\) 的值加上 \\(x\\)，给它的子节点（和 \\(v\\) 的深度差为 \\(i\\) ） 加上 \\(x - k\\times i\\) <br>

*  \\( 2\\ v \\) : 查询 \\(v\\) 的值 <br>


You are given a rooted tree consisting of \\(n\\) vertices numbered from \\(1\\) to \\(n\\). The root of the tree is a vertex number \\(1\\). <br>

Initially all vertices contain number \\(0\\). Then come \\(q\\) queries, each query has one of the two types: <br>

* The format of the query: \\(1\\ v\\ x\\ k \\) . In response to the query, you need to add to the number at vertex \\(v\\) number \\(x\\); to the numbers at the descendants of vertex \\(v\\) at distance \\(1\\), add \\(x - k\\); and so on, to the numbers written in the descendants of vertex \\(v\\) at distance \\(i\\), you need to add \\(x - (i\\times k)\\). The distance between two vertices is the number of edges in the shortest path between these vertices. <br>

* The format of the query: \\(2\\ v\\) . In reply to the query you should print the number written in vertex \\(v\\) modulo \\(1000000007\\ (10^9 + 7)\\).<br>

Process the queries given in the input. <br>

## Input

The first line contains integer \\(n\\ (1 ≤ n ≤ 3 \\times 10^5) \\)  — the number of vertices in the tree. The second line contains \\(n - 1\\) integers \\(p\_2, p\_3, ... p\_n\\ (1 ≤ p\_i < i) \\), where \\(p\_i\\) is the number of the vertex that is the parent of vertex \\(i\\) in the tree. <br>

The third line contains integer \\( q\\ (1 ≤ q ≤ 3\\times 10^5) \\) — the number of queries. Next \\(q\\) lines contain the queries, one per line. The first number in the line is type. It represents the type of the query. If type = 1, then next follow space-separated integers \\(v, x, k\\ (1 ≤ v ≤ n; 0 ≤ x < 10^9 + 7; 0 ≤ k < 10^9 + 7) \\). If type = 2, then next follows integer \\(v\\ (1 ≤ v ≤ n) \\) — the vertex where you need to find the value of the number.<br>


## Output

For each query of the second type print on a single line the number written in the vertex from the query. Print the number modulo \\( 1000000007 (109 + 7) \\).

## Sample Input

~~~
3
1 1
3
1 1 2 1
2 1
2 2
~~~

## Sample Output
~~~
2
1
~~~

## Solution

首先，我们把整棵树按 \\(DFS\\) 序列压平成一个数组，然后对于修改操作：<br>
\\( \\det = x -  i \\times k = x - ( dep[v] - dep[u]) \\times k = x + dep[u] \\times k - dep[v] \\times k \\) ，在该式中，\\(u\\) 的整个子树增加的值 \\( x + dep[u] \\times k\\) 是固定的，整个子树减少的值 \\( dep[v] \\times k \\) 和每个节点有关，但是有一部分是固定的，所以我们可以维护两个树状数组，做区间修改和点查询。  <br>

## AC code
{% highlight c++ %}

int n;
struct Edge
{
    int u, v, next;
}edge[MaxM];
int cont, head[MaxN];
void add(int u, int v)
{
    edge[cont].u = u, edge[cont].v = v, edge[cont].next = head[u], head[u] = cont++;
}
int dep[MaxN],L[MaxN],R[MaxN],dfn;
void DFS(int u, int fa,int d)
{
    dep[u] = d;
    L[u] = ++dfn;
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v = edge[i].v;
        if(v==fa) continue;
        DFS(v, u, d+1);
    }
    R[u] = dfn;
}
struct BIT{
	int n;
	LL tree[MaxN];
	void init( int n ){
		this->n = n ;
		CLR(tree);
	}
	
	int lowbit( int x ){
		return x & ( -x );
	}
	
	LL sum( int n ){
		LL ans = 0;
		for( int i = n; i ; i -= lowbit(i) ){
			ans = ( ans + tree[i] ) % mod ;
		}
		return ans;
	}
	
	void update( int x, LL val ){
		for( int i = x; i <= n; i += lowbit( i ) ){
			tree[i] = ( tree[i] + val ) % mod ;
		}
	}
}Add,Sub;
void solve()
{
    dfn = 0;
    DFS(1,1,1);
    Add.init(dfn);
    Sub.init(dfn);
}

void init()
{
    cont = 0, MST(head, -1);
}
int main()
{
    //std::ios::sync_with_stdio(false);
    while(~scanf("%d", &n))
    {
        init();
        int f,op,v,x,k;
        for(int i=2;i<=n;i++)
        {
            scanf("%d", &f);
            add(f, i);
            add(i, f);
        }
        solve();
        scanf("%d", &f);
        while(f--)
        {
            scanf("%d", &op);
            if(op==1)
            {
                scanf("%d%d%d", &v, &x, &k);
                LL val = 1LL*dep[v]*k+x;
                Add.update(L[v],   val);
                Add.update(R[v]+1,-val);

                Sub.update(L[v],   k);
                Sub.update(R[v]+1,-k);
            }
            else 
            {
                scanf("%d", &v);
                LL ans = (Add.sum(L[v]) - (LL)dep[v]*Sub.sum(L[v])%mod);
                printf("%lld\n", ((ans%mod)+mod)%mod );
            }
        }
    }
}

{% endhighlight %}


