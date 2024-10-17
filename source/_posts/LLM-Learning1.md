---
title: RAG 应知必会
date: 2024-04-08 20:58:56
tags: 
  - 大模型
  - RAG
categories: [大模型,RAG]
description: RAG笔记1 —— 基础
cover: https://s3.bmp.ovh/imgs/2024/04/08/2368e73f30658fe8.png
---

> 参考：[Retrieval-Augmented Generation for Large Language Models: A Survey](https://arxiv.org/abs/2312.10997)

# RAG的发展

RAG技术的发展可以从技术范式的角度分为几个阶段，主要包括：
- **朴素RAG(Naive RAG)**:这是RAG技术的基出形式，包括三个基本步骤：索引(将文档库分割成较短的Chunk并通过编码器构建向量索引)、检索(根据问题和chunks的相似度检索相关文档片段)、生成(以检索到的上下文为条件，生成问题的回答)。
- **进阶的RAG(Advanced RAG)**:为了解决Naive RAG在检索质量、响应生成质量以及增强过程中的挑战，Advanced RAG:范式被提出。它在数据索引、检索前和检索后都进行了额外处理，如更精细的数据清洗、设计文档结构和添加元数据等，以提升文本的
一致性、准确性和检索效率。
- **模块化RAG(Modular RAG)**:随着RAG技术的进一步发展，模块化RAG概念被提出。它在结构上更加自由和灵活，引入了更多的具体功能模块，例如查询搜索引擎、融合多个回答。技术上将检索与微调分开，提供了更高的自定义性和灵活性。

三个阶段之间是继承与发展的关系：
{% asset_img p1.png p1 %}

# 如何评价RAG

> 🚩 <big>__评估指标__</big>：
> 1. 上下文相关性一一检索的上下文是否相关
> 2. 上下文回溯一评估检索到的上下文是否包含了问题的所有必要信息
> 3. 忠实度一一生成的答案是否与检索的内容一致，是否保留了原始信息
> 4. 答案相关性一一评估生成的答案是否与问题相关，以及是否提供了有用的信息

## 评估角度
### RAG三元组
- **Context Relevance**：衡量召回的Context能够支持Query的程度。如果该得分低，反应出了召回了太多与Qury问题无关的内容，这些错误的召回知识会对LLM的最终回答造成一定影响。
- **Groundedness**：衡量LLM的Response遵从召回的Context的程度。如果该得分低，反应出了LLM的回答不遵从召回的知识，那么回答出现幻觉的可能就越大。
- **Answer Relevance**：衡量最终的Response回答对Query提问的相关度。如果该得分低，反应出了可能答不对题。
{% asset_img p2.png p2 %}

### 基于Ground-truth的指标
- Ground-truth是回答，那就可以<u>直接比较RAG应用的回答和ground-truth之间的相关性</u>，来端到端地进行衡量。
- 让LLM生成评估测试集，来生成query和ground-truth,比如，在ragas的 [Synthetic Test Data generation](https://docs.ragas.io/en/latest/concepts/testset_generation.html) 和 [llama-index QuestionGeneration](https://docs.llamaindex.ai/en/stable/examples/evaluation/QuestionGeneration.html) 中都有一些集成好的方法，可以直接方便地使用。
- Ground-truth是知识文档中的chunks，对比ground-truth doc chunks和召回的contexts之间的相关性。

### 基于LLM的定量评估
- 有了GPT-4后，其可行性就提高了。我们只需设计好prompt,将要打分的一些文字放入prompt,访问GPT-4,就可以得到一个想要的得分结果。

## 评估工具

1. RAGAs:https://docs.ragas.io/en/latest/concepts/metrics/context_recall.html
2. TruLens:https://www.trulens.org/trulens_eval/install/
3. Llama-Index:https://docs.llamaindex.ai/en/stable/optimizing/evaluation/evaluation.html
4. Phoenix:https://docs.arize.com/phoenix/
5. DeepEval:https://github.com/confident-ai/deepeval
6. LangSmith:https://docs.smith.langchain.com/evaluation/evaluator-implementations
7. OpenAI Evals:https://github.com/openai/evals

{% asset_img p3.png p3 %}

# RAG未来发展方向
## RAG的垂直优化
- 长下文长度
  - 检索内容过多，超过窗口限制怎么办？
  - 如果LLMs的上下文窗口不再受限制，RAG应该如何改进？
- 鲁棒性
  - 检索到错误内容怎么处理？
  - 怎么对检索出来内容进行过滤和验证？
  - 怎么提高模型抗毒、抗噪声的能力。
- 与微调协同
  - 如何同时发挥RAG和FT的效果，两者怎么协同，怎么组织，是串行、交替还是端到端？
- Scaling-Law
  - RAG模型是否满足Scaling Law?
  - RAG是否会，或是在什么场景下会出现Inverse Scaling Law的现象？
- LLM的角色
  - LLMs可以用于检索(用LLMs的生成代替检索或检索LLMs记忆)、用于生成、用于评估。如何进一步挖掘LLMs在RAG中的潜力？

## 工程实践
- 如何降低超大规模语料的检索时延？
- 如何保证检索出来内容不被大模型泄露？

## RAG的多模态的拓展
- 如何将RAG不断发展的技术和思想拓展到图片、音频、视频或代码等其他模态的数据中？
- 一方面可以增强单一模态的任务，另一方面可以通过RAG的思想将多模态进行融合。

## RAG的生态

- RAG的应用已经不仅仅局限于问答系统，其影响力正在扩展到更多领域。现在，推荐系统、信息抽取和报告生成等多种任务都开始受益于RAG技术的应用。
- 与此同时，RAG技术栈也在井喷。除了已知的Langchain和LlamaIndex等工具，市场上涌现出更多针对性的RAG工具，例如：用途定制化，满足更加聚焦场景的需求；使用简易化，进一步降低上手门槛的；功能专业化，逐渐面向生产环境。

