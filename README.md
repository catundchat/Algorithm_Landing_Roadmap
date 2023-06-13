# Algorithm_Business

As an algorithm engineer, thinking about how to help company's business get off the ground | 算法工程师如何结合业务将模型算法迅速落地

本文语境：微信小程序——AI爱家

# 目录

- [确定需求](#确定需求)
- [提升算法](#提升算法)
  - [优化输入（Prompt Tuning）](#优化输入Prompt-Tuning)
  - [优化输出（后处理策略）](#优化输出后处理策略)
- [考虑用户体验，用户反馈循环](#考虑用户体验用户反馈循环)
- [使用额外的数据源](#使用额外的数据源)
- [参考文献](#参考文献)


## 确定需求

1. 目标用户：处在单身，恋爱关系，亲密关系中的人。
2. 功能：以聊天机器人的形式帮助用户理解心理学概念，也能帮助用户进行心理评估，并且能对用户提出的心理学和亲密关系问题给出切实有效的回复。
3. 问题解决：帮助用户处理压力，理解情绪并且能够提供心理学亲密关系等知识
4. 交互方式：文字输入
5. 度量成功：使用次数，以及发送信息的条数，使用时间等指标

## 提升算法

使用机器学习，深度学习算法来更好地理解用户的情绪和需求。这可能需要大量的数据和训练，提高应用的准确性和效用。

### 优化输入（Prompt Tuning）

通过输入的 prompt 或 message 信息来为聊天对话设置上下文
```
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "I had a fight with my girlfriend."},
    {"role": "assistant", "content": "I'm sorry to hear that. Do you want to talk about it?"},
    {"role": "user", "content": "Yes, I need some advice."},
]
```
或者为 ChatGPT 指明更具体的角色和任务
```
messages = [
    {"role": "system", "content": "You are an assistant trained in relationship psychology."},
    {"role": "user", "content": "I had a fight with my girlfriend."},
]
```
### 优化输出（后处理策略）

尽管ChatGPT通常能够生成高质量的文本，但有时它可能会产生一些不相关或者不准确的信息，可以使用 NLU(Natural Language Understanding) 等后处理策略优化输出

1. 过滤不相关的输出
- 关键词方法：TF-IDF(term frequency and inverse document frequency) 或 Word2Vec
- 主题建模：将文档表示为主题的集合，更能理解文档的结构和内容进而比较不同文档的相似性,LSA and LDA
- 使用时可以需要一个阈值，当相似度低于这个阈值时，我们可以解释为回复相关性不强

2. Knowledge Graph 检查回答准确性
- 构建知识图谱：一个关于心理学和亲密关系的知识图谱可能包含实体如“压力”，“心理健康”，“恋爱”，“沟通技巧”等，以及它们之间的关系，如"压力影响心理健康"，"良好的沟通技巧有助于恋爱关系"等，确定心理学概念实体与关系，neo4j 构建知识图谱，而后提取关键信息匹配图谱，能够更好的让 C哈

<details>
<summary>知识图谱效果图：</summary>

![知识图谱效果图1](junjie_knowledge_graph.png)

![知识图谱效果图2](kg.png)
</details>

3. 处理冗余和保持回答连贯

- 通过余弦相似度测量句子相似性进而删除高度相似语句

4. 检查回答的情感

- 情感检测：使用在中文语料上微调后的 BERT 模型将 Chatbot 的回答分为共情态，鼓励态，安慰态等（对应junjie提出的三种状态），或者简单分成中立，消极，积极态。在情感检测分类领域，理论上 BERT 模型的效果是要比 GPT 分类准确度更高，因为在训练时 BERT 的训练流程要比 GPT 方法更适合处理情感分类问题
- 情感转换：利用中文情感词典与 Transformers 相关大模型，转换模型回答的情感

## 考虑用户体验，用户反馈循环

确保应用易于使用，回应速度快，并且尽可能提供有用的反馈。此外，考虑用户的隐私和安全问题，确保用户的对话内容得到保护

1. 点赞点踩：利用好点赞点踩功能，目前的想法是在后处理阶段优化回答的长度和详细程度，并且可以 prompt tuning 告诉模型用户喜欢的主题来增多此类信息

2. 构建反馈预测模型：这个模型的任务是，给定一个由ChatGPT生成的回答，预测用户是否会给予正面的反馈（点赞）。这可以看作一个二元分类任务，正面反馈为1，负面反馈为0（或像上述问题一样设定为三分类或多分类任务）训练模型（例如BERT、RoBERTa等），并用反馈数据（点赞点踩信息）对这些模型进行微调。这就得到了一个能预测用户反馈的模型

3. 利用反馈预测模型优化 ChatGPT 的输出：之后生成了一个回答后，可以使用反馈预测模型来预测这个回答是否会得到正面反馈。如果预测结果是负面的，你可以再次调用ChatGPT API，生成一个新的回答，并重复这个过程，直到预测模型预测出用户可能会对回答给予正面反馈

## 使用额外的数据源

1. 构建知识图谱检查AI爱家回答的准确性

2. 预处理和索引数据库：具体包括在本地实现文本清洗，分词，构建索引，将数据输入到全文搜索引擎。Dify使用 OpenAI 的 Embedding 接口或是离线向量引擎来构建知识库，我们也可以将知识库在本地搭建起来，因为上传知识库本身就意味着拖慢速度，在本地跑就是会快一些。

3. 查询数据库，构建辅助模型来整合信息：选择微调后的 BERT 模型来提取关键信息，将查询到的结果整合到输出中，我们可以在 ChatGPT 的回答后添加类似句段诸如 `我们在数据库中还为你找到了以下信息等`

## 参考文献
1.[算法工程师的核心竞争力——落地能力](https://cloud.tencent.com/developer/article/2117248?from=article.detail.1796795&areaSource=106000.1&traceId=UfHfB6AIB2VYRKaltQrJs)
2.[neo4j构建知识图谱](https://www.jianshu.com/p/d4175930e820)
3.[中文情感词典的构建与使用_文本情感识别](https://cloud.tencent.com/developer/article/2119441)
4.[NLP 民工乐园](https://github.com/fighting41love/funNLP)
5.[python中文分词jieba库](https://github.com/fxsjy/jieba)
6.[10分钟完成高精度中文情感分析](https://paddlenlp.readthedocs.io/zh/latest/get_started/quick_start.html)
7.[关于ChatGPT：GPT和BERT的差别（易懂版）](https://zhuanlan.zhihu.com/p/607605399)
