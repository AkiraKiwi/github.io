---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #1"
date:  2017-07-25 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #1 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+1&source=1&searchmode=source)

## Problem A

给出 \\(m\\) ，计算满足 \\(2^m-1=10^k\\) 的 \\(k\\)<br>

数据范围：\\( 1 ≤ m ≤ 10^{5}\\) <br>

## Solution 

\\( k = m \\times lg2\\) <br>


## Problem B

建立一个 \\(0-25\\) 到 \\(a-z\\)的满射，使得对于给定的 \\(n\\) 个字符串 \\(s\_i\\)，按映射转换为 \\(26\\) 进制数后的和最大 <br>

数据范围：\\( 1 ≤ n ≤ 10^{5}, 1 ≤ s\_i ≤ 10^{5}, \\sum s\_i ≤ 10^{6} \\) <br>

## Solution 

贪心策略：肯定是给 在高位出现次数更多的字符 赋更大的值，才能使最后的结果最大，那么统计每位数上字符的个数，超过26个就进位，这样将每个字符在每位上出现次数的状态转换为一个字符串，再按从高位到低位的字符排序，最后排在前面的赋更大的值即可。注意一下不要给在字符串第一个位置的字符赋为0，长度为1的除外。

## AC code
{% highlight c++ %}
int n;
LL num[26][MaxN];
char str[MaxN];
LL base[MaxM];
int high;
struct Node
{
    char ch;
    char ss[MaxN];
    bool operator<(const Node &a) const 
    {
        for(int i=high-1;i>=0;i--)
        {
            if(ss[i] != a.ss[i])
            {
                return ss[i] > a.ss[i];
            }
        }
        return false;
    }
}cnt[26];
LL val[26];
int vis[26];
void solve(int t)
{
    for(int i=0;i<high;i++)
    {
        for(int j=0;j<26;j++)
        {
            if(num[j][i]>=26) high = max2(high,i+2);
            num[j][i+1] += (num[j][i]/26);
            num[j][i] = num[j][i]%26;
            cnt[j].ss[i] = num[j][i]+'a';
        }
    }
    //debug(high)
    for(int i=0;i<26;i++) 
    {
        cnt[i].ch = 'a'+i;
    }
    sort(cnt, cnt+26);

    for(int i=25;i>=0;i--)
    {
        if(vis[cnt[i].ch-'a']!=1)
        {
            val[cnt[i].ch-'a'] = 0;
            break;
        }
    }
    int vv = 25;
    for(int i=0;i<26;i++) 
    {
        if(val[cnt[i].ch-'a']==-1)
        {
            val[cnt[i].ch-'a'] = vv;
            vv--;
        }
    }
    LL sum = 0;
    for(int i=0;i<high;i++)
    {
        for(int j=0;j<26;j++)
        {
            sum += ((1LL*num[j][i]*val[j])%mod*base[i]%mod);
            if(sum>mod) sum%=mod;
        }
    }
    printf("Case #%d: %lld\n", t, sum%mod);
}
int main()
{
    base[0] = 1;
    int t = 1;
    for(int i=1;i<=1000000;i++)
    {
        base[i] = (base[i-1]*26LL)%mod;
    }
    while(~scanf("%d", &n))
    {
        CLR(num);high = 0;CLR(vis);MST(val,-1);
        for(int i=1;i<=n;i++)
        {
            scanf("%s", str);
            int len = strlen(str);
            if(len>1) vis[str[0]-'a'] = 1;
            high = max(high,len);
            for(int j=len-1;j>=0;j--)
            {
                num[str[j]-'a'][len-j-1]++;
            }
        }
        solve(t++);
    }
}
{% endhighlight %}


## Problem C

给出一个 \\(n\\) 个节点的树，每个节点上有一种颜色 \\(c\_i\\)，对于树上的一条路径，它的价值等于这条路径上节点的颜色的种类数，求树上所有路径的价值和（路径数量：\\(\frac{n(n-1)}{2}\\)） <br>

数据范围：\\( 2 ≤ n ≤ 2\\times10^{5}, 1 ≤ c\_i ≤ n\\) <br>


## Solution

先一遍DFS统计出每颗子树的大小，再第二次DFS，计算每个节点的颜色对最后答案的贡献：<br>

* 该节点子树节点到非子树节点的路径：子树大小 \\(\\times\\) 外部节点个数（需要排除与该节点颜色相同的已经访问过的节点的子树，按照DFS序走的话就是，同一种颜色，后访问的节点不能计算先访问的节点的子树）

* 该节点的各个子树之间的路径数量：通过子树大小统计：对于第一棵子树，数量为 \\(size[v\_1]\\times (size[u]-size[v\_1])\\)，那么第二棵就是 \\(size[v\_2]\\times(size[u]-size[v\_1]-size[v\_2]) \\)，以此类推。

## AC code
{% highlight c++ %}

int n;
int col[MaxN];
struct Edge
{
    int u,v,next;
}edge[MaxM];
int cont, head[MaxN];
void add(int u, int v)
{
    edge[cont].u = u, edge[cont].v = v, edge[cont].next = head[u], head[u] = cont++;
}
LL sz[MaxN];
void DFS1(int u, int fa)
{
    sz[u] = 1;
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v = edge[i].v;
        if(v==fa) continue;
        DFS1(v,u);
        sz[u] += sz[v];
    }
}
LL sum[MaxN], tmp[MaxN],ans;
void DFS2(int u, int fa)
{
    ans += 1LL*(n - sz[u] - sum[col[u]])*sz[u];
    tmp[u] = sum[col[u]]; 
    int tsize = sz[u];
    for(int i=head[u];i!=-1;i=edge[i].next)
    {
        int v = edge[i].v;
        if(v==fa) continue;
        sum[col[u]] = n - sz[v];
        ans += 1LL*sz[v] * (tsize - sz[v]);
        DFS2(v,u);
        tsize -= sz[v];
    }
    sum[col[u]] = tmp[u]+sz[u];
}
void solve(int t)
{
    CLR(sz);
    DFS1(1,1);
    ans  = 0;
    DFS2(1,1);
    printf("Case #%d: %lld\n",t, ans);
}
void init()
{
    cont = 0, MST(head,-1), CLR(sum), CLR(tmp);
}

int main()
{
    int t = 1;
    while(~scanf("%d", &n))
    {
        for(int i=1;i<=n;i++) scanf("%d", &col[i]);
        int a,b;
        init();
        for(int i=1;i< n;i++)
        {
            scanf("%d%d", &a, &b);
            add(a,b);
            add(b,a);
        }
        solve(t++);
    }
}

{% endhighlight %}


## Problem F

给出两个序列 \\(a\_{1...n},b\_{1...m}\\) ，要求计算满足 \\(f(i)=b\_{f(a\_i)}\\) 的不同的映射\\(f\\) 的个数。<br>


数据范围：\\( 1 ≤ n ≤ 10^5, 1 ≤ m ≤ 10^5\\) <br>

## Solution

我们发现只有对于 \\(a\\) 数组的每一个闭环，都必须用 \\(b\\) 数组里的一些闭环，组合成同样大小的环才能满足题中要求。那么我们先求出每个数组的所有闭环及其数量，然后计算它们的配对方案数。计数可以在 \\(O(log(n))\\) 下处理出来：对于 \\(b\\) 中一个大小为 \\(x\\) 的环，我们先统计它的数量 \\(y\\)，然后对于 \\(x\\) 的每一个倍数，我们都能用 \\(y\\times x\\) 种方案拼凑上去（每个环旋转一次就是一个新方案），这样处理完 \\(b\\) 中所有能拼凑出的环的大小及其方案数，再查询 \\(a\\) 数组并统计答案。（在此感谢队友CPL大佬提供的 'PP' 算法）

## AC code

{% highlight c++ %}
int n,m;
int a[MaxN],b[MaxN];
bool vis[MaxN];
int anum[MaxN], bnum[MaxN];
long long p[MaxN];

void solve(int t)
{
    CLR(anum),CLR(bnum);
    CLR(vis);
    int aid=0;
    for(int i=0;i< n;i++)
    {
        if(!vis[i])
        {
            vis[i] = 1;
            int v = i;
            int tmp = 1;
            while(a[v] != i)
            {
                vis[a[v]] = 1;
                tmp++;
                v = a[v];
            }
            anum[aid++]=tmp;
        }
    }
    CLR(vis);
    int bid=0;
    for(int i=0;i<m;i++)
    {
        if(!vis[i])
        {
            vis[i] = 1;
            int v = i;
            int tmp = 1;
            while(b[v] != i)
            {
                vis[b[v]] = 1;
                tmp++;
                v = b[v];
            }
            bnum[tmp]++;
        }
    }

    memset(p,0,sizeof(p));
    for(long long i=1;i<=100000;i++)
    {
        if(bnum[i]!=0)
        {
            for(long long j=i;j<=100000;j+=i)
            {
                p[j]=(p[j]+i*bnum[i])%mod;
            }
        }
    }


    LL ans = 1;
    for(int i=0;i<aid;i++)
    {
        //debug(anum[i]) debug(bnum[i])
       ans=(ans*p[anum[i]]%mod)%mod;
    }
    printf("Case #%d: %lld\n", t, ans%mod);
}
int main()
{
    //std::ios::sync_with_stdio(false);
    int t=1;
    while(~scanf("%d%d", &n, &m))
    {
        for(int i=0;i<n;i++) scanf("%d", a+i);
        for(int i=0;i<m;i++) scanf("%d", b+i);
        solve(t++);
    }
}
{% endhighlight %}


## Contest Summary

* 一些代码量比较大的题最多分配2个人去做（对，说的就是B），保证稳定AC的同时不要落下题目进度

* F题刚开始就有正确方向的想法，但没有和队友沟通清楚，感觉拖了不少时间， : ( 类似思维题得先把idea说清楚

* C题类似的乱搞题讨论出大概方向就差不多了，然后在敲的过程中再定细节，这次就一开始XJB讨论到比赛结束然后在别人的一片AC声中打出GG， : )