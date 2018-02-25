---
layout: post
current: post
cover:  assets/images/stock/images-stock-photo-Yb054.jpg
navigation: True
title: "AtCoder Regular Contest 081 E"
date:  2017-08-20 23:59:59
tags: [algorithm]
class: post-template
author: kiwi
comments: true
---

## Problem Information

source: [It's from the Regular Contest 081 of the AtCoder](http://arc081.contest.atcoder.jp/tasks/arc081_c)

## Description

给出一个字符串 \\(A\\) ，求出一个不属于它的子序列的最短字符串，若有多个则取字典序最小的。<br>

数据范围：<br>
* \\( 1 ≤ \\|A\\| ≤ 2 \\times 10^5 \\) <br>
* \\(A\\) consists of lowercase English letters.<br>


## Input

\\(A\\) <br>


## Output

Print the lexicographically smallest string among the shortest strings consisting of lowercase English letters that are not subsequences of \\(A\\). <br>

## Sample Input
 
~~~
atcoderregularcontest
abcdefghijklmnopqrstuvwxyz
frqnvhydscshfcgdemurlfrutcpzhopfotpifgepnqjxupnskapziurswqazdwnwbgdhyktfyhqqxpoidfhjdakoxraiedxskywuepzfniuyskxiyjpjlxuqnfgmnjcvtlpnclfkpervxmdbvrbrdn
~~~

## Sample Output
~~~
b
aa
aca
~~~

## Solution

首先，字典序最小可以通过从小到大确定答案来解决。那么，我们只需要知道怎么找到最短的非子序列字符串。 <br>

我们考虑任意一个非子序列的字符串，它一定是匹配到了原串的某一个位置，且该位置后的字符少于 \\(26\\) 种，
此时我们添加一个没有出现过的字符即可凑出一个非子序列的字符串。那么再要求字符串长度最短，那么我们可以从最后往前递推，确定每个字符到缺省字符的最短路径。<br>

## AC code

```c++
char str[MaxN];
int loc[MaxN][26], to[26], dep[MaxN];
void solve()
{
	int len = strlen(str);
	for(int i=0;i<26;i++) to[i] = len+1;
	for(int i=len;i>=0;i--)
	{
		memcpy(loc[i], to, sizeof(to));
		to[str[i-1]-'a'] = i;
	}
	for(int i=len;i>=0;i--)
	{
		dep[i] = len+1;
		for(int j=0;j<26;j++)
		{
			dep[i] = min(dep[i], dep[loc[i][j]]+1);
		}
	}
	int u = 0,v;
	while(u!=len+1)
	{
		for(v=0;v<26;v++)
		{
			if(cost[u]==dep[loc[u][v]]+1) break;
		}
		printf("%c", v+'a');
		u = loc[u][v];
	}
	puts("");
}
int main()
{
	while(~scanf("%s", &str))
		solve();
}
```