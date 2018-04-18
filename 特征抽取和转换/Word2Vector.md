# Word2Vector

[Word2Vector](https://code.google.com/p/word2vec/)将词转换成分布式向量。分布式表示的主要优势是相似的词在向量空间距离较近，这使我们更容易泛化新的模式并且使模型估计更加健壮。

分布式的向量表示在许多自然语言处理应用（如命名实体识别、消歧、词法分析、机器翻译）中非常有用。

## 1 基本概念

### 1.1 词向量

一种简单直观的词表示方法是==One-hot==编码，用N位对N个词编码，每个词对应的N维向量中，只有一维为0。这种方式的缺陷是==词汇鸿沟==，即词与词之间相互孤立，忽略了它们之间的联系。而word2vec中的词向量是一种distributed representation，它的特点是**利用距离刻画词之间的相似性**。

### 1.2 统计语言模型

统计语言模型是**用来计算一个句子的概率的概率模型**，经典的有`HAL`,`LSA`,`COALS`等。通常它基于一个语料库来构建。

#### 句子的概率

$w_1^T$ 表示给定一个由T个词$(w_1,w_2,…,w_T)$按顺序构成的句子，根据Bayes，计算该句子的联合概率 $p(W)$:

$p(W)=p(w_1^T)=p(w_1,w_2,…,w_T)=p(w_1)∗p(w_2|w_1)∗p(w_3|(w_1^2))…p(w_T|(w_1^{T-1}))$

其中等式中的各个因子（条件概率）就是**语言模型的参数**，常用的语言模型都是在**近似**求 $p(W)$。如果给定一个词典大小为N的语料库，考虑长度为T的任意句子，理论上有$(N^T)$种可能组合，而每个句子需要计算T个参数，总共需要计算$(T*N^T)$个参数。无论是计算或保存，都需要很大开销。

#### n-gram模型

考虑 $p(w_k|w_1^{k-1})\ (k>1)$ 的近似计算，利用Bayes公式，有

$p(w_k|w_1^{k-1})=\frac{p(w_1^k)}{p(w_1^{k-1})}$

根据大数定理，当语料库足够大时，$p(w_k|w_1^{k-1})$ 可近似地表示为

$p(w_k|w_1^{k-1})\approx \frac{count(w_1^k)}{count(w_1^{k-1})}$

其中，$count(w_1^k)$ 和 $count(w_1^{k-1})$ 分别表示词串 $w_1^k$ 和 $w_1^{k-1}$ 在语料中出现的次数

## 2 模型

在`MLlib`中，`Word2Vector`使用`skip-gram`模型来实现。`skip-gram`的训练目标是学习词向量表示，这个表示可以很好的预测它在相同句子中的上下文。数学上，给定训练词 $w_1,w_2,…,w_T$，`skip-gram`模型的目标是最大化下面的平均对数似然。

$\frac{1}{T}\sum_{t=1}^{T}\sum_{j=-k}^{j=k}\log p(w_{t+j}|w_t)$

其中`k`是训练窗口的大小。在`skip-gram`模型中，每个词$w$和两个向量 $u_w$和 $v_w$ 相关联，这两个向量分别表示词和上下文。正确地预测给定词 $w_j$ 的条件下 $w_i$ 的概率使用`softmax`模型。

$p(w_i|w_j)=\frac{exp(u_{w_i}^{T}v_{w_j})}{\sum_{l=1}^{T}v_{w_j}}$

其中 $V$表示词汇数量。在`skip-gram`模型中使用`softmax`是非常昂贵的，因为计算 $\log p(w_i|w_j)$与$V$是成比例的。为了加快`Word2Vec`的训练速度，`MLlib`使用了分层`softmax`,这样可以将计算的复杂度降低为`O(log(V))`。

## 3 实例

下面的例子展示了怎样加载文本数据、切分数据、构造`Word2Vec`实例、训练模型。最后，我们打印某个词的40个同义词。

```scala
import org.apache.spark._
import org.apache.spark.rdd._
import org.apache.spark.SparkContext._
import org.apache.spark.mllib.feature.{Word2Vec, Word2VecModel}
val input = sc.textFile("text8").map(line => line.split(" ").toSeq)
val word2vec = new Word2Vec()
val model = word2vec.fit(input)
val synonyms = model.findSynonyms("china", 40)
for((synonym, cosineSimilarity) <- synonyms) {
  println(s"$synonym $cosineSimilarity")
}
```

## 3 源码分析

由于涉及神经网络相关的知识，这里先不作分析，后续会补上。要更详细了解`Word2Vector`可以阅读文献【2】。

# 参考文献

【1】[哈夫曼树与哈夫曼编码](http://www.cnblogs.com/Jezze/archive/2011/12/23/2299884.html)

【2】[Deep Learning 实战之 word2vec](docs/word2vec.pdf)

【3】[Word2Vector谷歌实现](https://code.google.com/p/word2vec/)

【4】[[ml] word2vec模型和源码解析](http://cherishzhang.github.io/post/ml/word2vec/)

【5】[word2vec 中的数学原理详解](http://www.cnblogs.com/peghoty/p/3857839.html)