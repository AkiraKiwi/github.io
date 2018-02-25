---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #8"
date:  2017-08-17 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #8 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+8&source=1&searchmode=source)

## Problem B

计算 \\( \\sum^n\_i \\sum^i\_{j} \\left \\lceil \\frac{i}{j} \\right \\rceil [gcd(i,j)==1] \\)。  <br>

数据范围：\\( 1 ≤ t ≤ 10^4, 1 ≤ n ≤ 10^{6} \\)。 <br>

## Solution 

推导：<br>
\\( \\ \\sum^n\_i \\sum^i\_{j} \left \lceil \\frac{i}{j} \right \rceil [gcd(i,j)=1] \\) <br>
\\( = \\sum^n\_i \\sum^i\_{j} \left \lceil \\frac{i}{j} \right \rceil \\sum\_{d\\|gcd(i,j)} \mu (d) \\)<br>
\\( = \\sum^n\_{d=1} \\mu (d) \\sum\_{k\_1 = 1}^{\\frac{n}{d}} \\sum^{k\_1}\_{k\_2 = 1} \left \lceil \\frac{k\_1}{k\_2} \right \rceil  \\)<br>
设 \\( g(n) = \\sum^n\_{i=1} \left \lceil \\frac{n}{i} \right \rceil \\)<br>
那么，原式为 \\( \\sum\_{d=1}^n \mu (d) \\sum^{\\frac{n}{d}}\_{k\_1=1}  g(k\_1) \\)<br>
通过找规律发现 \\( g(n) = g(n-1) + d(n-1) + 1 \\), 其中 \\(d(n)\\) 为 \\(n\\) 的约数个数。<br>
所以我们预处理出 \\(g(n)\\) 的前缀和，然后分块累加即可。<br>
时间复杂度：预处理为 \\(O(n)\\)，每次计算答案为 \\(O(\\sqrt n)\\) 。<br>


## AC Code
```c++
int n;

int mu[MaxN],prime[MaxN],d[MaxN],same[MaxN],g[MaxN],gg[MaxN],sum[MaxN];
bool not_prime[MaxN];
void Mobius()
{
    int tot = 0;
    mu[1] = 1;
    d[1] = 1;
    for(int i=2;i<=1E6;i++)
    {
        if(!not_prime[i])
        {
            prime[++tot] = i;
            mu[i] = -1;
            d[i] = 2;
            same[i] = 1;
        }
        for(int j=1;prime[j]*i<=1E6 ;j++)
        {
            not_prime[prime[j]*i] = 1;
            if(i%prime[j]==0)
            {
                mu[prime[j]*i] = 0;
                d[i*prime[j]] = d[i]/(same[i]+1)*(same[i]+2);
                same[i*prime[j]] = same[i] + 1;
                break;
            }
            d[prime[j]*i] = d[i]<<1;
            same[prime[j]*i] = 1;
            mu[prime[j]*i] = -mu[i];
        }
    }
    g[1] = 1 ;
    for(int i=2;i<=1E6;i++) g[i] = (g[i-1]+d[i-1]+1)%mod;
    sum[0] = 0; gg[0] = 0;
    for(int i=1;i<=1E6;i++) sum[i] = sum[i-1] + mu[i], gg[i] = (gg[i-1]+g[i])%mod;
}


int main()
{
    Mobius();
    while(~scanf("%d", &n))
    {
        LL ans = 0;
        for(int i=1,last;i<=n;i=last+1)
        {
            last = min(n/(n/i),n) ;
            ans = (ans + 1LL*(sum[last]-sum[i-1])%mod*1LL*gg[n/i]%mod )%mod;
        }
        printf("%lld\n", (ans%mod+mod)%mod );
    }
}
```

## Problem H

给出数列 \\( a\_{1...n} \\)。求能否从中挑出一些数累加得到 \\(k\\)。 <br>

数据范围：\\( -10^3 ≤ a\_i ≤ 10^3, 1 ≤ n ≤ 10^{3}, \\|k\\| ≤ 10^6 \\)。 <br>

## Solution 

利用 \\(bitset\\) 记录能够到达的值：每加入一个数\\(x\\)，将原来的 \\(bitset\\) 左移或者右移 \\(x\\) 位然后再与原 \\(bitset\\) 或运算得到新的 \\(bitset\\)，位移运算相当于将\\(x\\) 累加到了原来能够到达的所有值上，或运算相当于将新得到的值与原来的值合并在一块。

## AC Code
```c++
int T, n, k, a[MaxN];
char b[MaxN];
bitset<MaxM> Right, Left;

void solve()
{
    int rmax = 0, lmax = 0;
    for(int i=1;i<=n;i++)
        switch(b[i])
        {
        case 'N':
            rmax+=a[i];
            lmax+=a[i];
            break;
        case 'D':
            lmax+=a[i];
            break;
        case 'L':
            rmax+=a[i];
        }
    if( (k>0 && rmax<k) || (k<0 && lmax<-k) ) puts("no");
    else
    {
        for(int i=1;i<=n;i++)
        {
            switch(b[i])
            {
            case 'N':
                Right |= (Right<<a[i]);
                Left  |= (Left <<a[i]);
                break;
            case 'D':
                Left  |= (Left <<a[i]);
                break;
            case 'L':
                Right |= (Right<<a[i]);
            }
        }
        if(k==0 ||  (k>0 && Right.test(k)) || (k<0 && Left.test(-k)) ) puts("yes");
        else 
        {
            bool flag = false;
            for(int i=k+1;i<=1E6;i++)
            {
                if(k>0)
                {
                    if(Right.test(i) && Left.test(i-k)) 
                    {
                        flag = true;
                        break;
                    }
                }
                else
                {
                    if(Left.test(i) && Right.test(i+k)) 
                    {
                        flag = true;
                        break;
                    }
                }
            }
            if(flag) puts("yes");
            else puts("no");
        }
    }
}
int main()
{
    read(T);
    while(T--)
    {
        read(n);read(k);
        Left.reset();
        Right.reset();
        Left.set(0);
        Right.set(0);
        for(int i=1;i<=n;i++) read(a[i]);
        for(int i=1;i<=n;i++) scanf(" %c", &b[i]);
        solve();
    }
}
```