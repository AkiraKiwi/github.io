---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #5"
date:  2017-08-08 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #5 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+5&source=1&searchmode=source)

## Problem A

给出 \\( A\_{1...n},\\ B\_{1...m},\\ k\_{1...q} \\)，求对于每一个 \\( k \\) 有多少对 \\( (i,j)满足，A\_i  \\ mod \\ B\_j = k \\)  <br>

数据范围：\\( 1 ≤ t ≤ 5, 1 ≤ n,m,q,A\_i,B\_i ≤ 50000, 0 ≤ k\_i ≤ max{B\_i}, if \\ i ≠ j, \\ A\_i ≠ A\_j, B\_i ≠ B\_j \\) <br>

## Solution 

因为数值范围不大， 所以我们可以把 \\( A , B \\) 都值域化，利用 \\( bitset \\) 存储， 从大到小枚举 \\(k\\) 统计答案，同时插入 \\(B\_i\\) 。<br>

## AC Code
```c++
int t,n,m,q,limit,A[MaxN],B[MaxN],k[MaxN],ans[MaxN];
bool is_b[MaxN], is_k[MaxN];
bitset<MaxN> a,b; //a串值域化A数组， b串值域化B数组
int main()
{
    read(t);
    while(t--)
    {
        read(n);read(m);read(q);a.reset();b.reset();CLR(is_b),CLR(is_k);limit = 0;

        for(int i=1;i<=n;i++) 
            read(A[i]), a.set(A[i]);

        for(int i=1;i<=m;i++) 
            read(B[i]), limit = max2(limit, B[i]), is_b[B[i]] = true;

        for(int i=1;i<=q;i++) 
            read(k[i]), is_k[k[i]] = true;

        for(int i=limit;i>=0;i--)
        {
            if(is_k[i]) 
                ans[i]=(b&(a>>i)).count()&1;    //统计A中 %B 余 i 的个数
            if(is_b[i])
                for(int j=0;j<=limit;j+=i) 
                    b.flip(j);                  //在b串中添加 B 的倍数
        }
        for(int i=1;i<=q;i++) out(ans[k[i]]),puts("");
    }
} 
```

## Problem B

给出 \\( n \\) 个字符串，求有多少个长度为 \\( 2L \\) 的串满足 \\( \\forall i ∈[1,\\|s\\|], s[i]≠s[\\|s\\|-i+1] \\) 且包含给出的 \\(n\\) 个字符串 <br>

数据范围：\\( 1 ≤ t ≤ 5, 1 ≤ n ≤ 6, 1 ≤ L ≤ 100, 1 ≤ \\|s\_i\\| ≤ 20 \\) <br>

## Solution 

当 \\(L\\) 小于 \\(10\\) 时，我们可以直接状压枚举所有字符串，然后在AC自动机里判断是否包含了给定串 。<br>

当 \\(L\\) 大于 \\(10\\) 时，我们定义 \\( DP[i][stat][k1][k2] \\) 为从中轴线往两边生成了 \\(i\\) 个字符，当前字符串的匹配状态为 \\(stat\\)，且左半边在AC自动机上匹配到 \\(k1\\) 节点， 右半边在AC自动机上匹配到 \\(k2\\) 节点的方案数。那么对于可能跨过中轴线的字符串，我们可以预先处理出 \\(10\\) 以内的结果，然后在这个基础上继续 \\(DP\\)。<br>

P.S. AC自动机插入字符时正着插一次，反着插入一次，因为两边匹配的方向不一样。<br>


## Problem I

求区间 \\( [L,R] \\) 内满足能够在 \\( d\\) 进制下表示成: <br>\\(x = (A\_1 A\_2 .. A\_m)\_d,\\  A\_1 - A\_m \\ is\\ a\\ permutation\\ of\\ numbers\\ from\\ 0\\ to\\ d-1,\\ d>1 \\) 的数的个数。 <br>

数据范围：\\( 1 ≤ t ≤ 20, 1 ≤ L ≤ R ≤ 10^{5000}, ANS\\ mod\\ 998244353 \\) <br>

## Solution 

首先，原问题转换为求 \\([1, x]\\) 内的好数个数，做差就是区间答案。对于 \\( n\\) 进制下的任何一个好数 \\(k\\)，都有\\( n^{n-1} < k < n^n \\)，所以每一个进制下的好数范围都互不相交，那么我们只需要求出能表示出满足条件的 \\(x\\) 的最大的 \\(d\\) ，然后再用类似康拓展开的方法求出小于它的其它排列的数量就能得到答案。 

