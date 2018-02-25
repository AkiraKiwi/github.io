---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #7"
date:  2017-08-15 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #7 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+7&source=1&searchmode=source)

## Problem F

从 \\( [1, n] \\) 中至多选择 \\( k \\) 个数，使得它们的乘积不能被任何平方数整除，求方案数  <br>

数据范围：\\( 1 ≤ t ≤ 5, 1 ≤ n,k ≤ 500 \\) <br>

## Solution 

不能被任何平方数整除 \\(\\rightarrow\\) 每个质因子不超过两个，由于 \\( n \\) 不超过 \\(500\\)，所以我们考虑 \\(\\sqrt 500\\) 以内的质数，总共只有 \\(8\\) 个 <br>

那么我们将所有数按其质因子分解的情况分成 \\(2^8\\) 组（包含多个同种因子的数直接排除，而大于 \\(\\sqrt 500\\) 的质数每个数最多包含一个），然后我们定义 \\(DP[k][s]\\) 为选择 \\(k\\) 个数，质因子包含情况为 \\(s\\) 的方案数，然后遍历所有数，判断它能加入到那些状态里，统计方案（注意不同组不能相交）<br>

## AC Code
```c++
int prime[10] = {2,3,5,7,11,13,17,19};
vector<int> use[505];
int stat[505],a[505][505];
int T,n,k;
LL dp[505][505],last[505][505];

void init()
{
    for(int i=1;i<=n;i++) use[i].clear();
    for(int i=1;i<=n;i++)
    {
        int x = i;
        for(int j=0;j<8;j++)
        {
            if(x%prime[j]==0 && x%(Sqr(prime[j]))==0 )
            {
                stat[i] = -1;
                break;
            }
            if( x%prime[j]==0 )
            {
                stat[i] |= (1<<j);
                x/=prime[j];
            }
        }
        if(stat[i] == -1) continue;
        if(x==1) use[i].pb(stat[i]);
        else use[x].pb(stat[i]);
    }   
    for(int i=1;i<=n;i++)
        for(auto x : use[i])
        {
            a[i][x]++;
        }
}


int main()
{
    read(T);
    while(T--)
    {
        CLR(last);CLR(stat);CLR(a);CLR(dp);
        read(n); read(k);
        init();
        for(int i=1;i<=n;i++)
        {
            if(use[i].size() == 0) continue;
            for(int s=0;s<(1<<8);s++)
            {
                if(a[i][s]==0) continue;
                dp[1][s] = (dp[1][s]+1)%mod;   
                for(int j=1;j<=k;j++)
                    for(int ss=0;ss<(1<<8);ss++)
                        if( (s&ss)==0) dp[j][s|ss] = (dp[j][s|ss] + last[j-1][ss])%mod; 
            }
            for(int j=1;j<=k;j++)
                for(int s=0;s<(1<<8);s++)
                    last[j][s] = dp[j][s];
        }
        LL ans = 0;
        for(int j=1;j<=k;j++)
        for(int s=0;s<(1<<8);s++)
        {
            ans=(ans+last[j][s])%mod;
        }
        out(ans);puts("");
    }
}
```

## Problem G

给 \\( n \\) 个小朋友发糖果，每个人最多不能超过 \\( m \\) 个。 同时小朋友有 \\(k\\) 个要求，第\\( x\_i \\) 个小朋友拿的糖果数不能比第 \\(y\_i\\) 个小朋友多 \\(z\_i\\) 个以上。第 \\(i\\) 个小朋友拿 \\(j\\) 个糖果能获得 \\(w\_{i,j}\\) 的满足值，求最大的满足值  <br>

数据范围：\\( 1 ≤ t ≤ 5, 1 ≤ n,m ≤ 50, 1 ≤ k ≤ 150, 1 ≤ x,y ≤ n, \\|z\\| < 233, 0  ≤ w\_{i,j} ≤ 1000\\) <br>

## Solution 

如果不考虑 \\(k\\) 个要求，我们可以对于每个小朋友取最大的满足值即可，网络流建图就是从源点出发，对于每个小朋友，从给它一颗糖果连边一直到m颗糖果，最后到汇点，流量为 \\(1000-w\_{i,j}\\)，等价不满意值，那么整个图的最大流，即最小割就是最小的不满足值，对应最大的满足值。<br>

然后我们考虑 \\(k\\) 个要求，若要求 \\(x\_i\\) 和 \\(y\_i\\) 糖果数相差不超过 \\(z\_i\\)，那么相当于如果选择两个相差大于 \\(z\_i\\) 的边是不能割开整个图的，那么我们可以在 \\(x\_{i,j}\\)  和 \\(y\_{i,j-z}\\) 间连接一条 \\(INF\\) 的边，使得如果选择相差大于 \\(z\\) 的两条边无法割开整个图。<br>

## Problem I

给出一个序列 \\( a\_{1...n},\\ 0 ≤  a\_i < p  \\)，求有多少对 \\( i,j \\) 满足 \\( \\frac{1}{a\_i + a\_j} \\equiv \\frac{1}{a\_i} + \\frac{1}{a\_j}  \\)  <br>

数据范围：\\( 1 ≤ t ≤ 5, 1 ≤ n  ≤ 10^{5}, 2 ≤ p  ≤ 10^{18} \\) <br>

## Solution 

* 法一：推公式<br>
    由 \\( \\frac{1}{a\_i + a\_j} \\equiv \\frac{1}{a\_i} + \\frac{1}{a\_j} \\ (mod\\ p) \\) 得:<br>
     \\( a^2\_{i} + a^2\_j + a\_{i}a\_{j} \\equiv 0 \\ (mod\\ p) \\) <br>
    那么分两种情况 <br>
    * \\(a\_i = a\_j\\)：只有当 \\( p = 3 \\) 的时候有解，且 \\(a\_i = 1\\) 或者 \\(a\_i = 2\\)。<br>
    * \\(a\_i ≠  a\_j\\)：在原式两边同乘以 \\(a\_i-a\_j\\) 后转换为 \\(a^3\_i - a^3\_j \\equiv 0 \\ (mod \\ p) \\)，那么只要统计立方后 \\(mod \\ p\\) 同余的数即可。 <br>

* 法二：二次剩余<br>
    待补

## AC Code
```c++
int T,n;
LL p,a[MaxN];

map<LL,map<LL,LL> > same;
map<LL,LL>num;

LL AddMod(LL a, LL b, LL p)
{
    LL ans = 0;
    while(b>0)
    {
        if(b%2) ans = (ans+a)%p;
        b = b/2;
        a = (a+a)%p;
    }
    return ans;
}
LL cal(LL x)
{
    return x*(x-1)/2;
}
int main()
{
    read(T);
    while(T--)
    {
        read(n);read(p);
        for(int i=1;i<=n;i++) read(a[i]);
        if(p==3)
        {
            int one = 0, two = 0;
            for(int i=1;i<=n;i++) 
            {
                if(a[i]==1) one++;
                if(a[i]==2) two++;
            }
            LL ans = cal(1LL*one) + cal(1LL*two);
            out(ans);puts("");
        }
        else
        {
            same.clear(), num.clear();
            for(int i=1;i<=n;i++)
            {
                if(a[i]==0) continue;
                LL x = a[i];
                LL twice = AddMod(x, x, p);
                LL triple = AddMod(x, twice, p);
                same[triple][x]++;
                num[triple]++;
            }
            LL ans = 0;
            for(auto val: num)
            {
                ans += cal(val.second);
                for(auto s : same[val.first])
                {
                    ans -= cal(s.second);
                }
            }
            out(ans);puts("");
        }
    }
}
```

## Problem J

给出一个序列 \\( a\_{1...n},\\ 0 ≤  a\_i < p  \\)，求 \\( m \\) 次前缀异或后的序列 \\( b\_{1...n}  \\)  <br>

数据范围：\\( 1 ≤ t ≤ 5, 1 ≤ n ≤ 2\\times 10^{5}, 1 ≤ m ≤ 10^{9}, 0 ≤ a\_i ≤ 2^{30}-1 \\) <br>

## Solution 

我们可以发现一个递推式：\\(DP[i][j] = DP[i-1][j] xor DP[i][j-1]\\) <br>
其中 \\(DP[i][j]\\) 指第 \\(i\\) 次前缀异或的 \\(a\_j\\) 的值。<br>
那么我们继续展开发现 <br>
\\(DP[i][j] = DP[i-2][j] xor DP[i-1][j-1] xor DP[i-1][j-1] xor DP[i][j-2]\\)<br>
中间两次重复异或可以消去，再继续展开，最后我们能得到 <br>
\\(DP[i][j] = DP[i-2^k][j] xor DP[i][j-2^k], (i-2^{k+1}<1, j-2^{k+1}<1) \\)<br>
所以我们每次取不大于 \\(m\\) 的最大的 \\(2^k\\) 并计算贡献，就能得到 \\(m\\) 次前缀异或后的序列。 <br>

## AC Code
```c++
int T;
int n, m, a[MaxN];
int main()
{
    read(T);
    while(T--)
    {
        read(n);read(m);
        for(int i=1;i<=n;i++) read(a[i]);
        while(m>0)
        {
            int k = 1;
            while(k<m) 
            {
                k<<=1;
                if(k>n) break;
            }
            if(k>n || k>m) k>>=1;
            for(int i=1;i+k<=n;i++) 
            {
                a[k+i]^=a[i];
            }
            m-=k;
        }
        for(int i=1;i<=n;i++) printf("%d%c", a[i], " \n"[i==n]);
    }
}
```

## Contest Summary

* 涉及运算较多且复杂的题，先把公式都算清楚再开敲，并且一人敲，一人检查

* 碰到递推式，多展开，多尝试。

* 理解网络流的建图技巧