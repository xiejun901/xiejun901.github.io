title: hihocoder算法练习之trie图
date: 2015-10-11 19:21:06
tags: [hihocoder, algorithm, C++, trie图]
categories: hihocoder
---

## 输入

每个输入文件有且仅有一组测试数据。

每个测试数据的第一行为一个整数N，表示河蟹词典的大小。

接下来的N行，每一行为一个由小写英文字母组成的河蟹词语。

接下来的一行，为一篇长度不超过M，由小写英文字母组成的文章。

对于60%的数据，所有河蟹词语的长度总和小于10, M<=10

对于80%的数据，所有河蟹词语的长度总和小于10^3, M<=10^3

对于100%的数据，所有河蟹词语的长度总和小于10^6, M<=10^6, N<=1000

## 输出

对于每组测试数据，输出一行"YES"或者"NO"，表示文章中是否含有河蟹词语。

## 样例输入


	6
	aaabc
	aaac
	abcc
	ac
	bcd
	cd
	aaaaaaaaaaabaaadaaac
	

## 样例输出


	YES


## 说明


对于一本词典，要找一篇文章中是否含有这本词典中的词语，如果依次枚举每个词典中的每个单词，然后与文章进行匹配，如果词典单词数量是N，每个单词长度是L，文章长度为M，复杂度是O(MLN)。若采用KMP算法进行匹配，那么需要进行的次数也是O(NM).如果将词典构成trie树，枚举文章每一个单词作为起点，然后去trie树中走，如果能从某条变走到终点结点，则说明文章中存在词典中的单词，这种方式复杂度是O(ML)。

我们可以发现，采用trie树这种方式是可以优化的，如下图，构造了一本字典，含有的单词是ab,bc,aba,bab,bca

![](http://xiejun901.qiniudn.com/hihocoder1036trie.jpg)

我们要查看abcbca是否在词典中，采用trie树来进行枚举会经历以下过程

	枚举s[0]=a;0->1;1->3;3的c边不存在；回朔
	枚举s[1]=b;0->b->5;5的b边不存在；回朔
	枚举s[2]=c;0的c边不存在；回朔
	枚举s[3]=b;0->2->5->8;找到两个终点借点
	枚举s[4]=c;0的c边不存在；回朔
	枚举s[5]=a;0->1；字符串完，且1不为为终点结点。

考虑字符串abcac,0->1->3,此时发现1的c结点不存在，按照以上方法，则需要回朔到根结点，然后对比bcac这个字符串，过程是0->1->3=>0->2->5->8，但是1->3,b这条边已经走过了，所以最好是直接在0->1->3之后，直接跳到2结点，即：0->1->3(2)->5->8.所以，对于每一个结点，如果我们能求得它括号里边那个结点，就能大幅度优化这个算法了。括号里面那个结点，我们称之为后缀结点。

对于按照 aba这样一条边走到的结点的后缀结点，应该是按照ba走到的结点，即对于s[0...n]走到的结点的后缀结点为按照s[1...n]走到的结点。对于上图有则有以下结论，next(i)表示i结点的后缀结点。

	next(0)=0;
	next(1)=0;#"a"去掉第一个字符为空。
	next(2)=0;
	next(3)=2;#"ab"去掉第一个字符为"b"
	next(4)=1;#"ba"              "a"
	next(5)=?;
	
在计算next(5)的时候我们发现，"bc"去掉第一个字符"c",在原树中找不到，这种情况下，其实说明了以c开头的枚举都不能找到，那么我们直接跳到"bc"的后缀"c"再取后缀" "就行了，用结点8来举例：

	"bca"的后缀"ca",因为0的c边不存在所以直接取"a"， 即next(8)=1

如果在"bca"过程中有终点结点，为了不漏掉，我们引入了“后缀结点为标记结点的结点也是标记结点”理解这一段可以参考一下[提示三：如何求解Trie树中每个结点的后缀结点](http://hihocoder.com/problemset/problem/1036) 里面有更详细的解释。（懒得画图，就不在这详细说明了。。。。。）举个例子说明一下是怎样把一颗trie树中的结点的后缀结点求出来，补充完整为空的结点构成trie图的：

![trie2](http://xiejun901.qiniudn.com/hihocoder1036trie2.jpg)

ni,表示结点i，ni->son[a]表示结点i沿a边走的结点，ni->next表示结点i的后缀结点。

	n0->next=n0;补全不存在的边：n0->son[d]=n0;
	n1->next=n0;补全不存在的边：n1->son[b]=n1->next->son[b]=n2;n1->son[d]=n1->next->son[d]=n0;
	n2->next=n0;补全不存在的边：n2->son[a]=n2->next->son[a]=n1;......
	n3->next=n0;补全不存在的边：n3->son[a]=n3->next->son[a]=n1;......
	n4->next=n4->p->next->son[a]=n1;补全不存在的边：......
	n5->next=n5->p->next->son[c]=n3;补全不存在的边：......
	......
	
算法描述如下：

	%
	%
	
c语言实现函数如下：

```c
void trieSolveNext(triePtr root)
{
	int i;
	qList queue1;
	triePtr node;
	memset(&queue1, 0, sizeof(queue1));
	node = root;
	queuePushback(&queue1, node);
	node->next = node;
	while (queue1.front != NULL)
	{
		node = queuePopfront(&queue1);
		if (node->next->isTerminal == 1)
			node->isTerminal = 1;
		for (i = 0; i < N; i++)
		{
			if (node->son[i] != NULL)
			{
				queuePushback(&queue1, node->son[i]);
				if (node == root)
					node->son[i]->next = node;
				else
					node->son[i]->next = node->next->son[i];
			}
			else
			{
				if (node == root)
					node->son[i] = node;
				else
					node->son[i] = node->next->son[i];
			}
		}
	}
}
```

在构建好trie图之后，只需要从根结点开始，沿着给出的文章的字符往下走，如果走到了标记结点，则表明文章中有字典给出的单词，如果到文章走完都没遇到标记结点，那么则说明无字典中的单词

```c
int searchTrie(triePtr node, char *s)
{
	while (*s)
	{
		if (node->isTerminal == 1)
			return 1;
		node = node->son[*s - 'a'];
		s++;
	}
	if (node->isTerminal == 1)
		return 1;
	else
		return 0;
}
```


## 整体代码


```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#define N 26
typedef struct tnode{
	int isTerminal;
	int num;
	struct tnode *p;
	struct tnode * son[N];
	struct tnode *next;
} trieNode;
typedef trieNode *triePtr;
typedef struct qnode{
	triePtr key;
	struct qnode * next;
} qNode;
typedef qNode *qPtr;
typedef struct qlist{
	qPtr front, back;
} qList;
typedef qList *qListPtr;
triePtr trieInsert(triePtr node, char * s);
int prefixCount(triePtr node, char *s);
void queuePushback(qListPtr q, triePtr key);
triePtr queuePopfront(qListPtr q);
void trieBfs(triePtr node);
void trieSolveNext(triePtr root);
int searchTrie(triePtr node, char *s);
int main()
{
	int m;
	char s[100];
	int temp;
	triePtr root = NULL;
	root = (triePtr)malloc(sizeof(trieNode));
	memset(root, 0, sizeof(trieNode));
	//trieInsert(root, "aaabc");
	//trieInsert(root, "aaac");
	//trieInsert(root, "abcc");
	//trieInsert(root, "ac");
	//trieInsert(root, "bcd");
	//trieInsert(root, "cd");
	scanf("%d", &m);
	while (m--)
	{
		scanf("%s", s);
		trieInsert(root, s);
	}
	scanf("%s", s);
	//trieBfs(root);
	trieSolveNext(root);
	temp = searchTrie(root, s);
	if (temp == 1)
		printf("YES");
	else
		printf("NO");
	return 0;
}

triePtr trieInsert(triePtr node, char * s)
{
	triePtr newNode;
	//	int i;
	while (*s)
	{
		if (node->son[*s - 'a'] == NULL)
		{
			newNode = (triePtr)malloc(sizeof(trieNode));
			memset(newNode, 0, sizeof(trieNode));
			newNode->p = node;
			node->son[*s - 'a'] = newNode;
		}
		node = node->son[*s - 'a'];
		node->num++;
		s++;
	}
	node->isTerminal = 1;
	return node;
}

int prefixCount(triePtr node, char *s)
{
	while (*s)
	{
		if (node->son[*s - 'a'] == NULL)
			return 0;
		node = node->son[*s - 'a'];
		s++;
	}
	return node->num;
}

void queuePushback(qListPtr q, triePtr key)
{
	qPtr newPtr;
	newPtr = (qPtr)malloc(sizeof(qNode));
	newPtr->key = key;
	newPtr->next = NULL;
	if ((*q).front == NULL)
	{
		(*q).front = (*q).back = newPtr;
	}
	else
	{
		(*q).back->next = newPtr;
		(*q).back = newPtr;
	}
}

triePtr queuePopfront(qListPtr q)
{
	triePtr newPtr;
	qPtr tempPtr;
	newPtr = (*q).front->key;
	tempPtr = (*q).front;
	(*q).front = tempPtr->next;
	free(tempPtr);
	return newPtr;
}

void trieBfs(triePtr node)
{
	int i;
	qList queue1;
	memset(&queue1, 0, sizeof(queue1));
	queuePushback(&queue1, node);
	while (queue1.front != NULL)
	{
		node = queuePopfront(&queue1);
		printf("%p\n", node);
		for (i = 0; i < N; i++)
		{
			if (node->son[i] != NULL)
			{
				queuePushback(&queue1, node->son[i]);
			}
		}

	}
}

void trieSolveNext(triePtr root)
{
	int i;
	qList queue1;
	triePtr node;
	memset(&queue1, 0, sizeof(queue1));
	node = root;
	queuePushback(&queue1, node);
	node->next = node;
	while (queue1.front != NULL)
	{
		node = queuePopfront(&queue1);
		if (node->next->isTerminal == 1)
			node->isTerminal = 1;
		for (i = 0; i < N; i++)
		{
			if (node->son[i] != NULL)
			{
				queuePushback(&queue1, node->son[i]);
				if (node == root)
					node->son[i]->next = node;
				else
					node->son[i]->next = node->next->son[i];
			}
			else
			{
				if (node == root)
					node->son[i] = node;
				else
					node->son[i] = node->next->son[i];
			}
		}
	}
}

int searchTrie(triePtr node, char *s)
{
	while (*s)
	{
		if (node->isTerminal == 1)
			return 1;
		node = node->son[*s - 'a'];
		s++;
	}
	if (node->isTerminal == 1)
		return 1;
	else
		return 0;
}
```
