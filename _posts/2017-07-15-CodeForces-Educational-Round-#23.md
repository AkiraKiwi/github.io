---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-419599.jpg
navigation: True
title: "Codeforces Educational Round #23"
date:  2017-07-15 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [It's from the Educational Round #23 of the codeforces](http://codeforces.com/contest/817)

## Problem C

给出两个整数 \\(n,s\\)， 设 \\(n\\) 的数位和为 \\(ds\\)，求在区间 \\([1,n]\\) 之间满足 \\( det = (x-ds)≥s \\) 的数 \\(x\\) 有多少个？<br>

数据范围：\\( 1 ≤ n,s ≤ 10^{18}\\) <br>


## Solution 1

通过计算我们可以发现 \\( det\\) 每间隔 \\(10\\) 个数增加一次，整体上是递增的，那么我们可以在区间 \\([1,n]\\) 内通过二分查找出第一个满足 \\(det>=s\\)的数 \\(x\\)，那么答案就是\\(n-x+1\\)<br>


## AC code
{% highlight c++ %}
LL n,s;
bool check(LL x)
{
    LL tmp = x,cnt=0;
    while(x)
    {
        cnt += x%10;
        x/=10;
    }
    if(tmp-cnt>=s) return true;
    else return false;
}
void solve()
{
    LL L = 0, R = n;
    LL ans = R+1;
    while(L<=R)
    {
        LL mid = (L+R)>>1;
        if(check(mid))
        {
            ans = min(mid,ans);
            //debug(mid) 
            R = mid-1;
        }
        else L = mid+1;
    }
    printf("%lld\n", n-ans+1);
}
int main()
{
    while(~scanf("%lld%lld", &n,&s))
    solve();
    //system("pause");
}

{% endhighlight %}

## Solution 2

因为数位和的范围为 \\( [1,162]\\)，那么我们可以枚举数位和，每次通过数位统计 \\(DP\\) 计算出区间 \\([1,n]\\) 内满足 \\(det>=s\\) 且数位和等于枚举值的数的数量，累加就是答案<br>

## AC code
{% highlight c++ %}
LL n,S;
int digits[20];
LL Pow[20];
LL DP[20][22][200];

LL dfs(int pos, int dif, int sum, LL tar, bool limit)
{
    if(pos==0)
    {
        if (sum+S==tar && (dif-10)>=0) return 1;
        else return 0;
    }
    if(!limit && DP[pos][dif][sum]!=-1) return DP[pos][dif][sum];
    int Max = limit ? digits[pos] : 9;
    LL ans = 0;
    for(int i=0;i<=Max;i++)
    {
        int x = (dif-10)*10+i - tar/Pow[pos-1]%10;
        if(x>10) x = 10;
        if(x<-10) x = -10;
        ans += dfs(pos-1, x+10, sum+i, tar, limit&&(i==digits[pos]) );
    }
    if(!limit) DP[pos][dif][sum] = ans;
    return ans;
}

LL find(LL x)
{
    int pos = 0;
    while(x)
    {
        digits[++pos] = x%10;
        x/=10;
    }
    LL ans = 0;
    for(int i=0;i<=162;i++)
    {
        MST(DP,-1);
        LL tar = i+S;
       // debug(tar) debug(n)
        if(tar>n) break;
        ans += dfs(pos, 10, 0, tar, true);
    }
    return ans;
}
void solve()
{
    printf("%lld\n", find(n));
}
int main()
{
    
    Pow[0] = 1;
    for(int i=1;i<=20;i++) Pow[i] = Pow[i-1]*10;
    //std::ios::sync_with_stdio(false);
    scanf("%lld%lld", &n, &S);
    solve();
    system("pause");
}


{% endhighlight %}


## Problem D

给出一个长度为 \\(n\\) 的序列 \\(a\_1,...a\_n\\)，求序列的所有子区间中的最大值和最小值的差值的累加和。<br>

数据范围：\\( 1 ≤ n, a\_i ≤ 10^6\\) <br>


## Solution

对于每一个数 \\(a\_i\\)，我们计算出以它为最大值的最长区间：计算这个数左边第一个比它大的数 \\(a\_l\\)，右边第一个比它大的数 \\(a\_r\\)，然后这个数对该区间的贡献为 \\( (i-l+1)\\times (r-i+1)\\times a\_i\\)，然后累加每个数作为最大值的贡献。<br>

使用同样的方法计算出每个数作为最小值的贡献，用最大值的贡献减去最小值的贡献，即可得到答案

## AC code
{% highlight c++ %}
int n,A[MaxN];
int dp[5][MaxN];
void cal()
{
    stack<int> S;

    dp[0][1] = 1;
    S.push(1);
    for(int i=2;i<=n;i++)
    {
        while( S.size()>=1 && A[i]>=A[S.top()])
        {
            S.pop();
        }
        if(S.size()) dp[0][i] = S.top()+1;
        else dp[0][i] = 1;
        S.push(i);
    }
    while(S.size()) S.pop();

    dp[1][n] = n;
    S.push(n);
    for(int i=n-1;i>=1;i--)
    {
        while( S.size()>=1 && A[i]>A[S.top()])
        {
            S.pop();
        }
        if(S.size()) dp[1][i] = S.top()-1;
        else dp[1][i] = n;
        S.push(i);
    }
    while(S.size()) S.pop();

    dp[2][1] = 1;
    S.push(1);
    for(int i=2;i<=n;i++)
    {
        while( S.size()>=1 && A[i]<A[S.top()])
        {
            S.pop();
        }
        if(S.size()) dp[2][i] = S.top()+1;
        else dp[2][i] = 1;
        S.push(i);
    }
    while(S.size()) S.pop();

    dp[3][n] = n;
    S.push(n);
    for(int i=n-1;i>=1;i--)
    {
        while( S.size()>=1 && A[i]<=A[S.top()])
        {
            S.pop();
        }
        if(S.size()) dp[3][i] = S.top()-1;
        else dp[3][i] = n;
        S.push(i);
    }
    while(S.size()) S.pop();
}
void solve()
{
    cal();
    LL ans = 0;
    for(int i=1;i<=n;i++)
    {
        ans += (LL)(i+1-dp[0][i])*(dp[1][i]+1-i)*A[i];
        ans -= (LL)(i+1-dp[2][i])*(dp[3][i]+1-i)*A[i];
        //debug(dp[0][i]) debug(dp[1][i]) debug(dp[2][i]) debug(dp[3][i])

    }
    printf("%lld\n", ans);
}
int main()
{
    //std::ios::sync_with_stdio(false);
    while(~scanf("%d", &n))
    {
        CLR(dp);
        for(int i=1;i<=n;i++) scanf("%d", &A[i]);
        solve();
    }
    //system("pause");
    //printf("%lld\n", (x%mod+mod)%mod );
}

{% endhighlight %}