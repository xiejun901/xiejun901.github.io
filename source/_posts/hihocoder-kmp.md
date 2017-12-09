title: hihocoder算法练习之kmp算法
date: 2015-10-11 15:53:09
tags: [hihocoder, algorithm, C++, kmp, next数组]
categories: hihocoder
---

## 输入

第一行一个整数N，表示测试数据组数。接下来的N*2行，每两行表示一个测试数据。在每一个测试数据中，第一行为模式串，由不超过10^4个大写字母组成，第二行为原串，由不超过10^6个大写字母组成。其中N<=20

## 输出

对于每一个测试数据，按照它们在输入中出现的顺序输出一行Ans，表示模式串在原串中出现的次数。

## 样例输入

	
	5
	HA
	HAHAHA
	WQN
	WQN
	ADA
	ADADADA
	BABABB
	BABABABABABABABABB
	DAD
	ADDAADAADDAAADAAD
	
## 样例输出

	
	3
	1
	3
	1
	0

## 说明


判断给出的一串字符串（原串）中是否含有特定的字符串（模式串），对于原串s="bababababababababb"，模式串p="bababb"，按照最普通的匹配方法

当匹配到i=5,j=5时候无法匹配
	
	b a b a b a b a b a b a b a b a b b
	b a b a b b
	
此时需要使i=1,j=0;
	
	b a b a b a b a b a b a b a b a b b
	  b a b a b b
	  
继续失配一直到i=2,j=0又能继续搭配上：
		
	b a b a b a b a b a b a b a b a b b
	    b a b a b b
	  
我们可以观察到p0p1p2==p2p3p4;所以当匹配到i=5,j=5失配时，因为这个时候有p0p1p2p3p4=s0s1s2s3s4,所以此时应该用p0去与s2对齐进行匹配，即在i=5,j=5之后下一个需要匹配的是i=5,j=3;
	
	b a b a b a b a b a b a b a b a b b
	    b a b a b b
		
即在匹配到失配时，i保持不变等于5，j取"p0p1p2p3p4"前缀与后缀相等的最长长度，3。 对于字符串"bababb"有：next[4]==3;

## KMP算法

	首先对字符串进行匹配，在原串的位置i，模式串的位置j,发现失配了。此时i保持不变，j=next[j-1],由nex数组性质可以得到p0p1...p[j-1]是跟原串能完全匹配上的，所以只需要继续匹配p[j]与s[i]即行.
	
## next数组求解

求解next数组可以采用递归的方法，对于"bababb"，
	
	i=1,j=0;
	b a b a
	  b
	不等且j=0;因此next[1]=0;然后继续求解下一个next
	i=2,j=0;
	b a b a 
	    b
	匹配上了，j++,next[2]=j=1;继续求解下一个next
	i=3,j=1;
	b a b a b b
	    b a
	匹配上了，j++,next[3]=j=2;继续求解下一个next
	i=4,j=2;
	b a b a b b
	    b a b
	匹配上了，j++,next[3]=j=3;继续求解下一个next
	i=5,j=3;
	b a b a b b
	    b a b a 
	失配，j=next[j-1]=1,继续匹配
	b a b a b b 
	        b a
	失配，j=next[j-1]=0,继续匹配 
	b a b a b b
	          b
	匹配上了，j++，next[5]=1。(这儿如果是匹配不上的话，因为j=0，所以会直接得到next[5]=0)
	
以上为一个求解过程的示例


## 代码

	
```c++
#include <iostream>
#include <string>
using namespace std;
void getnext(string s, int * next);
int main()
{
	string s;
	string p;
	int n;
	int i, j, count;
	cin>>n;
	while (n--)
	{
		cin >> p >> s;
		int * next = new int[p.length()];
		getnext(p, next);
		i = 0;
		j = 0;
		count = 0;
		while (i < s.length())
		{
			while (j != 0 && s[i] != p[j])
			{
				j = next[j - 1];
			}
			if (s[i] == p[j])
			{
				if (j == p.length() - 1)
				{
					j = next[j]; 
					count++;
				}
				else
					j++;
			}
			i++;
		}
		cout << count << endl;


	}
	return 0;


}
void getnext(string s, int * next)
{

	next[0] = 0;
	int i = 1;
	int j = 0;
	while (i < s.length())
	{
		while ((j != 0) && (s[i] != s[j]))
			j = next[j-1];
		if (s[i] == s[j])
			j++;
		next[i] = j;
		i++;
	}

}
```
