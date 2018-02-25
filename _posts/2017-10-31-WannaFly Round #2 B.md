---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-01.jpg
navigation: True
title: "WannaFly Round #2 B"
date: 2017-10-31 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Contest Information

source: [It's from the nowcoder](https://www.nowcoder.com/acm/contest/17/B)

## Description

\\(N\\) 座城市围成一个环，标号为 \\(1-N\\)，给出 \\(d\[i\]\\) 表示第 \\(i\\) 到 \\(i+1\\) 座城市的距离。另外还有 \\(M\\) 条捷径，每条捷径连接两座城市，所有路径都是双向边。最后给出 \\(Q\\) 个查询，询问两座城市之间的最短路径长度。<br>

数据范围：\\( 1 ≤ N,Q ≤ 52501; 1 ≤ M ≤ 20; 1 ≤ d[i],w[i] ≤ 2^{30} \\) <br>

## Input
```
4 1                                                                                   
1 2 3 6
1 3 2
5
1 2
1 4
1 3
2 3
4 3
```

## Output
```
1                                                                                     
5
2
2
3
```

## Solution 

如果没有捷径的话，那么求环上两个点的最短路，只需要比较一下从左边走和从右边走的距离即可。<br>
若加上捷径的话，那么有可能走某些捷径到达目的地的距离会更短一些，那么假设从 \\(u\\) 到 \\(v\\) 的最短路需要经过某条捷径，那么最短路上一定也经过了这条捷径的端点。由于最多只有 \\(20\\) 条捷径，即最多 \\(40\\) 个捷径端点，那么我们可以预处理出以每个捷径端点为源点的最短路数组，然后对于每个查询，我们先取左边和右边的最短路长度，然后枚举捷径端点，从数组中取该端点到起点和终点的最短路长度，最后保留最小值为答案。<br>
时间复杂度为 : \\(O(MNlgN+QM)\\)<br>

## AC code

```c++
vector<int> st;                                                                       
LL d[MaxN];
int main()
{
    std::ios::sync_with_stdio(false);
    while(cin >> N >> M)
    {
        init();
        int u,v; LL w;
        for(int i=1;i<=N;i++) {
            cin >> w;d[i]=d[i-1]+w;
            if(i==N) {
                add(N,1,w); add(1,N,w);
            }
            else{
                add(i, i+1, w); add(i+1, i, w);
            }
        }
        st.clear();
        while(M--)
        {
            cin >> u >> v >> w;
            add(u,v,w); add(v,u,w);
            st.pb(u),st.pb(v);
        }
        sort(st.begin(),st.end());
        st.erase(unique(st.begin(),st.end()), st.end());
        for(int i=0;i<st.size();i++){
            Dijkstra(i,st[i]);
        }
        int Q;
        cin >> Q;
        while(Q--)
        {
            cin >> u >> v;
            LL ans = abs(d[u-1]-d[v-1]);
            ans = min(ans, d[N]-ans);
            for(int i=0;i<st.size();i++)
                ans = min(ans, dis[i][u]+dis[i][v]);
            cout << ans << endl;
        }
    }
}
```