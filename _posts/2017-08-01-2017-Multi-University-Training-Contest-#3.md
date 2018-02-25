---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #3"
date:  2017-08-01 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #3 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+3&source=1&searchmode=source)

## Problem C

给出一个序列 \\(A\_1, A\_2,.., A\_n \\) ，定义 \\(f(l,r,k)\\) 为 \\(A[l..r]\\) 内的第 \\(k\\) 大数，且 \\(k>r-l+1,f(l,r,k)=0\\)。给定 \\(k\\)，要求计算 \\(\\sum\_{l=1}^n \\sum\_{r=l}^n f(l,r,k)\\)  <br>

数据范围：\\( 1 ≤ T ≤ 10, k ≤ min(n,80), \\sum n ≤ 5\\times 10^{5}, A[1..n]\\ is\\ a\\ permutation\\ of\\ [1..n] \\) <br>

## Solution 

又是一道算贡献的题，我们只需要算出每一个数能够出现在哪些区间然后计数即可：算出它往左边作为第 \\(k\\) 大的区间数量，往右边作为第 \\(k\\) 大的区间数量。 因为 \\(k≤80\\)，那么我们可以从小到大进行计数，使用链表存储，每次计算结束将当前数删除，就能保证链表中没有比枚举值小的数了，那么每次左右最多走 \\(k\\)次 ，时间复杂度为 \\(O(kn)\\)。

## AC Code
```c++
int T,n,k,A[MaxN],loc[MaxN];
struct Node
{
    int left, right,id;
}node[MaxN];
LL ans;
int L[100], R[100], limit;
void cal(int pos)
{
    CLR(L), CLR(R);
    int left = node[pos].left,pre = pos;
    int cnt = 0;
    while(cnt<limit)
    {
        if(left == -1)
        {
            L[cnt] = node[pre].id;
            break;
        }
        int dis = abs(node[left].id - node[pre].id);
        L[cnt++] = dis;
        pre = left;
        left = node[left].left;
    }
    int right = node[pos].right;pre = pos;
    cnt = 0;
    while(cnt<limit)
    {
        if(right==-1)
        {
            R[cnt] = n-node[pre].id+1;
            break;
        }
        int dis = abs(node[right].id-node[pre].id);
        R[cnt++] = dis;
        pre = right;
        right = node[right].right;
    }
    for(int i=0;i<limit;i++)
    {
        ans += 1LL * L[i] * R[limit-1-i]*A[pos];
    }
    node[node[pos].left].right = node[pos].right;
    node[node[pos].right].left = node[pos].left;
}
void solve()
{
    sort(loc+1, loc+1+n, [](int a, int b){return A[a]<A[b];});
    ans = 0;
    for(int i=1;i<=n;i++)
    {
        cal(loc[i]);
    }
    printf("%lld\n", ans);
}

int main()
{
    scanf("%d", &T);
    while(T--)
    {
        scanf("%d%d", &n, &k);
        limit = k;
        for(int i=1;i<=n;i++) 
        {
            scanf("%d", A+i);
            loc[i] = i;
            node[i].id = i;
            node[i].left = i-1?i-1:-1;
            node[i].right = i+1>n?-1:i+1;
        }
        solve();
    }
}
```

## Problem D

给出一个序列 \\( A\_{1...n} \\)，要求计算出满足 \\(i < j < k, ((A[i] xor A[j]) < (A[j] xor A[k])\\) 的三元组 \\((i,j,k)\\) 的数量。 <br>

数据范围：\\( 1 ≤ T ≤ 20, 1 ≤ \\sum n ≤ 5\\times 10^5, 0 ≤ A[i] < 2^{30}  \\) <br>

## Solution 

对于一个给定的 \\(A[i]\\) 和 \\(A[k]\\)，我们可以判断出满足条件的 \\(A[j]\\) 必然为在 \\(A[i]\\) 和 \\( A[k]\\) 从高往低第一个不同位上与 \\(A[i]\\) 相同的数。那么我们先预处理出前 \\(x\\) 个数，在第 \\(y\\) 位上为\\(0\\) 和 为\\(1\\) 各有多少个，然后从后往前遍历以保证计算时 \\(i< j < k\\)：<br>

每次先用当前数作为 \\(A[i]\\) 在字典树中查找答案：对于 \\(A[i]\\) 的每一位，我们把它的相反位当做 \\(A[k]\\)，那么累加相反位所在节点的答案：\\((j,k)\\) 对的数量，同时减去其中 \\(j < i\\) 的 \\((j,k)\\) 对的数量，就是这一位上的 \\((i,j,k)\\) 的数量；<br>

查询后再将当前数作为 \\(A[k]\\) 插入到字典树中，插入过程中统计每一位上 \\((j,k)\\) 对的数量：在预处理的数组中查找当前位置相反位满足 \\(j< k\\) 的数量，即当前节点的答案。<br>

## AC Code

```c++
int T,n,A[MaxN],sum[MaxN][31][2];
struct Trie
{
    int num = 0;
    struct Node
    {
        int sum;
        LL ans;
        int son[2];
    }node[31*MaxN];
    void clear(int n)
    {
        for(int i=0;i<=n*31;i++) node[i].ans = node[i].sum = node[i].son[0] = node[i].son[1] = 0;
        num = 0;
    }
    void insert(int pos, int v)
    {
        bitset<31> b(v);
        node[0].sum ++;
        int u = 0;
        for(int i=29;i>=0;i--)
        {
            int t = b[i];
            if(node[u].son[t]==0) node[u].son[t] = ++num;
            u = node[u].son[t];
            node[u].ans += sum[pos-1][i][t^1];
            node[u].sum ++;
        }
    }
    LL find(int pos, int p)
    {
        bitset<31> P(p); 
        int u = 0;
        LL ans = 0;
        for(int i=29;i>=0;i--)
        {
            int t = P[i];
            if( node[u].son[t^1] ){
                int loc = node[u].son[t^1];
                ans += node[loc].ans - 1LL*node[loc].sum*sum[pos][i][t];
            }
            if(!node[u].son[t]) break;
            u = node[u].son[t];
        }
        return ans;
    }
}trie;

void solve()
{
    CLR(sum);
    for(int i=1;i<=n;i++)
    {
        for(int j=0;j<=29;j++)
        for(int k=0;k<=1;k++) sum[i][j][k] = sum[i-1][j][k];
        for(int j=0;j<=29;j++){
            int v = ((A[i]&(1<<j))>0);sum[i][j][v]++;
        }
    }
    trie.clear(n);
    LL ans = 0;
    for(int i=n;i>=1;i--)
    {
        ans += trie.find(i,A[i]);
        trie.insert(i, A[i]);
    }
    printf("%lld\n", ans);
}

int main()      
{
    scanf("%d", &T);
    while(T--)
    {
        scanf("%d", &n);
        for(int i=1;i<=n;i++) scanf("%d", &A[i]);
        if(n<3) puts("0");
        else solve();
    }
}
```


## Problem E

给出一棵 \\(n\\) 个节点的树，要求将节点 \\(2...n\\) 分成 \\(k\\) 个集合 \\(S\_1, S\_2, S\_3,... S\_K\\)，使其满足 \\(S\_i \\cap S\_j = \\varnothing, i≠j \\)，并且 \\(\\sum\_{i=1}^{k}f(\\{1\\}\\cup S\_i) \\) 的值最大，（其中 \\(f(S)\\) 为点集 \\(S\\) 在树上的最小斯坦纳树） <br>

数据范围：\\( 1 ≤ k ≤ n ≤ 10^{6}, 1 ≤ edge.w < 10^5 \\) <br>

## Solution 

问题转换：将节点分成 \\(k\\) 个不同的集合等价于将节点标上不同的标号，对于每一个集合求最小斯坦纳树然后使得总和最大等价于将相同标号的节点尽可能分散到每一棵子树下，即任一节点下的子节点的种类数尽可能多：\\(min(k,size(u))\\)，这样该节点向上连的边能被计数的次数就最多了，即总和最大。

## AC Code
```c++
int n,k;
struct Edge
{
    int u, v, w, next;
}edge[MaxM];
int cont, head[MaxN];
void add(int u, int v, int w)
{
    edge[cont].u = u, edge[cont].v = v, edge[cont].w = w, edge[cont].next = head[u], head[u] = cont++;
}
int sz[MaxN];
void cal(int u, int fa)
{
    sz[u] = 1;
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v = edge[i].v;
        if(v==fa) continue;
        cal(v,u);
        sz[u]+=sz[v];
    }   
}
LL ans;
void dfs(int u, int fa, int x)
{ 
    ans += 1LL*min(sz[u], k)*x;
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v = edge[i].v, w = edge[i].w;
        if(v==fa) continue; 
        dfs(v,u,w);
    }
}

void solve()
{
    ans = 0;
    cal(1,1);
    dfs(1,1,0);
    printf("%lld\n", ans);
}

void init()
{
    cont = 0, MST(head,-1);
}

int main()
{
    while(~scanf("%d%d", &n,&k))
    {
        init();
        int a,b,c;
        for(int i=1;i<n;i++)
        {
            scanf("%d%d%d", &a,&b,&c);
            add(a,b,c);
            add(b,a,c);
        }
        solve();
    }
}
```

