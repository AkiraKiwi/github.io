---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-419599.jpg
navigation: True
title: "CodeForces Round #436 (Div. 2) F"
date: 2017-10-03 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Problem Information

source: [It's from the Round #436 (Div. 2) of the codeforces](http://codeforces.com/contest/864/problem/F)

## Description

有一个 \\(n\\) 个结点，\\(m\\) 条边的有向图，其中没有自环。定义从 \\(s\\) 到 \\(t\\) 的一条路径为一个连续的点集 \\(  p\_1, p\_2, p\_3, ..., p\_k \\)，其中 \\(p\_1 = s, p\_k = t \\)，从\\(p\_i\\) 到 \\(p\_{i+1}\\) 存在一条有向边，其中 \\(t\\) 点只能出现一次。 <br>

现在给出 \\(q\\) 个查询，每次询问 \\(s\_i\\) 到 \\(t\_i\\) 字典序最小的那条路径上的第 \\(k\_i\\) 个点的标号。如果两点间无法到达，或者不存在字典序最小的路径则输出 \\(-1\\)。<br>

## Input
```
7 7 5                                                                                
1 2
2 3
1 3
3 4
4 5
5 3
4 6
1 4 2
2 6 1
1 7 3
1 3 2
1 3 5
```

## Output
```
2                                                                                     
-1
-1
2
-1

```

## Solution

建图时，同时建一个反向图，在建完原图后同时将原图中每个点能到达的节点排个序，以保证遍历时是按字典序最小的方式走的。<br>

那么对于每一个点，以它作为终点，在反向图中标记出所以它能抵达的点，即在原图中能够到达它的点，再遍历原图中每一个点 \\(u\\) 能到达的点 \\(v\\)，找到第一个被标记过的 \\(v\\)，
则 \\(v\\) 是 \\(u\\) 到当前终点的路径中的字典序最小的点，即将 \\(u\\) 保存在 \\(v\\) 的上一个节点。
最后从当前终点开始 DFS，用 vector 模拟栈来保存当前路径。每次走当前节点的上一个节点，即保证了当前路径永远是字典序最小的那个，
同时，也只会计算到该节点字典序最小的路径的答案，对于不可抵达和不是最小字典序的路径，是不会遍历到的。<br>

## AC code
```c++
int n,m,q;                                                                            
vector<int> G[MaxN],IG[MaxN];
vector< pair<int,int> > qs[MaxN][MaxN];
vector<int> Path, Fa[MaxN];
int ans[MaxM];
bool vis[MaxN];
void dfs(int u)
{
    if(vis[u]) return;
    vis[u] = 1;
    for(auto v : IG[u])
        dfs(v);
}

void cal(int t, int u)
{
    Path.pb(u);
    for(auto it : qs[t][u])
        if( it.first < Path.size() ) 
            ans[it.second] = Path[Path.size()-1 - it.first] + 1;
    for( auto fa : Fa[u])
    {
        cal(t, fa);
    }
    Path.pop_back();
}

void solve()
{
    MST(ans,-1);
    for(int i=0;i<n;i++)
    {
        CLR(vis);
        dfs(i);
        for(int j=0;j<n;j++) Fa[j].clear();
        for(int j=0;j<n;j++)
        {
            if(j==i) continue;
            int first_node = j;
            for(auto v : G[j])
            {
                if(vis[v]){
                    first_node = v;
                    break; 
                }
            }
            if(first_node != j) Fa[first_node].pb(j);
        }
     }
    for(int i=1;i<=q;i++) printf("%d\n", ans[i]);
}

int main()
{
    scanf("%d%d%d", &n, &m, &q);
    {
        int a,b,k;
        for(int i=1;i<=m;i++)
        {
            scanf("%d%d", &a, &b);
            a--,b--;
            G[a].pb(b);
            IG[b].pb(a);
        }
        for(int i=0;i<n;i++) sort(G[i].begin(), G[i].end());
        for(int i=1;i<=q;i++)
        {
            scanf("%d%d%d", &a, &b, &k);
            a--,b--;
            qs[b][a].pb( k-1,i );
        }
        solve();
    }
}
```