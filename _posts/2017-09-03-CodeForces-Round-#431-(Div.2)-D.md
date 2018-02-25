---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-419599.jpg
navigation: True
title: "CodeForces Round #431 (Div. 2) D"
date:  2017-09-03 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
--- 

## Problem Information

source: [It's from the Round #431 (Div. 2) of the codeforces](http://codeforces.com/contest/849/problem/D)

## Description

在一个平面坐标系上有一个矩形舞台，四个顶点为 \\( (0,0), (w,0), (w,h), (0,h) \\)，在舞台的左侧和下方一共有 \\(n\\) 个舞者， 坐标分别为 \\( (x\_i, 0), (0, y\_i) \\)，每个舞者在第 \\(t\_i\\) 时刻会向垂直于其所在坐标系的方向以每秒一个单位的速度移动，如果与其它舞者碰撞，则互相交换移动方向，直到抵达舞台边缘停止。求每个舞者最后抵达的位置。<br>

On a Cartesian coordinate plane lies a rectangular stage of size \\(w × h\\), represented by a rectangle with corners \\( (0, 0), (w, 0), (w, h)\\) and \\((0, h)\\). It can be seen that no collisions will happen before one enters the stage.<br>

On the sides of the stage stand \\(n\\) dancers. The \\(i\\)-th of them falls into one of the following groups:<br>

Vertical: stands at \\((x\_i, 0)\\), moves in positive \\(y\\) direction (upwards);<br>

Horizontal: stands at \\((0, y\_i)\\), moves in positive \\(x\\) direction (rightwards).<br>

According to choreography, the \\(i\\)-th dancer should stand still for the first \\(t\_i\\) milliseconds, and then start moving in the specified direction at 1 unit per millisecond, until another border is reached. It is guaranteed that no two dancers have the same group, position and waiting time at the same time.<br>

When two dancers collide (i.e. are on the same point at some time when both of them are moving), they immediately exchange their moving directions and go on.<br>

Dancers stop when a border of the stage is reached. Find out every dancer's stopping position.<br>

## Input

The first line of input contains three space-separated positive integers n, w and h (1 ≤ n ≤ 100 000, 2 ≤ w, h ≤ 100 000) — the number of dancers and the width and height of the stage, respectively.

The following n lines each describes a dancer: the i-th among them contains three space-separated integers gi, pi, and ti (1 ≤ gi ≤ 2, 1 ≤ pi ≤ 99 999, 0 ≤ ti ≤ 100 000), describing a dancer's group gi (gi = 1 — vertical, gi = 2 — horizontal), position, and waiting time. If gi = 1 then pi = xi; otherwise pi = yi. It's guaranteed that 1 ≤ xi ≤ w - 1 and 1 ≤ yi ≤ h - 1. It is guaranteed that no two dancers have the same group, position and waiting time at the same time.

## Output

Output n lines, the i-th of which contains two space-separated integers (xi, yi) — the stopping position of the i-th dancer in the input.

## Sample Input

~~~
8 10 8
1 1 10
1 4 13
1 7 1
1 8 2
2 2 0
2 5 14
2 6 0
2 6 1

3 2 3
1 1 2
2 1 1
1 1 5
~~~

## Sample Output

~~~
4 8
10 5
8 8
10 6
10 2
1 8
7 8
10 6

1 3
2 1
1 3
~~~

## Solution

首先，我们判断出哪些舞者会发生碰撞：由于每个舞者都要等待 \\(t\_i s\\) 才开始移动的，所以我们可以假设其初设坐标为 \\( (-t\_i, x\_i) \\) or \\( (y\_i, -t\_i) \\)，那么所有舞者开始移动的时间都相同了，如果两个舞者发生碰撞，那么它们必定是在同一时刻到达了同一坐标，又移动速度相同，所以它们的起始坐标的 \\(x+y\\) 的值肯定也相同。<br>

那么我们可以把起始坐标 \\(x+y\\) 值相同的舞者划分为一组，它们之间每一个舞者至少会和另一个发生碰撞，通过观察发现，在碰撞点的连线上，发生碰撞的两个舞者在碰撞前后的相对位置不变，那么我们只要按相同的排序方法把舞者的初始坐标和最终坐标排个序，一一对应就OK了


## AC code
```c++
int n,w,h;
struct Node
{
    int x,y,g;
    Node(int x=0, int y=0, int g=0):x(x),y(y),g(g){}
}node[MaxN],ans[MaxN];

vector<int> stat[MaxN<<1];
vector<int> xs,ys;

void solve()
{
    for(int i=0;i<=2E5;i++) stat[i].clear();
    for(int i=1;i<=n;i++) 
    {
        int tmp = node[i].x+node[i].y+1E5;
        stat[tmp].pb(i);
    }
    for(int s=0;s<=2E5;s++)
    {
        if(stat[s].size()==0) continue;
        xs.clear(),ys.clear();
        for(auto id : stat[s])
        {
            if(node[id].g==2) ys.pb(node[id].y);
            else xs.pb(node[id].x); 
        }

        sort(xs.begin(), xs.end());
        sort(ys.begin(), ys.end());
        sort(stat[s].begin(), stat[s].end(), [](int a, int b)
        {
            if( node[a].g != node[b].g) return (node[a].g==2);
            else return node[a].g==2? node[a].y>node[b].y : node[a].x<node[b].x;
        });
        for(int j=0;j<xs.size();j++)
        {
            ans[stat[s][j]] = Node(xs[j], h);
        }
        for(int j=0;j<ys.size();j++)
        {
            ans[stat[s][j+xs.size()]] = Node(w,ys[ys.size()-1-j]);
        }
    }
    for (int i = 1; i <= n; ++i) printf("%d %d\n", ans[i].x, ans[i].y);
}

int main()
{
    while(~scanf("%d%d%d", &n, &w, &h))
    {
        int g,p,t;
        for(int i=1;i<=n;i++)
        {
            scanf("%d%d%d", &g, &p, &t);
            if(g==1) node[i] = Node(p,-t, g);
            else node[i] = Node(-t,p,g);
        }
        solve();
    }
}
```