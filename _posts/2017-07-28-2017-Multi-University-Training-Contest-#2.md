---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-20170726.jpg
navigation: True
title: "2017 Multi-University Training Contest #2"
date:  2017-07-28 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [The 2017 Multi-University Training Contest #2 from HDU OJ](http://acm.hdu.edu.cn/search.php?field=problem&key=2017+Multi-University+Training+Contest+-+Team+2&source=1&searchmode=source)

## Problem F

函数 \\(F\_{x,y}\\) 满足： <br>

\\[
F\_{1,1} = F\_{1,2} = 1 \\\\ 
F\_{1,i} = F\_{1,i-1} + 2\\times F\_{1,i-2} \\  (i≥3) \\\\ 
F_{i,j} = \sum^{j+N+1}\_{k=j} F\_{i-1,k} \\  (i≥2,j≥1)
\\]

给定 \\(N,M\\), 计算 \\(F_{m,1}\\ mod\\ 1e9+7\\) <br>

数据范围：\\( 1 ≤ T ≤ 10^{4}, 1 ≤ N,M < 2^{63} \\) <br>

## Solution 

<figure>
	<a href="https://akirakiwi.github.io/assets/img/picture/2017MUT2.jpg"><img src="https://akirakiwi.github.io/assets/img/picture/2017MUT2.jpg"></a>
	<figcaption>
		矩阵快速幂
	</figcaption>
</figure>


## Problem H

给出一个 \\(N\\times M\\) 的矩阵，每个位置有一个数，现在随机选取一个子矩阵，其中不同数字的个数为这次选取的价值，求随机选取一次子矩阵的价值的期望。 <br>

数据范围：\\( 1 ≤ T ≤ 8, 1 ≤ n,m ≤ 100, 0 ≤ g\_{i,j} < N\\times M  \\) <br>

## Solution 

首先，对于任意一个给定的矩阵，子矩阵可能的情况为 \\( \\frac{n(n+1)\\times m(m+1)}{4} \\)，然后我们需要计算出每一个子矩阵的价值：<br>

* 枚举原矩阵每一个位置，我们判断它可能在哪些子矩阵中被计数，因为子矩阵价值等于不同数字的个数，所以我们规定，从左上角按行遍历矩阵，对于每一种数字，它有贡献的矩阵不能包含出现在它前面的同种数字，但是出现在后面的没有限制。这样我们就能保证总价值不会算重。

* 对于某一个位置 \\(g\_{i,j}\\)，我们统计包含它的子矩阵，只需要统计矩阵左上角和右下角的位置即可：
  * 左上角：我们使用vector保存每一种数字在每一行出现的情况。首先设 \\(L=1\\)，特判当前行，如果当前行还有该数字 \\(g\_{i,j'}\\)，那么 \\(L\\) 更新为 \\(j'+1\\)，并统计当前行左上角可能的情况，有 \\(j-L+1\\) 种。然后我们逐行往上遍历，每次查找该数字有没有出现在 \\((L,j)\\) 之间，有则更新 \\(L\\)，且每一行都统计 \\(j-L+1\\)。如果这一行在 \\(j\\) 处也有该数字，那么跳出遍历，因为到上界了。
  * 右下角：右下角往下的范围永远是当前行到最后一行，但是往右是有边界的。首先设 \\(R=M\\)，当前行直接统计 \\( (R-j+1)\\times (N-i+1) \\) 种情况。然后我们逐行往上遍历，每次查找该数字有没有出现在 \\((j,R)\\) 之间，有则更新 \\(R\\)，且每一行都统计 \\( (R-j+1)\\times (N-i+1) \\)。如果这一行在 \\(j\\) 处也有该数字，那么跳出遍历，因为到上界了。
  * 左上角和右上角是同步统计的，对于每一行，我们都会累加上 \\( (j-L+1)\\times (R-j+1)\\times (N-i+1) \\)

## AC code
{% highlight c++ %}
int T,N,M;
int G[110][110];
int chazhao[10005];
vector<pair<int,int> > V[10005];
void solve()
{
    for(int i=0;i<N*M;i++) V[i].clear();
    for(int i=0;i<N*M;i++) vector<pair<int,int> >().swap(V[i]);
    MST(chazhao,-1);
    int cont = 0;
    for(int i=1;i<=N;i++)
    {
        for(int j=1;j<=M;j++)
        {
            if(chazhao[G[i][j]]==-1) chazhao[G[i][j]] = cont++;
            int tt = chazhao[G[i][j]];
            V[tt].pb(mp(i,j));
        }
    }
    LL ans = 0;
    for(int i=1;i<=N;i++)
    {
        for(int j=1;j<=M;j++)
        {
            int num = chazhao[G[i][j]];
            int L = 1,R = M;
            int loc = (lower_bound(V[num].begin(), V[num].end(), mp(i,j))-V[num].begin()-1);
            if(V[num][loc].first==i) L = V[num][loc].second+1;
            ans += (j-L+1)*(R-j+1)*(N-i+1);
            for(int k=i-1;k>=1;k--)
            {
                loc = (lower_bound(V[num].begin(), V[num].end(), mp(k,0))-V[num].begin());
                if(loc==V[num].size() || V[num][loc].first > k)
                {
                    ans += (j-L+1)*(R-j+1)*(N-i+1);
                    continue;
                }
                loc = (lower_bound(V[num].begin(), V[num].end(), mp(k,j))-V[num].begin());
                if(V[num][loc].first == k && V[num][loc].second == j) break;
                else 
                {
                    if(loc == 0 || V[num][loc-1].first < k)
                    {
                        R = min(R,V[num][loc].second-1);
                        ans += (j-L+1)*(R-j+1)*(N-i+1);
                    }
                    else if(V[num][loc].first > k)
                    {
                        L = max(L, V[num][loc-1].second+1);
                        ans += (j-L+1)*(R-j+1)*(N-i+1);
                    }
                    else
                    {
                        L = max(L, V[num][loc-1].second+1);
                        R = min(R,V[num][loc].second-1);
                        ans += (j-L+1)*(R-j+1)*(N-i+1);
                    }
                }
            }
        }
    }
    LL tot = (1LL*N*(N+1)/2LL)*(1LL*M*(M+1)/2LL);
    double tmp =  ans*1.0/(tot*1.0);
    printf("%.9f\n",tmp);
    
}
int main()
{
    scanf("%d",&T);
    while(T--)
    {
        scanf("%d%d", &N, &M);
        for(int i=1;i<=N;i++)
        {
            for(int j=1;j<=M;j++)
            {
                scanf("%d", &G[i][j]);
            }
        }
        solve();
    }
}
{% endhighlight %}


## Problem I

给出一个数组 \\(A\_{1...n}\\)，问有多少个数组 \\(B\_{1...n}\\) 满足：<br>

* \\( 1≤B\_i≤A\_i \\)

* 对于任意一个区间 \\([l,r]\\)，\\(gcd(b\_l,b\_{l+1},...,b\_r)≥2\\)

数据范围：\\( 1 ≤ n, A\_i ≤ 10^{5}\\) <br>


## Solution

首先，由第二个条件问题转换为找到一些 \\(B_{1..n}\\) 使得每一个数都比相应的 \\(A\_i\\) 小，且 \\(gcd(B\_1,...B\_n) ≥ 2\\) <br>
因为数值范围较小，所以先将数组值域化：记录区间 \\([1,i]\\) 之间有多少个数。然后我们从 \\(2\\) 开始枚举 \\(gcd\\) 值，直到大于任意一个 \\(A_i\\)。对于每一个 \\(gcd\\)，我们枚举它的倍数 \\(x\\)，然后统计 \\([x\\times gcd, (x+1) \\times gcd)\\) 内 \\(A\_i\\) 的数量 \\(N\\)，即代表相应的 \\(B\_i\\) 可以在该 \\(gcd\\) 下可以有 \\(x\\) 种取值，那么该 \\(gcd\\) 下的答案累乘上 \\(x^N\\) 次即可，然后最终答案累加每一次枚举 \\(gcd\\) 的答案。<br>
这样枚举不同的 \\(gcd\\) 的倍数得到的 \\(B\_i\\) 肯定是有重复的，所以我们需用到容斥原理去重：对于某一个 \\(gcd\\)，我们将它质因子分解，如果它含有两个及以上的相同质因子，就直接跳过；如果含有奇数个不同的质因子，则累加答案；如果含有偶数个不同的质因子，则减去答案。这样可以保证答案不重复，证明详见莫比乌斯反演。 <br>


## Problem K

二维平面上给N个整数点，问能构成多少个不同的正多边形。<br>


数据范围：\\( 1 ≤ N ≤ 500, -100 ≤ x,y ≤ 100\\) <br>

## Solution

容易得知只有正四边形可以使得所有的顶点为整数点。（具体证明可参考杨景钦在2017的国家队论文） 所以正解即求出所有的正四边形个数。 <br>

## AC code

{% highlight c++ %}
int n;
struct Node
{
    int x,y;
    bool operator < (const Node &a) const
    {
        if(x == a.x) return y<a.y;
        else return x<a.x;
    }
}node[MaxN+5];
vector<Node> Hash[MaxN+5];

bool check(Node a)
{
    int sum = abs(a.x+a.y)%MaxN;
    if(Hash[sum].size()==0) return false;
    else
    {
        for(int i=0;i<Hash[sum].size();i++)
        {
            if(a.x == Hash[sum][i].x && a.y == Hash[sum][i].y) return true;
        }
        return false;
    }
}
void insert(Node one)
{
    int sum = abs(one.x + one.y)%MaxN;
    Hash[sum].push_back(one);
}
void solve()
{
    sort(node,node+n);
    for(int i=0;i<n;i++) insert(node[i]);
    int ans = 0;
    for(int i=0;i<n;i++)
    {
        for(int j=i+1;j<n;j++)
        { 
            Node tmp;
            tmp.x = node[i].x+node[i].y-node[j].y;
            tmp.y = node[i].y+node[j].x-node[i].x;
            if(!check(tmp)) continue;
            tmp.x = node[j].x+node[i].y-node[j].y;
            tmp.y = node[j].y+node[j].x-node[i].x;
            if(!check(tmp)) continue;
            ans++;
        }
    }
    printf("%d\n", ans/2);
}
void init()
{
    for(int i=0;i<MaxN;i++) Hash[i].clear();
}
int main()
{
    while(~scanf("%d", &n))
    {
        init();
        for(int i=0;i<n;i++)
        {
            scanf("%d%d",&node[i].x, &node[i].y);
        }
        solve();
    }
}
{% endhighlight %}


## Contest Summary

* 由于队长请假，所以这场比赛是CPL大佬拖着本蒟蒻战战兢兢打完的 \_(:з」∠)\_ 

* F题 CPL大佬一波推出等比数列公式带走，不过正解是去推矩阵快速幂

* 最后一个多小时怼08，由于蜜汁状态，没商量好做法就开始瞎敲，最后直接WA翻。赛后发现之前的代码应该是100W个vector内存出问题了，然后压缩成1W个vector，捣鼓好边界判定条件就过了，时间还算比较快，内存空间也很小。

* 沟通很重要，沟通很重要，沟通很重要！！！