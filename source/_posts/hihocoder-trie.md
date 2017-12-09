title: hihocoder算法练习之trie树
date: 2015-10-11 12:07:05
tags : [hihocoder, algorithm, C++, trie树, 字典树]
categories: hihocoder
---

## 输入

输入的第一行为一个正整数n，表示词典的大小，其后n行，每一行一个单词（不保证是英文单词，也有可能是火星文单词哦），单词由不超过10个的小写英文字母组成，可能存在相同的单词，此时应将其视作不同的单词。接下来的一行为一个正整数m，表示询问的次数，其后m行，每一行一个字符串，该字符串由不超过10个的小写英文字母组成，表示一个询问。


## 输出

对于每一个询问，输出一个整数Ans,表示词典中以询问给出的字符串为前缀的单词的个数。


## 样例输入

	
	5
	babaab
	babbbaaaa
	abba
	aaaaabaa
	babaababb
	5
	babb
	baabaaa
	bab
	bb
	bbabbaab
	

## 样例输出

	
	1
	0
	3
	0
	0


## 代码

	
```cpp
#include <stdio.h>
#include <iostream>
#include <string>
#include <cstring>
using namespace std;
typedef struct trie_node{
	int m;
	trie_node * next[26];
} trie_node;
int main()
{
	trie_node * root=new trie_node;
	trie_node * current;
	trie_node * newnode;
	int n;
	int i;
	int flag=0;
	string s1;
	memset(root->next, 0, sizeof(root->next));
	root->m = 1;
	cin >> n;
	while (n--)
	{
		current = root;
		cin >> s1;
		for (i = 0; i < s1.length(); i++)
		{
			if (current->next[s1[i] - 'a'] == NULL)
			{
				newnode = new trie_node;
				memset(newnode->next, 0, sizeof(newnode->next));
				newnode->m = 1;
				current->next[s1[i] - 'a'] = newnode;
			}
			else
				((current->next[s1[i] - 'a'])->m)++;
			current = current->next[s1[i] - 'a'];
		}
	}
	cin >> n;
	while (n--)
	{
		current = root;
		flag = 0;
		cin >> s1;
		for (i = 0; i < s1.length(); i++)
		{
			if (current->next[s1[i] - 'a'] == NULL)
			{
				flag = 1;
				break;
			}
			else
				current = current->next[s1[i] - 'a'];
		}
		if (flag)
			cout << 0 << endl;
		else
			cout << current->m << endl;

	}
}
```