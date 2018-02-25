---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-419599.jpg
navigation: True
title: "Codeforces Educational Round #25"
date:  2017-07-26 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [It's from the Educational Round #25 of the codeforces](http://codeforces.com/contest/825)

## Problem E

给一个 \\(n\\) 个节点，\\(m\\) 条边的有向无环图标号，使其满足：<br>

* \\(1-n\\) 的每一个数字出现且仅出现一次

* 父节点标号比子节点小

* 节点序号输出的标号一定是字典序最小的

数据范围：\\( 2 ≤ n ≤ 10^{5}, 1 ≤ m ≤ 10^{5} \\) <br>


## Solution

因为要保证字典序最小，如果从父节点往下标号的话，因为不知道哪边的子树含有更小的子节点，所以没法从小往大分配标号。<br>

那么我们可以考虑反着做，从叶子节点开始先分配大的标号，对于一堆待分配的叶子节点，为了使字典序更小，肯定是序号大的拿更大的标号。这样一层一层往上走肯定能保证父节点标号小于子节点，同时贪心策略也能保证字典序最小。<br>


## AC code
{% highlight c++ %}
int n,m;
struct Edge
{
    int u,v,next;
}edge[MaxM];
int cont, head[MaxN],in[MaxN],ans[MaxN];
void solve()
{
    priority_queue < int > Q;
    for(int i=1;i<=n;i++) if(in[i]==0) Q.push(i);
    int cnt = n;
    while(!Q.empty())
    {
        int u = Q.top();
        ans[u] = cnt--;
        Q.pop();
        for(int i = head[u];i!=-1;i=edge[i].next)
        {
            int v = edge[i].v;
            in[v]--;
            if(in[v]==0) Q.push(v);
        }
    }
    for(int i=1;i<=n;i++)
    {
        printf("%d%c", ans[i], i==n?'\n':' ');
    }
}
void add(int u, int v)
{
    edge[cont].u = u, edge[cont].v = v, edge[cont].next = head[u], head[u] = cont++;
}
void init()
{
    cont = 0, MST(head,-1), CLR(in), CLR(ans);
}

int main()
{
    while(~scanf("%d%d", &n, &m))
    {
        init();
        int u,v;
        while(m--)
        {
            scanf("%d%d",&u,&v);
            add(v,u);
            in[u]++;
        }
        solve();
    }
}
{% endhighlight %}

## Problem F

给出一个字符串 \\(s\\)，可以把连续的相同的子串压缩，例如 \\(aaaa \\rightarrow 4a,\\ abaaba\\rightarrow 2aba\\) ，求压缩后的最短长度。<br>

数据范围：\\( 1 ≤ s ≤ 8\\times 10^3\\) <br>


## Solution

定义 \\(DP[i]\\) 为前 \\(i\\) 个字符压缩后的最短距离，那么这明显是 \\(O(n^2)\\) 更新答案。但是问题在于怎么用 \\(DP[j],\\ (j < i) \\) 更新 \\(DP[i]\\) 的值：<br>
首先，我们需要计算出 \\(s\_{j..i}\\) 压缩后的最短距离。因为是压缩相同的子串，相当于找一段字符串的循环节，然后加上循环次数就是压缩后的长度。
但是求循环节需要先计算 \\(Next\\) 数组，如果对于每一段 \\(s\_{j..i}\\)， 随着 \\(j\\) 往前递减都重新计算一次，那就是 \\(O(n^3)\\) 的复杂度，肯定不行。
我们发现循环更新的顺序是把 \\(j\\) 从 \\(i\\) 往前递推，那么我们就可以从 \\(i\\) 往前计算 \\(Next\\) 数组，即把字符串倒过来变成 \\(s\_{i..j}\\) ，因为开始的位置不变，那么 \\(Next\\) 数组可以直接更新重复使用。<br>

计算循环节长度方法：
```
if( len %(len-Next[len-1]) == 0 )
    length = len - Next[len-1];
else 
    length = len;
```

## AC code
{% highlight c++ %}
string str;
int DP[MaxN];
int Next[MaxN];
int cal(int x){
    int cnt = 0;
    while (x != 0)
        cnt++, x /= 10;
    return cnt;
}
void solve()
{
    MST(DP,INF);
    DP[0] = 0;
    DP[1] = 2; 
    int N = str.length();
    str = ' ' + str;
    for(int i=1; i<=N; i++)
    {
        string s = str.substr(1,i);
        reverse(ALL(s));
        CLR(Next);
        DP[i] = min(DP[i-1]+2, DP[i]);
        for(int j=1;j<i;j++)
        {
            int loc = Next[j-1];
            while(loc>0 && s[j]!=s[loc]) 
                loc = Next[loc-1];
            if(s[j]==s[loc]) loc++;
            Next[j] = loc;

            int len = j+1, length;
            if( len %(len-Next[len-1]) == 0 )
                length = len - Next[len-1];
            else 
                length = len;
            int num = len/length;
            
            DP[i] = min(DP[i], DP[i-j-1]+cal(num) + length);
        }
    }
    cout << DP[N] << endl;
}

int main()
{
    std::ios::sync_with_stdio(false);
    while(cin >> str)
    {
        solve();
    }
}
{% endhighlight %}

