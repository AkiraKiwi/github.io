---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-419599.jpg
navigation: True
title: "CodeForces Round #396 (Div. 2) E"
date:  2017-10-02 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Problem Information

source: [It's from the Round #396 (Div. 2) of the codeforces](http://codeforces.com/contest/766/problem/E)

## Description

在一颗 \\(n\\) 个结点的树上，每个结点带有一个权值，定义从结点 \\(x\\) 到结点 \\(y\\) 所经过的所有节点的权值的异或值为这条路径的长度（ \\(x≤y\\) ），求树上所有路径的长度和。 <br>

## Input
```
3
1 2 3
1 2
2 3
5
1 2 3 4 5
1 2
2 3
3 4
3 5
```

## Output
```
10
52
```

## Solution

又是一道贡献计算的题，按套路走就是分别统计每个节点对最终答案的贡献。因为路径的起点小于等于终点，意味着每个点对只会被计算一次，那么可以取任意一个点作为根节点，深搜一遍即可。<br>

那么对于每个点对来说，它能贡献的路径就是不同子树的节点之间组合出来的所以路径数，可以按照DFS序保证不重不漏；它所作出的贡献就是它每一位上的 \\(0 / 1\\) 的个数与子树内节点异或后传上来的在这一位上的 \\(0 / 1\\) 的个数相乘得到该位的值在这棵子树上贡献的次数。<br>

对于每个节点，在回溯的时候，根据当前节点的值在每一位上累加子树节点里\\(0 / 1\\) 的个数：<br>
定义 \\( DP\[u\]\[j\]\[0/1\] \\) 为第 \\(u\\) 个节点在第 \\(j\\) 位上异或得到 \\(0/1\\) 的路径数（从节点\\(u\\) 出发到它的子树）<br>
* 当前节点的第 \\(j\\) 位是 \\(1\\) ：\\( DP\[u\]\[j\]\[1\] += DP\[v\]\[j\]\[0\]; \\ DP\[u\]\[j\]\[0\] += DP\[v\]\[j\]\[1\]; \\) <br>
* 当前节点的第 \\(j\\) 位是 \\(0\\) ：\\( DP\[u\]\[j\]\[0\] += DP\[v\]\[j\]\[0\]; \\ DP\[u\]\[j\]\[1\] += DP\[v\]\[j\]\[1\]; \\) <br>

## AC code
```c++
int n,a[MaxN];
struct Edge
{
    int u,v,next;
}edge[MaxM];
int cont,head[MaxN];
void add(int u, int v)
{
    edge[cont].u = u, edge[cont].v = v, edge[cont].next = head[u], head[u] = cont++;
}
LL DP[MaxN][20][2],ans;
void DFS(int u, int pre)
{
    int pos = 0;
    LL val = a[u];
    for(int i = 0; i < 20; i++)
    if(val & (1 << i)) DP[u][i][1]++;
    else DP[u][i][0]++;   
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v = edge[i].v;
        if(v==pre) continue;
        DFS(v,u);
        for(int j=0;j<20;j++)
        {
            ans += (DP[u][j][1]*DP[v][j][0])*(1<<j);
            ans += (DP[u][j][0]*DP[v][j][1])*(1<<j);

            if( (val&(1<<j)) )
            {
                DP[u][j][1] += DP[v][j][0];
                DP[u][j][0] += DP[v][j][1];
            }
            else 
            {
                DP[u][j][0] += DP[v][j][0];
                DP[u][j][1] += DP[v][j][1];
            }
        }
    }
}
void init()
{
    cont = 0; MST(head,-1);CLR(DP);
}
int main()
{
    while(~scanf("%d", &n))
    {
        init();
        ans = 0;
        for(int i=1;i<=n;i++) scanf("%d", &a[i]), ans+=a[i];
        int u,v;
        for(int i=1;i<n;i++)
        {
            scanf("%d%d", &u, &v);
            add(u,v);add(v,u);
        }
        DFS(1,1);
        out(ans); puts("");
    }
}
```