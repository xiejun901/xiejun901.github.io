title: word2vec 笔记
date: 2017-05-09 00:05:47
categories: ml
tags: [word2vec, ml]
---

## 准备知识

### sigmoid 函数

在机器学习中，sigmoid 函数非常常见，常用来作为神经网络的激活函数

$$
\begin{equation}
\sigma(x)= \frac{1}{1 + e^ {-x} } 
\end{equation}
$$
$$
\begin{align}
{\sigma ^ {\prime}}(x) &= \frac{e ^ {-x} }{(1 + e^ {-x} )^2 }\\
& = \frac{(1 + e ^ {-x}) - 1}{(1 + e ^ {-x}) ^ 2} \\
& = \sigma(x)\left [ 1-\sigma(x) \right ]
\end{align}
$$
$$
\begin{equation}
\left[log\sigma(x) \right] ^ \prime = 1-\sigma(x)
\end{equation}
$$
$$
\begin{equation}
\left[log(1-\sigma(x))\right] ^ \prime = - \sigma(x)
\end{equation}
$$

### Huffman 编码

huffman编码是根据字符(单词)出现的次数来构建二叉树，然后将出现频次更高的字符(单词)使用更短的二进制编码来表示的一种编码方法。 

#### huffman树的构造

- 1.统计字符出现的次数，每个字符构成一个结点，结点的权值为字符出现的次数，所有单个结点的树构成森林
- 2.选取森林中根结点权值最小的两棵树，构成一颗新的树，新的树的根结点权值为选出来的两颗树根结点权值之和。
- 3.删除选出的两棵树，加入新构成的树到森林中
- 4.重复步骤2和3直到森林中只有一棵树为止

#### 编码

根据生成的huffman树，从根结点开始，将每个结点的左结点编码为1(或0), 右结点编码为0(活1),可以得到每个字符(叶子结点)的编码，显然词频越高的编码越短。

####  notation

为了在后续 word2vec 原理中使用，对于一颗 huffman 树，定义以下符号

对于每个叶子结点 w

- $p^w$: 从根结点到 $w$ 叶子结点的路径
- $l^w$: 从根结点到叶子结点包含的结点个数
- $d_j^w$: 词w的huffman编码第 $j-1$ 位编码(即从根结点开始数第j个结点的编码), $j$ 的取值范围是 [2, $l^w$]
- $\theta_j^w$: $p^w$路径上从根几点开始第 $j$ 个结点对应的向量，$j$ 的取值范围是 [1, $l^w - 1$]

## Word2Vec

### 原理

对于一个句子 $W=w_1 w_2 w_3 w_4 ... w_T$,其中 $w$ 表示词 

$$
\begin{align}
p(W) & = p(w_1, w_2, w_3, w_4, ... , w_T) \\
& = p(w_1)p(w_1|w_2)p(w_3|w_1,w_2)p(w_4|w_1,w_2,w_3) ... p(w_T|w_1, w_2, w_3, w_4, ... , w_{T-1}) \\
& = p(w_1)p(w_2|w_1)p(w_3|w_1^2)p(w_4|w_1^3) ... p(w_T|w_1^{T-1})
\end{align}
$$

如果提前求好 $p(w_k|w_1^{k-1})$ 的值，然后就可以很方便的计算出 $p(W)$了, 通常利用以下公式

$$
p(w_k|w_1^{k-1})=\frac{p(w_1^k)}{p(w_1^{k-1})} \approx \frac{count(w_1^k)}{count(w_1^{k-1})}
$$

在 n-gram 算法中，是假设每个词只与前$n-1$个词有关系，即用以下公式来计算

$$p(w_k|w_1^{k-1}) \approx p({w_k|w_{k-n+1}^{k-1}})  = \frac{p(w_{k-n+1}^k)}{p(w_{k-n+1}^{k-1})} \approx \frac{count(w_{k-n+1}^k)}{count(w_{k-n+1}^{k-1})} $$

这n-gram中，复杂度为$O(N^n)$, 所以n通常取得较小

在 Word2Vec 算法中，是通过一个三层的神经网络来建模$p(w|context(w))$, 通过训练来得到模型, 然后将输入通过训练好的模型之后即可得到概率。

训练过程即是去优化一个目标函数，采用最大对数似然，目标函数设为

$$
L=\sum_{w \in C}{logp(w|context(w))}
$$

C表示的是语料库中的所有的词, $context(w)$ 表示是 $w$ 的上下文，可以是 $w$ 之前的几个词，也可以是 $w$ 之后和之后的几个词。

根据 使用的优化的是 $L=\prod_{w \in C}{logp(w|context(w))}$ 还是 L=\prod_{w \in C}{logp(context(w)|w)} 可以将模型分为 CBOW(Continuos Bag-Of-Word Model) 和 Skip-gram (Continuos Skip-gram Model)

### CBOW 模型

CBOW 模型即优化的目标函数式 $L=\prod_{w \in C}{logp(w|context(w))}$, 模型的输入是$context(x)$
模型有三层, 假设单词的词向量的长度为m

#### 层次结构

- 第一层是输入层, 包含了$context(x)$ 中的单词的词向量, 假设context由$w$的前c个单词和后c个单词构成, 则这里的输入层是2c个单词的词向量
- 第二层是投影层, 将2c个单词的的词向量映射为1个长度为m的向量, 具体的做法是

$$
x_w = \sum_{i=1}^{2c}v(context(w)_i)
$$

- 第三层数输出层, 是一颗由语料中的词构造出的 Huffman 树

#### Hierarchical Softmax

Hierarchical Softmax 是 Word2Vec 提高性能的一项关键技术, 通过词库的词建立的 Huffman 树来计算梯度。
对于路径$p^w$中除开根结点的点, 都对应着单词 $w$ 的 Huffman 编码中的一位(0或者1), 对于 $p^w$中除开叶子结点的每一个结点，我们都可以训练一个分类器, 分类器的输出表示的是预测结果为正例的概率。对于一个单词$w$, 其 Huffman 编码构成了一个序列${d_2^w, d_3^w, d_4^w, ... , d_{l^w}^w}$, 对于其中的某一位$d_j^w$, 分类器输出是$d_j^w$的概率是

$$
p(d_j^w| x_w, \theta_{j-1^w}) = 
\begin{cases}
\sigma(x_w^T  \theta_{j-1}^w), & \text{$d_j^w = 1$} \\
\sigma(x_w^T  \theta_{j-1}^w), & \text{$d_j^w = 0$}
\end{cases} 
$$

当然，如果在定义 $d_j^w = 1$ 作为负例的，上式就可以写成

$$
p(d_j^w| x_w, \theta_{j-1^w}) = 
\begin{cases}
\sigma(x_w^T  \theta_{j-1}^w), & \text{$d_j^w = 0$} \\
1 - \sigma(x_w^T  \theta_{j-1}^w), & \text{$d_j^w = 1$}
\end{cases} 
$$

也可以整体写成

$$
p(d_j^w| x_w, \theta_{j-1^w}) = {\sigma(x_w^T  \theta_{j-1}^w)}^{1- d_j^w}{[1 - \sigma(x_w^T  \theta_{j-1}^w)]^{d_j^w}}
$$

用每个分类器来表示词$w$中的1位, 那么对于所有的 $l^w-1$个分类器, 在输入为$context(w)$, 输出为$w$的概率为

$$
p(w|context(w)) = \prod_{i=2}^{l^w} p(d_j^w|x_w, \theta_{j-1^w})
$$  

带入对数似然目标函数

$$
\begin{align}
L & = \sum_{w \in C}{log[p(w|context(w))]} \\
& = \sum_{w \in C}{log[\prod_{i=2}^{l^w} ({\sigma(x_w^T  \theta_{j-1}^w)}^{1- d_j^w}{[1 - \sigma(x_w^T  \theta_{j-1}^w)]^{d_j^w}}))]} \\
& = \sum_{w \in C}\sum_{i=2}^{l^w} \lbrace {(1- d_j^w) log{\sigma(x_w^T  \theta_{j-1}^w)} + {d_j^w}log[1 - \sigma(x_w^T  \theta_{j-1}^w)]} \rbrace
\end{align}
$$

取双重求和符号里面的部分进行推导，即

$$
L(w, j) = {(1- d_j^w) log{\sigma(x_w^T  \theta_{j-1}^w)} + {d_j^w}log[1 - \sigma(x_w^T  \theta_{j-1}^w)]}
$$

$$
\begin{align}
\frac{ \partial L(w, j)}{\partial \theta_{j-1}^w} 
& = (1 - d_j^w)[1 - \sigma(x_w^T  \theta_{j-1}^w)]x_w + {d_j^w}[-\sigma(x_w^T  \theta_{j-1}^w)]x_w \\
& = x_w [ 1 - d_j^w - \sigma(x_w^T  \theta_{j-1}^w) ]
\end{align}
$$

同理, 

$$
\begin{align}
\frac{ \partial L(w, j)}{\partial x_w} 
& = \theta_{j-1}^w [ 1 - d_j^w - \sigma(x_w^T  \theta_{j-1}^w) ]
\end{align}
$$

可以得到 $\theta_{j-1}^w$ 的更新公式为

$$
\theta_{j-1}^w := \theta_{j-1}^w + \eta x_w [ 1 - d_j^w - \sigma(x_w^T  \theta_{j-1}^w) ]
$$

对于词向量的更新则是根据所有非叶子结点计算出来的梯度的和去对上下文中的每个词进行更新

$$
v(w) := v(w) + \eta \sum_{i=2}^{l^w} {\theta_{j-1}^w [ 1 - d_j^w - \sigma(x_w^T  \theta_{j-1}^w) ]}
$$

#### 梯度更新伪代码

```
for w in C:
    for w in context(w):
        x += v(w)
    e = 0
    for l in 2 to l_w:
        temp = 1 - d_w(l) - sigma(x*theta)
        e += eta * theta * temp
        theta += eta * x * temp
    for w in context(w):
        v(w) += e
```

#### 实现



