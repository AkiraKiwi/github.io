---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #6"
date:  2017-08-10 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #6 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+6&source=1&searchmode=source)

## Problem A

给出 \\( N \\) 个字符串 \\(W\_i\\) 和 \\( Q \\) 个查询，每个查询包含两个字符串 \\(S\_i, P\_i\\)，要求输出在之前的 \\( N \\) 个串里，满足以\\(S\_i\\)为前缀，\\(P\_i\\) 为后缀的字符串的数量  <br>

数据范围：\\( T ≤ 5, 0 < N,Q ≤ 10^5, \\sum(S\_i+P\_i) ≤ 500000, \\sum(W\_i) ≤ 500000\\) <br>

## Solution 

把 \\( S\_i, P\_i \\) 分别用字符连接然后插入到AC自动机，然后把原字符串 \\(W\_i\\) double后也用相同字符连接再首尾相接拼成一个长串，每个字符串中间用间隔符隔开，最后用这个长串去查询。<br>


## Problem G

给出一个 \\(1-N\\) 的排列，再给出 \\(M\\) 个查询 \\([L,R]\\)，要求计算 \\( \\sum^R\_{i=L} \\sum^R\_{j=i+1} \\sum^R\_{k=j+1} (gcd(A[i],A[j])==A[k]) \\times A[k] \\) <br>

数据范围：\\( 1 ≤ T ≤ 100, 1 ≤ N,M ≤ 10^5, 1 ≤ L,R ≤ N \\) <br>

## Solution 

首先，我们会想到是枚举 \\(k\\)，计算 \\(A[k]\\) 的贡献，那么由题意得，要计算 \\(A[k]\\) 的贡献，就要先计算所有满足 \\(i < j < k \\) 且 \\( gcd( \\frac{A[i]}{A[k]}, \\frac{A[j]}{A[k]})==1 \\) 的 \\( < i, j > \\) 的个数。<br>

那么，我们从左往右枚举 \\(k\\)，标记访问过的 \\(A[k]\\)，便于计算 \\(< i, j >\\) 时判断该数是否在 \\(k\\) 左边，然后对于 \\(i < j\\) ，我们可以把已访问过的 \\(A[k]\\) 的倍数按下标从大到小排序，然后遍历时先当做 \\(i\\) 查询再当做 \\(j\\) 处理。怎么查询呢？问题转换成：给定一个数 \\( x \\)，在一堆数中查询有多少个数与它互质。我们可以先预处理出每一个数整数分解的结果，然后处理 \\(j\\) 的时候累加因子个数，那么查询 \\(i\\) 时就可以直接用莫比乌斯容斥计算答案了 \\( f(n) = \\sum\_{d\\|n} μ(d) F(\\frac{n}{d}) \\)。 对于每一个 \\(i\\) 查询得到的结果都对 \\( L ∈ [1,i], R = k\\) 这些区间有贡献，所以需要区间修改，这一步可以树状数组维护。<br>

最后，离线所有查询，在每个 \\(A[R]\\) 处计算完答案，再查询相应 \\([L,R]\\) 的结果。<br>

## AC Code
```c++

int mu[MaxN],prime[MaxN];
bool not_prime[MaxN];
vector<int> fac[MaxN];

void Mobius()
{
    int tot = 0;
    mu[1] = 1;
    for(int i=2;i<=MaxN;i++)
    {
        if(!not_prime[i])
        {
            prime[++tot] = i;
            mu[i] = -1;
        }
        for(int j=1;prime[j]*i<=MaxN;j++)
        {
            not_prime[prime[j]*i] = 1;
            if(i%prime[j]==0)
            {
                mu[prime[j]*i] = 0;
                break;
            }
            mu[prime[j]*i] = -mu[i];
        }
    }
    // deal with the factor
    for(int i=1;i<MaxN;i++)
    for(int j=1;j*j<=i;j++)
    {
        if(j*j == i && mu[j]) fac[i].pb(j);
        else if(i%j==0)
        {
            if(mu[j])   fac[i].pb(j);
            if(mu[i/j]) fac[i].pb(i/j);
        }
    }
}

int T,N,M,A[MaxN],loc[MaxN],num[MaxN];
struct Query
{
    int id, L;
    Query(){}
    Query(int id, int L):id(id),L(L){}
};
vector<Query> Q[MaxN];
vector<int> mul;
LL ans[MaxN];

void cal(int i, int x, int val)
{
    LL tmp = 0;
    for( auto y : fac[x] )
        tmp += 1LL*mu[y] * num[y];
    bit.update(1, val*tmp);
    bit.update(i+1, -val*tmp);
    for( auto y : fac[x] ) num[y]++;
}
void solve()
{
    for(int i=1;i<=N;i++)
    {
        mul.clear();
        for(int j=2;j*A[i]<=N;j++)
            if(loc[j*A[i]]) mul.pb(loc[j*A[i]]);
        sort(mul.begin(), mul.end(), [](int a, int b){return a>b;});
        for(auto j : mul) cal(j,A[j]/A[i],A[i]);
        for(int j=0;j<=N/A[i];j++) num[j] = 0;
        loc[A[i]] = i;
        for(auto q : Q[i]) ans[q.id] = bit.sum(q.L);
    }
    for(int i=1;i<=M;i++) printf("%lld\n", ans[i]);
}
void init()
{
    for(int i=1;i<=N;i++) Q[i].clear();CLR(loc);CLR(ans);CLR(num);bit.init(N);
}
int main()
{
    Mobius();
    read(T);
    while(T--)
    {
        read(N);read(M);
        init();
        for(int i=1;i<=N;i++) read(A[i]);
        int l,r;
        for(int i=1;i<=M;i++)
        {
            read(l);read(r);
            Q[r].pb(Query(i,l));
        } 
        solve();
    }
}
```

## Contest Summary

* 字符串的题可以想到很多转换的方法，正着做很难，可以反过来想想。

* 三分时使用黄金分割比例可以优化时间

* [圆的反演](http://jingyan.baidu.com/article/77b8dc7f8a792e6174eab623.html)

* 满足 \\( i < j < k \\) 的三元组的统计问题，一般都是先固定 \\(K\\)，然后处理 \\(j\\)，查询\\(i\\)