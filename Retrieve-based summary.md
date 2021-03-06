refer:<br>[智能客服FAQ问答任务的技术选型探讨](https://zhuanlan.zhihu.com/p/50799128)<br>[问题对语义相似度计算-参赛总结](http://www.zhuzongkui.top/2018/08/10/competition-summary/)

#### key phrases

- architecture, difficulty

#### System Architecture

+ 检索（粗排）

  关键词匹配，主要保证准确率

  ...

+ 匹配（粗排）

  字面匹配（lexical match），关注准确率

  语义匹配（semantic match），关注召回率（Q-Q pairs/Q-A pairs 语义匹配的精度不高，所以也只适用于召回场景）

+ 排序（learning to rank）（精排）



[AnyQ](https://github.com/baidu/AnyQ)

![AnyQ-Framework](https://github.com/baidu/AnyQ/raw/master/docs/images/AnyQ-Framework.png)

[System Architecture for Single-turn](Deep Chit-Chat: Deep Learning for ChatBots)

![architecture](https://github.com/bifeng/dialogue-system/raw/master/image/system_architecture_single_turn.png)

[System Architecture for Multi-turn](Deep Chit-Chat: Deep Learning for ChatBots)

![architecture](https://github.com/bifeng/dialogue-system/raw/master/image/system_architecture_multi_turn.png)

#### Labeled Data (数据标注)

根据业务场景，选择合适的建模方式及数据标注方案。

Q-Q pairs建模的应用场景及优点：

1. 支持业务需求变更的任务型chatbot

   Q-Q paris建模，相当于将问题与答案的解耦，只需要关注于问题，当业务发生变更时，只需更新相关问题的答案，无需重新训练模型！

2. Question Understanding/Representation

   Q-Q paris建模，相当于一个NLU引擎，对于问题都能获得一个很好的表示，有利于语料更新及模型迭代。如通过语义聚类，发现新问题与重复问题去重。

3. 迁移学习

   预训练一个开放域的Q-Q pars 语义匹配模型，再在垂直域进行fine-tuning！

Q-A pairs建模的应用场景及优点：

1. 闲聊型chatbot
2. 

Q-Q paris构建方法及建模的注意事项：

1. 构建方法

   方法一，前期用规则方法，待数据积累到一定量级，从日志中抽取用户的输入，然后粗略用一些规则和聚类算法，然后把每个类的对应question列表，两两组合去生成样本，然后人工来标注

   方法二，Paraphrase Generation

   方法三，人工编写相似问法

   方法一 避免脑补的相似问法；方法二 如何保证相似问法的多样性，而不仅仅是关键词的排列组合？方法三 不是真实的相似问法，且各标注人员的业务理解存在差异

   **难点**：

   如何构建合适的正样本？

2. 数据增强

   假设 Q1 在所有样本里出现2次，分别是

   label, question, question

   1，Q1，Q2
   1，Q3，Q1
   模型无法正确学习Q1与Q2/Q3的相同，而是会认为只要input里有Q1即为正样本。
   需要通过数据处理引导模型进行“比较”，而不是“拟合”。
   解决方案：通过构建一部分补充集（负例），对冲所有不平衡的问题。

3. 后处理

   - 传递关系

     相似：（AB=1，AC=1）—> BC=1

     不相似：（AB=1，AC=0）—> BC=0

   - infer机制

     除了判断test集的每个样本得分以外，还会通过已知同义问题集的其他样本比对进行加权；融合时轻微降低得分过高的模型权重，补偿正样本过多的影响；将已知确认的样本修正为0/1。

Q-A paris构建方法及建模的注意事项：

1. 构建方法

   领域相关，领域内随机选择负样本；主题相关，根据主题索引召回相关response作为负样本；语义相关，根据语义索引召回相关response作为负样本。

   **难点**：

   如何选择合适的负样本？

2. 

#### Evaluation

+ method 1

  The test data contains 1000 dialogue context, and for each context we create 10 responses as candidates. We recruited three labelers to judge if a candidate is a proper response to the session. A proper response means the response can naturally reply to the message given the context. Each pair received three labels and the majority of the labels was taken as the final decision.

  https://github.com/MarkWuNLP/MultiTurnResponseSelection



#### Difficulty

##### entity association

Solution:

1. 根据建模场景

   Q-Q pairs建模 - Question含有实体 - 预处理，固定实体、无关实体部分交给排序

   Q-A pairs建模 - 

   1）Attention Pooling

   2）Answer Expansion

   ​     借助知识图谱对answer进行扩展，获得更多关于answer的信息。

   3）Answer含有实体 - 后处理，通过规则关联实体

2. 这类问题在Question Answer场景中经常出现，可以借鉴其方法。

##### logic/topic consistency

Solution:

1. 根据建模场景

   Q-Q pairs建模 - 不存在这个问题

   Q-A pairs建模 - 如何解决？

Neural Discourse Modeling [paper](http://www.cs.brandeis.edu/~tet/papers/thesis_rutherford_final.pdf)

Sentence Ordering and Coherence Modeling using Recurrent Neural Networks  [paper](https://yale-lily.github.io/public/aaai2018_CR.pdf)

Modeling Topical Coherence in Discourse without Supervision [paper](https://export.arxiv.org/pdf/1809.00410)

##### threshold control

+ 排序模型的概率得分
+ 生成模型的概率得分

##### long tail problem



#### QA

+ how to filter the unrelated response for the ranking?

  define a threshold at the match stage ?

+ how to make the sentence similarity score (word2vec/bert, etc.) be more distinction between each other?

  define a threshold at the match stage ?

+ The cosine similarity of two sentence vectors is unreasonably high (e.g. always > 0.8), what's wrong?

  (This question is from [bert as service](https://github.com/hanxiao/bert-as-service).)

  A decent representation for a downstream task doesn't mean that it will be meaningful in terms of cosine distance. Since cosine distance is <u>a linear space</u> where all dimensions are weighted equally. if you want to use cosine distance anyway, then please focus on the rank not the absolute value. Namely, do not use:

```
if cosine(A, B) > 0.9, then A and B are similar
```

​	Please consider the following instead:

```
if cosine(A, B) > cosine(A, C), then A is more similar to B than C.
```



