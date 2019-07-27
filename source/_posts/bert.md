---
title: bert
date: 2019-07-27 15:52:02
tags: [nlp, bert, attention, self attention]
categories: machine learning
---


使用的图片来自于  http://jalammar.github.io/illustrated-transformer/  


```python
from comment_pretrain.utils.bert_util import CommentBertModelWrapper, GoogleBertModelWrapper
```

# 读入一个已经训练好的bert


```python
model_path="/mnt/comment/model/bert_models/en_bert_v0"
model = CommentBertModelWrapper(model_path)
```


```python
config = model.model.config
print(config)
```

    {
      "attention_probs_dropout_prob": 0.1,
      "hidden_act": "gelu",
      "hidden_dropout_prob": 0.1,
      "hidden_size": 256,
      "initializer_range": 0.02,
      "intermediate_size": 1024,
      "layer_norm_eps": 1e-12,
      "max_position_embeddings": 256,
      "num_attention_heads": 8,
      "num_hidden_layers": 8,
      "type_vocab_size": 2,
      "vocab_size": 50100
    }
    


vocab_size: 词表的大小
type_vocab_size: token的type，因为是两句话拼接起来的，用这个来区分是第一句话还是第二句话
hidden_size: 隐层的大小，token的embedding，position的embedding，token_type的embedding

以下是一整个bert的模型结构：

```python
model.model.bert
```




    BertModel(
      (embeddings): BertEmbeddings(
        (word_embeddings): Embedding(50100, 256, padding_idx=0)
        (position_embeddings): Embedding(256, 256)
        (token_type_embeddings): Embedding(2, 256)
        (LayerNorm): BertLayerNorm()
        (dropout): Dropout(p=0.1)
      )
      (encoder): BertEncoder(
        (layer): ModuleList(
          (0): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
          (1): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
          (2): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
          (3): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
          (4): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
          (5): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
          (6): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
          (7): BertLayer(
            (attention): BertAttention(
              (self): BertSelfAttention(
                (query): Linear(in_features=256, out_features=256, bias=True)
                (key): Linear(in_features=256, out_features=256, bias=True)
                (value): Linear(in_features=256, out_features=256, bias=True)
                (dropout): Dropout(p=0.1)
              )
              (output): BertSelfOutput(
                (dense): Linear(in_features=256, out_features=256, bias=True)
                (LayerNorm): BertLayerNorm()
                (dropout): Dropout(p=0.1)
              )
            )
            (intermediate): BertIntermediate(
              (dense): Linear(in_features=256, out_features=1024, bias=True)
            )
            (output): BertOutput(
              (dense): Linear(in_features=1024, out_features=256, bias=True)
              (LayerNorm): BertLayerNorm()
              (dropout): Dropout(p=0.1)
            )
          )
        )
      )
      (pooler): BertPooler(
        (dense): Linear(in_features=256, out_features=256, bias=True)
        (activation): Tanh()
      )
    )



## 输入处理

首先文本需要转化为token， token需要通过词典转换为id


```python
text = "you are soooocute"
print(f"原始文本:    {text}")
print(f"转化为token: {model.tokenizer.encode_pieces(text)}")
print(f"转化为id   : {model.tokenizer.tokenize(text)} ")
```

    原始文本:    you are soooocute
    转化为token: ['you', 'are', 'soooo', '##cute']
    转化为id   : [12221, 12323, 12856, 17824] 



```python
input_ids, segment_ids, input_masked = model.covert_text_to_features(text)
print(f"token ids: {input_ids}")
print(f"token types: {segment_ids}")
input_ids = torch.tensor(np.array([input_ids]))
segment_ids = torch.tensor(np.array([segment_ids]))
input_masked = torch.tensor(np.array([input_masked]))
print(f"input_ids的shape: {input_ids.shape}")
print(f"token type的shape: {segment_ids.shape}")
print(f"mask的shape: {input_masked.shape}")
```

    token ids: [3, 12221, 12323, 12856, 17824, 2]
    token types: [0, 0, 0, 0, 0, 0]
    input_ids的shape: torch.Size([1, 6])
    token type的shape: torch.Size([1, 6])
    mask的shape: torch.Size([1, 6])


# 模型

## embedding

```python
words_embeddings = self.word_embeddings(input_ids)
position_embeddings = self.position_embeddings(position_ids)
token_type_embeddings = self.token_type_embeddings(token_type_ids)

embeddings = words_embeddings + position_embeddings + token_type_embeddings
embeddings = self.LayerNorm(embeddings)
embeddings = self.dropout(embeddings)
```


```python
embedding_output = model.model.bert.embeddings(input_ids, segment_ids)
print(f"对输入id取embeding {model.model.bert.embeddings(input_ids, segment_ids).shape}")

```

    对输入id取embeding torch.Size([1, 6, 256])


## encoder

对输入做attention，然后一层全连接，然后将attention的输出和全连接的输出加在一起再过一层全连接，然后normalize，输出跟输入相同的shape, 这个就是一个encoder，然后这个过程重复多次，这个对应的就是参数中的 num_hidden_layers 

![](http://jalammar.github.io/images/t/transformer_resideual_layer_norm.png)

```python
attention_output = self.attention(hidden_states, attention_mask)
intermediate_output = self.intermediate(attention_output)
layer_output = self.output(intermediate_output, attention_output)
```


```python
attention_mask = input_masked.unsqueeze(1).unsqueeze(2).to(dtype=next(model.model.parameters()).dtype)
encoder0_output = model.model.bert.encoder.layer[0](embedding_output, attention_mask)
print(f"经过一层encoder的shape {encoder0_output.shape}")
```

    经过一层encoder的shape torch.Size([1, 6, 256])


###  attention



#### self attention

![](http://jalammar.github.io/images/t/transformer_self_attention_vectors.png)



![](http://jalammar.github.io/images/t/transformer_self_attention_score.png)



![](http://jalammar.github.io/images/t/self-attention_softmax.png)



![](http://jalammar.github.io/images/t/self-attention-output.png)


将单个attention写成矩阵形式

输入 $x \in R^{n*h}$ 每一行是一个token对应的向量

$Q$ 矩阵 $Q \in R^{h \* \hat{h}}$ ,  $K$ 矩阵 $K \in R^{h \* \hat{h}}$ ,  $V$ 矩阵 $V \in R^{h*\hat{h}}$ 这块输出长度取的 $\hat{h}$ 是 h / num_attention_head, 目的是为了在经过多头multihead attention之后每个token对应向量的长度跟之前是一样的。

然后通过计算 $q$ $k$ $v$

$q=Qx \in R^{n*\hat{h}}$  每行是一个token对应的向量

$k=Kx \in R^{n*\hat{h}}$  每行是一个token对应的向量

$v=Vx \in R^{n*\hat{h}}$  每行是一个token对应的向量

然后计算attention score，即某一个位置对其他每个位置的分数

$score = q \* k^T  \in R^{n \* n} $ 这个地方第一行是每个token对应的k和第一个token对应的q计算出来的score，$score$ 第一行的第一维需要取乘以第一个token对应的v，第二维度乘以第二个token对应的v，第三维乘以第三个token对应的v 写出来就是
$$
z_{1,:} = s_{11}  v_{1,:} + s_{12}  v_{2,:} + s_{13}  v_{3,:} + s_{14}  v_{4,:}
$$

$$
z_{2,:} = s_{21}  v_{1,:} + s_{22}  v_{2,:} + s_{23}  v_{3,:} + s_{24} v_{4,:}
$$

相当于第一个元素就是 

$$
z_{11} = s_{11}  v_{11} + s_{12} v_{21} + s_{13}v_{31} + s_{14}v_{41}
$$

写成矩阵形式就是 $SCORE*V$

完整的就是 

$$
Z=softmax(\frac{Q*K^T}{\sqrt{d_k}})V
$$

$$
Z \in R^{n*\hat{h}}
$$

multihead attention的做法就是相当于有多个$Q$ $K$ $V$ 分别去做上面的计算，计算出来 多个 $Z$ ，实际上就是使用列数更多的$Q$ $K$ $V$就实现列这个效果，计算的时候通过维度变换可以实现. 

经过attention计算后的token的向量会和输入concat，实现一个短路的操作，然后再做normalize再进行后续操作

因为经过短路操作和multi-head之后向量的宽度就发生变化来，不再是$h$了，为了能完全复用的送如下一层(同时也能表达更多的信息)，将维度映射回h。


```python
class BertSelfAttention(nn.Module):
    def __init__(self, config):
        super(BertSelfAttention, self).__init__()
        self.num_attention_heads = config.num_attention_heads
        self.attention_head_size = int(config.hidden_size / config.num_attention_heads)
        self.all_head_size = self.num_attention_heads * self.attention_head_size

        self.query = nn.Linear(config.hidden_size, self.all_head_size)
        self.key = nn.Linear(config.hidden_size, self.all_head_size)
        self.value = nn.Linear(config.hidden_size, self.all_head_size)

        self.dropout = nn.Dropout(config.attention_probs_dropout_prob)

    def transpose_for_scores(self, x):
        new_x_shape = x.size()[:-1] + (self.num_attention_heads, self.attention_head_size)
        x = x.view(*new_x_shape)
        return x.permute(0, 2, 1, 3)

    def forward(self, hidden_states, attention_mask):
        mixed_query_layer = self.query(hidden_states)
        mixed_key_layer = self.key(hidden_states)
        mixed_value_layer = self.value(hidden_states)

        query_layer = self.transpose_for_scores(mixed_query_layer)
        key_layer = self.transpose_for_scores(mixed_key_layer)
        value_layer = self.transpose_for_scores(mixed_value_layer)

        # Take the dot product between "query" and "key" to get the raw attention scores.
        attention_scores = torch.matmul(query_layer, key_layer.transpose(-1, -2))
        attention_scores = attention_scores / math.sqrt(self.attention_head_size)
        # Apply the attention mask is (precomputed for all layers in BertModel forward() function)
        attention_scores = attention_scores + attention_mask

        # Normalize the attention scores to probabilities.
        attention_probs = nn.Softmax(dim=-1)(attention_scores)

        # This is actually dropping out entire tokens to attend to, which might
        # seem a bit unusual, but is taken from the original Transformer paper.
        attention_probs = self.dropout(attention_probs)

        context_layer = torch.matmul(attention_probs, value_layer)
        context_layer = context_layer.permute(0, 2, 1, 3).contiguous()
        new_context_layer_shape = context_layer.size()[:-2] + (self.all_head_size,)
        context_layer = context_layer.view(*new_context_layer_shape)
        return context_layer
    
class BertSelfOutput(nn.Module):
    def __init__(self, config):
        super(BertSelfOutput, self).__init__()
        self.dense = nn.Linear(config.hidden_size, config.hidden_size)
        self.LayerNorm = BertLayerNorm(config.hidden_size, eps=config.layer_norm_eps)
        self.dropout = nn.Dropout(config.hidden_dropout_prob)

    def forward(self, hidden_states, input_tensor):
        hidden_states = self.dense(hidden_states)
        hidden_states = self.dropout(hidden_states)
        hidden_states = self.LayerNorm(hidden_states + input_tensor)
        return hidden_states
    
class BertAttention(nn.Module):
    def __init__(self, config):
        super(BertAttention, self).__init__()
        self.output_attentions = output_attentions
        self.self = BertSelfAttention(config)
        self.output = BertSelfOutput(config)

    def forward(self, input_tensor, attention_mask):
        self_output = self.self(input_tensor, attention_mask)
        self_output = self_output
        attention_output = self.output(self_output, input_tensor)
        return attention_output

```


```python
attention_head_size = int(config.hidden_size / config.num_attention_heads)
all_head_size = config.num_attention_heads * attention_head_size
print(f"每个self attention head 输出维度 {attention_head_size}")
print(f"所有的multi-head输出维度 {all_head_size}")
```

    每个self attention head 输出维度 32
    所有的multi-head输出维度 256



```python
Q = torch.nn.Linear(config.hidden_size, all_head_size)
K = torch.nn.Linear(config.hidden_size, all_head_size)
V = torch.nn.Linear(config.hidden_size, all_head_size)
print(f"Q的shape {Q}")
print(f"K的shape {K}")
print(f"V的shape {V}")
```

    Q的shape Linear(in_features=256, out_features=256, bias=True)
    K的shape Linear(in_features=256, out_features=256, bias=True)
    V的shape Linear(in_features=256, out_features=256, bias=True)



```python
def transpose_for_scores(x):
    new_x_shape = x.size()[:-1] + (config.num_attention_heads, attention_head_size)
    ## batch_size * seq_length * attention_head * attention_head_size
    x = x.view(*new_x_shape)
    ## batch_size * attention_head * seq_length * attention_head_size
    return x.permute(0, 2, 1, 3)
```


```python
hidden_states = embedding_output
mixed_query_layer = Q(hidden_states)
mixed_key_layer = K(hidden_states)
mixed_value_layer = V(hidden_states)

query_layer = transpose_for_scores(mixed_query_layer)
key_layer = transpose_for_scores(mixed_key_layer)
value_layer = transpose_for_scores(mixed_value_layer)

print(mixed_value_layer.shape)
print(value_layer.shape)

```

    torch.Size([1, 6, 256])
    torch.Size([1, 8, 6, 32])


实际上每个multi-head的输出应该是 32 维，但是因为将多个Q K V整合在了一起，这个地方输出的是256维，因此使用 transpose_for_scores 将维度进行变换，还原成 batch_size * num_attention_heads * seq_length * attention_head_size

然后使用矩阵乘都是在后两个维度进行，保持了batch size 和 head num 的不变


```python
attention_scores = torch.matmul(query_layer, key_layer.transpose(-1, -2))
attention_scores = attention_scores / math.sqrt(attention_head_size)
# Apply the attention mask is (precomputed for all layers in BertModel forward() function)
attention_scores = attention_scores + attention_mask
attention_probs = torch.nn.Softmax(dim=-1)(attention_scores)
print(f"计算出来的 scores 的维度 {attention_probs.shape}")
```

    计算出来的 scores 的维度 torch.Size([1, 8, 6, 6])


然后根据之前的公式，计算出输出, 然后再将维度变换到原来的形状  batch_size * seq_length * (context_layer * attention_head_size) = batch_size * seq_length * hidden_size


```python
context_layer = torch.matmul(attention_probs, value_layer)
context_layer = context_layer.permute(0, 2, 1, 3).contiguous()
new_context_layer_shape = context_layer.size()[:-2] + (all_head_size,)
context_layer = context_layer.view(*new_context_layer_shape)
print(f"在self attention计算结束后的输出 {context_layer.shape}")
```

    在self attention计算结束后的输出 torch.Size([1, 6, 256])


####  self attention output

这一层的目的是对self attention的输出进行一层全连接变换，然后与self attention的输入相加，过layer normalize 然后输出到后面


```python
hidden_states = model.model.bert.encoder.layer[0].attention.output.dense(context_layer)
hidden_states = model.model.bert.encoder.layer[0].attention.output.dropout(context_layer)
hidden_states = model.model.bert.encoder.layer[0].attention.output.LayerNorm(context_layer + hidden_states)

print(f"经过output这一层之后的shape {hidden_states.shape}")
```

    经过output这一层之后的shape torch.Size([1, 6, 256])


### 中间层

这一层就是简单的将attention的输出做一个全连接激活变换，然后输到下一层

```python
class BertIntermediate(nn.Module):
    def __init__(self, config):
        super(BertIntermediate, self).__init__()
        self.dense = nn.Linear(config.hidden_size, config.intermediate_size)
        if isinstance(config.hidden_act, str) or (sys.version_info[0] == 2 and isinstance(config.hidden_act, unicode)):
            self.intermediate_act_fn = ACT2FN[config.hidden_act]
        else:
            self.intermediate_act_fn = config.hidden_act

    def forward(self, hidden_states):
        hidden_states = self.dense(hidden_states)
        hidden_states = self.intermediate_act_fn(hidden_states)
        return hidden_states
```

### 输出层

这一层也比较简单，将中间层的输出再做一次全连接激活，然后跟中间层的输入相加做normalize输出

```python
class BertOutput(nn.Module):
    def __init__(self, config):
        super(BertOutput, self).__init__()
        self.dense = nn.Linear(config.intermediate_size, config.hidden_size)
        self.LayerNorm = BertLayerNorm(config.hidden_size, eps=config.layer_norm_eps)
        self.dropout = nn.Dropout(config.hidden_dropout_prob)

    def forward(self, hidden_states, input_tensor):
        hidden_states = self.dense(hidden_states)
        hidden_states = self.dropout(hidden_states)
        hidden_states = self.LayerNorm(hidden_states + input_tensor)
        return hidden_states
```


## Pooler

encoder的输出是 batch_size * seq_length * hidden_size的shape，这个seq_length实际上是变成的，而通常后面的层又不需要处理变长的结构，这个地方就需要做一个pooler,(有时候也可以用rnn这一系列的结构来处理，或者都拼成相同的长度). 最简单的 pooler方式，直接取第一个token。这个token肯定是存在的，不会因为长度的问题变得没有，实际上也可以取max pooling 或者 average pooling等等

```python
class BertPooler(nn.Module):
    def __init__(self, config):
        super(BertPooler, self).__init__()
        self.dense = nn.Linear(config.hidden_size, config.hidden_size)
        self.activation = nn.Tanh()

    def forward(self, hidden_states):
        # We "pool" the model by simply taking the hidden state corresponding
        # to the first token.
        first_token_tensor = hidden_states[:, 0]
        pooled_output = self.dense(first_token_tensor)
        pooled_output = self.activation(pooled_output)
        return pooled_output
```

很多下游任务都是套在这个pooling之后的结果上的，比如通常的分类就是在pooling之后再套一层全连接，预训练的时候的两个任务也都是接在这个 pooling的结果之后的。 需要相关的需要输出pooler前面的那一层

## 预训练任务

我们一般两个预训练任务，一个是语言模型，一个是预测下一个句子，这块实际上是可以根据要做的任务来进行修改的，可以选择更贴切的预训练任务或者增加更多的预训练任务。一般来说就是在 pooling后面再叠一部分就够了。


```python
class BertPreTrainingHeads(nn.Module):
    def __init__(self, config, bert_model_embedding_weights):
        super(BertPreTrainingHeads, self).__init__()
        self.predictions = BertLMPredictionHead(config, bert_model_embedding_weights)
        self.seq_relationship = nn.Linear(config.hidden_size, 2)

    def forward(self, sequence_output, pooled_output):
        prediction_scores = self.predictions(sequence_output)
        seq_relationship_score = self.seq_relationship(pooled_output)
        return prediction_scores, seq_relationship_score
```

预训练数据的产生非常简单，根据任务来就行，原本的就是两个句子使用seq隔开，开头加cls，末尾加sep，然后以15%的概率选择需要mask的词作为语言模型的label，mask的词80%概率换成 mask，10%的概率换成随机，10%的概率不变。


## 下游任务

比如分类，就是在pooling之后的向量上增加一个全连接的分类层就行来

```python
class BertForSequenceClassification(BertPreTrainedModel):
 
    def __init__(self, config, num_labels=2, output_attentions=False):
        super(BertForSequenceClassification, self).__init__(config)
        self.output_attentions = output_attentions
        self.num_labels = num_labels
        self.bert = BertModel(config, output_attentions=output_attentions)
        self.dropout = nn.Dropout(config.hidden_dropout_prob)
        self.classifier = nn.Linear(config.hidden_size, num_labels)
        self.apply(self.init_bert_weights)

    def forward(self, input_ids, token_type_ids=None, attention_mask=None, labels=None):
        outputs = self.bert(input_ids, token_type_ids, attention_mask, output_all_encoded_layers=False)
        if self.output_attentions:
            all_attentions, _, pooled_output = outputs
        else:
            _, pooled_output = outputs
        pooled_output = self.dropout(pooled_output)
        logits = self.classifier(pooled_output)

        if labels is not None:
            loss_fct = CrossEntropyLoss()
            loss = loss_fct(logits.view(-1, self.num_labels), labels.view(-1))
            return loss
        elif self.output_attentions:
            return all_attentions, logits
        return logits
```



