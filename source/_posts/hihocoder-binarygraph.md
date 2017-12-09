categories : hihocoder
title: hihocoder算法练习之二分图
tags : [hihocoder, algorithm, C++, 二分图, 二分图判定, 最大匹配]
date: 2015-10-11 19:31:05
---

# 二分图判定


## 输入

第1行：1个正整数T(1≤T≤10)
接下来T组数据，每组数据按照以下格式给出：
第1行：2个正整数N,M(1≤N≤10,000，1≤M≤40,000)
第2..M+1行：每行两个整数u,v表示u和v之间有一条边


## 输出


第1..T行：第i行表示第i组数据是否有误。如果是正确的数据输出”Correct”，否则输出”Wrong”

## 样例输入


	2
	5 5
	1 2
	1 3
	3 4
	5 2
	1 5
	5 5
	1 2
	1 3
	3 4
	5 2
	3 5


## 样例输出


	Wrong
	Correct

## 说明


根据输入生成图，然后按照bfs或者dfs对顶点进行着，如果相邻的两个顶点为相同颜色，则判定不为二分图，注意数据中的非连通图。


## 代码


```
#include<iostream>
#include<vector>
#include<cstring>
#include<stack>
#define M 40000
#define N 10000
#define gray 0
#define black 1
#define white -1
int bfsGraph(int(&vertex)[N + 1], std::vector<int>(&vec)[N + 1], int k);
int main()
{
	int t;
	std::cin >> t;
	while (t--)
	{
		int n, m;
		int u, v;
		int flag = 1;
		std::vector<int> vec[N + 1];
		int vertex[N + 1];
		std::memset(vertex,0,sizeof(vertex));
		std::cin >> n >> m;
		for (int i = 0; i < m; i++)
		{
			std::cin >> u >> v;
			vec[u].push_back(v);
			vec[v].push_back(u);
		}
		for (int i = 1; i <= n; i++)
		{
			if (vertex[i] == gray)
			{
				if (!bfsGraph(vertex, vec, i))
				{
					flag = 0;
					break;
				}
			}
		}
		if (flag)
			std::cout << "Correct" << std::endl;
		else
			std::cout << "Wrong" << std::endl;
	}
	return 0;
}
int bfsGraph(int (&vertex)[N + 1], std::vector<int> (&vec)[N + 1],int k)
{
	std::stack<int> s;
	int index=k;
	vertex[index] = black;
	s.push(index);
	while (!s.empty())
	{
		index = s.top();
		s.pop();
		for (std::vector<int>::iterator iter = vec[index].begin(); iter != vec[index].end(); ++iter)
		{
			if (vertex[*iter] == gray)
			{
				vertex[*iter] = 0 - vertex[index];
				s.push(*iter);
			}
			else
			{
				if (vertex[*iter] + vertex[index] != 0)
					return 0;
			}
		}
	}
	return 1;
}
```

# 二分图--最大匹配


## 描述


给定一个二分图G，在G的一个子图M中，M的边集中的任何两天边都不依附于同一个顶点，则称M是一个匹配，选择这样的子集中边数最大的子集称为图的最大匹配问题

增光路(增广轨，交错轨)：P（增光轨）的顶点为未匹配点，属于M的边和不属于M的边在P上交错出现。

匈牙利算法：找到增广路，并将增广路置反，每进行一次可以使M的匹配书加1

如果所示：

![](http://xiejun901.qiniudn.com/hihocoder二分图最大匹配.jpg)

![](http://xiejun901.qiniudn.com/hihocoder二分图最大匹配2.jpg)

如上图，在对点5进行判断时，发现点5‘已经于点4连接，此时我们需要递归对点4进行增广轨判断，如果点4的增光轨存在，我们需要将5-5’加入M，5’-4从M中去掉。递归会一直持续到判断点1，可以发现与点1相连的点1’未被匹配，因此结束递归。

伪代码：
	
	finpath(u)
		for v connect u
			if(v未被访问过)
				v置访问
				if(v未被匹配 || findpath(v))
					将u-v加入M
					return 1
		return 0；



## 完整代码


```
#include<iostream>
#include<vector>
#include<cstring>
#include<queue>
#define N 10+1
#define M 10+1
#define gray 0
#define black 1
#define white 0
using namespace std;
int vertex[N] = { 0 };
int match[N] = { 0 };
int visit[N] = { 0 };
vector<int> adjList[N];
int n, m;
void insertSide(int u, int v);
void drawColor();
int findPath(int u);
int main()
{
	cin >> n>>m;
	int u, v;
	int count=0;
	for (int i = 0; i < m; i++)
	{
		cin >> u >> v;
		insertSide(u, v);
	}
	drawColor();
	for (int i = 1; i <= n; i++)
	{
		if (vertex[i] == black)
		{
			memset(visit, 0, sizeof(visit));
			count += findPath(i);

		}
	}
	cout << count << endl;
	return 0;

}

void insertSide(int u, int v)
{
	adjList[u].push_back(v);
	adjList[v].push_back(u);
}
void drawColor()
{
	memset(vertex, 0, sizeof(vertex));
	for (int i = 1; i <= n; i++)
	{
		if (vertex[i] == gray)
		{
			vertex[i] = black;
			queue<int> q;
			q.push(i);
			while (!q.empty())
			{
				int index = q.front();
				q.pop();
				for (vector<int>::iterator iter = adjList[index].begin(); iter != adjList[index].end(); ++iter)
				{
					if (vertex[*iter] == gray)
					{
						q.push(*iter);
						vertex[*iter] = 0 - vertex[index];
					}
				}
			}			
		}
	}
}
int findPath(int u)
{
	for (vector<int>::iterator iter = adjList[u].begin(); iter != adjList[u].end(); ++iter)
	{
		if (!visit[*iter])
		{
			visit[*iter] = 1;
			if (findPath(match[*iter])||match[*iter] == 0)
			{
				match[u] = *iter;
				match[*iter] = u;
				return 1;
			}
		}
	}
	return 0;
}
```

