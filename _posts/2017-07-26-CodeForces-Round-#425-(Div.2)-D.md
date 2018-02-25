---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-419599.jpg
navigation: True
title: "CodeForces Round #425 (Div. 2) D"
date:  2017-07-26 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
--- 

## Problem Information

source: [It's from the Round #425 (Div. 2) of the codeforces](http://codeforces.com/contest/832/problem/D)

## Description

给出一棵 \\(n\\) 个节点的树，执行 \\(q\\) 次查询，每次从中挑出三个节点，选择一个为终点使得另外两个点到终点的路径重合的点数最多，输出这个点数。<br>

Misha and Grisha are funny boys, so they like to use new underground. The underground has \\(n\\) stations connected with \\(n-1\\) routes so that each route connects two stations, and it is possible to reach every station from any other.<br>

The boys decided to have fun and came up with a plan. Namely, in some day in the morning Misha will ride the underground from station \\(s\\) to station \\(f\\) by the shortest path, and will draw with aerosol an ugly text "Misha was here" on every station he will pass through (including \\(s\\) and \\(f\\)). After that on the same day at evening Grisha will ride from station \\(t\\) to station \\(f\\) by the shortest path and will count stations with Misha's text. After that at night the underground workers will wash the texts out, because the underground should be clean.<br>

The boys have already chosen three stations \\(a\\), \\(b\\) and \\(c\\) for each of several following days, one of them should be station \\(s\\) on that day, another should be station \\(f\\), and the remaining should be station \\(t\\). They became interested how they should choose these stations \\(s\\), \\(f\\), \\(t\\) so that the number Grisha will count is as large as possible. They asked you for help.<br>

## Input

The first line contains integer \\(n\\ (1 ≤ n ≤ 3 \\times 10^5) \\)  — the number of vertices in the tree. The second line contains \\(n - 1\\) integers \\(p\_2, p\_3, ... p\_n\\ (1 ≤ p\_i < i) \\), where \\(p\_i\\) is the number of the vertex that is the parent of vertex \\(i\\) in the tree. <br>

The first line contains two integers \\(n\\) and \\(q\\ (2 ≤ n ≤ 10^5, 1 ≤ q ≤ 10^5)\\) — the number of stations and the number of days.<br>

The second line contains \\(n - 1\\) integers \\(p\_2, p\_3, ..., p\_n\\ (1 ≤ p\_i ≤ n)\\). The integer \\(p\_i\\) means that there is a route between stations \\(p\_i\\) and \\(i\\). It is guaranteed that it's possible to reach every station from any other.<br>

The next \\(q\\) lines contains three integers \\(a, b, c\\) each \\((1 ≤ a, b, c ≤ n)\\) — the ids of stations chosen by boys for some day. Note that some of these ids could be same.<br>


## Output

Print \\(q\\) lines. In the i-th of these lines print the maximum possible number Grisha can get counting when the stations \\(s, t, f\\) are chosen optimally from the three stations on the \\(i\\)-th day.

## Sample Input

~~~
3 2
1 1
1 2 3
2 3 3
~~~

## Sample Output
~~~
2
3
~~~

## Solution

首先用倍增法求预处理然后可以在线求任意两个结点的 \\(Lca\\)，对于每一组查询，我们枚举每个点作为终点，那么每次有两种情况：<br>

* 终点在两个起点的 \\(Lca(b,c)\\) 的子树内：共同路径节点数就是 \\(Lca(a,b), Lca(a,c)\\) 中不等于 \\(Lca(b,c)\\) 的节点到终点的路径的节点数

* 终点在两个起点的 \\(Lca(b,c)\\) 的子树外：共同路径节点数就是 \\(Lca(b,c))\\) 到 \\(a\\) 的路径的节点数

## AC code
{% highlight c++ %}
const int log2n = 20;
int n,q;
struct Edge
{
    int u,v,next;
}edge[MaxM];
int cont, head[MaxN];
void add(int u, int v)
{
    edge[cont].u = u, edge[cont].v = v, edge[cont].next = head[u], head[u] = cont++;
}

int dep[MaxN], pre[MaxN][log2n], fa[MaxN], dfn[MaxN], low[MaxN], clo;
void dfs(int u)
{
	pre[u][0] = fa[u], dfn[u] = low[u] = ++clo;
	for (int i=head[u]; i!=-1; i=edge[i].next)
	{
        int v = edge[i].v;
        dep[v] = dep[u]+1;
        dfs(v);
        low[u] = max(low[u], low[v]);
    }
    //debug(dep[u])
}

void lcabinarylifting(int n)
{
	for(int j=1;(1<<j)<=n;++j)
		for(int i=1;i<=n;++i)
			if(pre[i][j-1]!=-1) pre[i][j]=pre[pre[i][j-1]][j-1];   ///2^j=2^(j-1)+2^(j-1)

}

inline int lca(int x,int y)
{
    int D = 0;
    if(dep[x]<dep[y])swap(x,y);
    while( (1<<D) <= dep[x])D++;
    D--;
    for(int i =D;i>=0;i--) if(dep[x]-(1<<i)>=dep[y]) x=pre[x][i];
    if(x==y) return x;
    for(int i=D;i>=0;--i)
        if(pre[x][i]!=-1 && pre[x][i]!=pre[y][i])
        {
            x=pre[x][i];y=pre[y][i];
        }
    return pre[x][0];
}
int ans;
void cal(int a,int b,int c)
{
	int x=lca(b,c);
	int sum=0;
	if(dfn[a]>=dfn[x] && dfn[a]<=low[x])
	{
		int y=lca(a,b),z=lca(a,c); 
		if(y==x) sum=dep[a]-dep[z]+1;
		else sum=dep[a]-dep[y]+1;
	}
	else
	{
		int y=lca(x,a);
		sum=dep[x]+dep[a]-2*dep[y]+1;
	}
	ans=max(ans,sum);
}

void solve()
{
    clo = 0;
    dfs(1);
    lcabinarylifting(n);
}
void init()
{
    cont = 0, MST(head,-1);
    fa[1] = -1, dep[1] = 0;
    MST(pre,-1);
}
int main()
{
    while(~scanf("%d%d", &n, &q))
    {
        init();
        for(int u=2;u<=n;u++)
        {
            scanf("%d", &fa[u]);
            add(fa[u],u);
        }
        solve();
        int a, b, c;
        while(q--)  
        {
            scanf("%d%d%d", &a, &b, &c);
            ans = 0;
            cal(a,b,c);
            cal(b,a,c);
            cal(c,a,b);
            printf("%d\n", ans);
        }
    }
}

{% endhighlight %}


