---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #4"
date:  2017-08-03 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #4 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+4&source=1&searchmode=source)

## Problem C

定义 \\( d(n) \\) 为正整数 \\( n \\) 的因数个数，要求计算 \\( (\\sum^r_{i=l}d(i^k))\\ mod\\ 998244353\\)  <br>

数据范围：\\( 1 ≤ T ≤ 15, 1 ≤ l ≤ r ≤ 10^{12}, r-l ≤ 10^{6}, 1 ≤ k ≤ 10^7 \\) <br>

## Solution 

首先，\\(d(n) = (r\_1+1)(r\_2+1)..(r\_k+1), (n = p\_1^{r\_1}p\_2^{r\_2}...p\_k^{r\_k})\\) 。<br>
那么，我们需要知道每个数的质因子数的数量和种类，而单独对每个数进行质因子分解，最多可能需要遍历几万个质数，而区间长度最大为 \\(10^6\\) ，所以时间复杂度过不去。<br>
我们思考，每次分解每一个数的时候都从头枚举一遍质数，而实际上很多质数都不能分解当前这个数，为了减少这种无用操作，我们可以改成枚举质数，然后判断它能分解哪些数：找到区间内第一个能被它分解的数，然后累加该质数得到的数都能被它分解。这样就可以省很多时间，不过由于我们只能找到 \\(10^6\\) 以内的素数，而数据范围为 \\(1^{12}\\)，所以可能枚举完素数后有的数还没有被分解完，但它只可能含有一个大于 \\(10^6\\) 的大质数，所以最后判断一下即可。<br>

## AC Code
```c++
LL l,r,k;
const int NICO = 1000000 + 10;
int prime[NICO+1];

void getPrim()
{
    CLR(prime);
    for(int i=2;i<NICO;i++)
    {
        if(!prime[i])
        {
           prime[++prime[0]] = i;
        }
        for(int j=1;(j<=prime[0])&&(prime[j]<=(NICO/i));j++)
        {
            prime[prime[j]*i] = 1;
            if(i%prime[j]==0) break;
        }
    }
}

LL A[MaxN], ans[MaxN];
void solve()
{
    for(LL i=l;i<=r;i++) A[i+1-l] = i,ans[i+1-l] = 1;
    int len = r-l+1;
    bool flag = true;
    int cnt = 1;
    for(int i=1;i<=prime[0] && Sqr(prime[i]) <= r;i++)
    {
        int begin = 1;
        if(l%prime[i]) begin += prime[i] - (l%prime[i]);
        while(begin<=len)
        {
            int cnt = 0;
            while(A[begin]%prime[i]==0)
            {
                cnt++;
                A[begin]/=prime[i];
            }
            ans[begin] = ans[begin]*(k*cnt+1)%mod;
            begin += prime[i];
        }
    }
    for(int i=1;i<=len;i++) if(A[i]>1) ans[i] = ans[i]*(k+1)%mod;
    LL ANS = 0;
    for(int i=1;i<=len;i++)
    {
        ANS = (ANS+ans[i])%mod;
    }
    printf("%lld\n", ANS);
}

int main()
{
    getPrim();
    int T;
    scanf("%d", &T);
    while(T--)
    {
        scanf("%lld%lld%lld", &l, &r, &k);
        solve();
    }
}
```

## Problem D

给出一个序列 \\( a\_{1...n} \\)，要求计算 \\( \frac{size(l,r)}{r-l+1} \\) 的最大值，其中 \\(size(l,r)\\) 为区间 \\([l,r]\\) 中不同数字的个数 <br>

数据范围：\\( 1 ≤ T ≤ 15, 1 ≤ n ≤ 6\\times 10^{4}, 1 ≤ a\_i ≤ n \\) <br>

## Solution 

这是一道很明显的分数规划问题，我们把最优值问题转换为可行性问题：二分答案判定解是否存在。<br>

那么假设 \\( ans = \frac{size(l,r)}{r-l+1} \\)，将左右端点分开，得到 \\( size(l,r)+ans\\times l = ans \\times (r+1) \\)，所以对于每一个答案，每一个区间的 \\(ans\\times l\\)  是固定的，我们只需要从左往右枚举右端点 \\(r\\)，同时使用线段树维护区间最小值 \\( size(l,r)+ans\\times l \\) ：每次先查找 \\(r\\) 之前的区间的最小值，然后插入当前位置更新 \\(size(l,r)\\) ，直到找到一个最小值小于 \\(  ans \\times (r+1) \\) 。<br>


## AC Code

```c++
bool check()
{
    T.build(1,1,n);
    CLR(pre);
    for(int i=1;i<=n;i++)
    {
        T.update(1,pre[a[i]]+1, i);
        double tmp = T.query(1,i).sum;
        if( (tmp-MID*(i+1))<eps ) return true;
        pre[a[i]] = i;
    }
    return false;
}
void solve()
{
    double l = 0.0, r = 1.0, ans = 1;
    while( r-l>=eps )
    {
        MID = (l+r)/2;
        if(check())
        {
            ans = MID;
            r = MID;
        }
        else l = MID;
    }
    printf("%.10f\n", ans);
}
int main()
{
    scanf("%d", &t);
    while(t--)
    {
        scanf("%d", &n);
        for(int i=1;i<=n;i++) scanf("%d", a+i);
        solve();
    }
}
```

## Problem E

给出四个 \\(checkpoint\\)，其中 \\(dis\_{1,2}, dis\_{2,3}, dis\_{3,4}, dis\_{4,1}\\) 为它们之间的路径长度，问从 \\(checkpoint \\ 2\\) 出发，沿着路径来回跑，最后回到起点时跑的总路径长度大于 \\(k\\) 的最小总路径长度。 <br>

数据范围：\\( 1 ≤ T ≤ 15, 1 ≤ K ≤ 10^{18}, 1 ≤ dis\_{i,j} ≤ 30000 \\) <br>

## Solution 

问题相当于让你在图中利用这四条路径构造出一个能从起点出发最后回到起点的方案，由于任何一个方案加上 \\(w = 2\\times min(dis\_{2,3}, dis\_{1,2})\\) 都能构成一个新的方案，所以我们只需要找到 \\(mod\\ w = j \\) 的能构造出来的最小路径长度即可。

## AC Code
```c++
int T;
LL K;
LL d12,d23,d34,d41,Map[5][5];

typedef pair<LL,pair<int,int> > pii;   
struct cmp
{                        
    bool operator()(pii a,pii b)
    {
        return a.first>b.first;
    }
};
LL dis[5][MaxN];
int Dijkstra()
{
    CLR(Map);
    LL w = min(d12, d23), w = 2*w;
    Map[1][2] = Map[2][1] = d12;
    Map[2][3] = Map[3][2] = d23;
    Map[3][4] = Map[4][3] = d34;
    Map[1][4] = Map[4][1] = d41;
    for(int i=0;i<w;i++) for(int j=0;j<=4;j++) dis[j][i] = INF;
    dis[2][0] = 0;
    priority_queue<pii,vector<pii>,cmp> q;
    q.push(mp(dis[2][0],mp(2,0)));
    while(!q.empty())
    {
        pii tmp = q.top();q.pop();
        int j = tmp.second.first, u = tmp.second.second; LL val = tmp.first;
        for(int i=1;i<=4;i++)
        {
            if(Map[j][i]==0) continue;
            LL v = (val+Map[j][i])%w;
            if( dis[i][v] > val+Map[j][i])
            {
                dis[i][v] = val+Map[j][i];
                q.push(mp(dis[i][v],mp(i,v)));
            }
        }
    }
    LL ans = INF;
    for(int i=0;i<w;i++)
    {
        LL tmp = dis[2][i];
        if(tmp<K) 
        {
            LL num = (K-tmp)/w;
            if((K-tmp)%w) num++;
            tmp += num*w;
        }
        if(ans>tmp) 
        {
            ans = tmp;
        }
    }
    printf("%lld\n", ans);
}
int main()
{
    scanf("%d", &T);
    while(T--)
    {
        scanf("%lld%lld%lld%lld%lld", &K, &d12, &d23, &d34, &d41);
        Dijkstra();
    }
}
```