---
title: RAG开发的痛点以及解决方案
date: 2024-04-10 11:43:46
tags:
  - 大模型
  - RAG
categories: [大模型,RAG]
description: RAG笔记2 —— RAG面临的问题及解决方案
cover: https://s3.bmp.ovh/imgs/2024/04/08/2368e73f30658fe8.png
---

{% hideToggle 参考,bg,color %}
- [12 RAG Pain Points and Proposed Solutions](https://towardsdatascience.com/12-rag-pain-points-and-proposed-solutions-43709939a28c)
- https://baoyu.io/translations/rag/12-rag-pain-points-and-proposed-solutions
{% endhideToggle %}



# 引言
{% asset_img p1.webp p1 %}

图源自 Barnett 等人的研究 [工程化检索增强生成系统时的七大挑战](https://baoyu.io/translations/ai-paper/2401.05856v1-seven-failure-points-when-engineering-a-retrieval-augmented-generation-system)。后面会介绍上述7大挑战，并且补充5个额外的挑战，以及它们的解决方案：

- 痛点 1：缺失内容
- 痛点 2：关键文档被遗漏
- 痛点 3：文档整合的长度限制 —— 超出上下文
- 痛点 4：提取困难
- 痛点 5：格式错误
- 痛点 6：缺乏具体细节
- 痛点 7：回答不全面
- 痛点 8：数据摄入的可扩展性问题
- 痛点 9：结构化数据的问答
- 痛点 10：从复杂 PDF 文档提取数据
- 痛点 11：备用模型策略
- 痛点 12：大语言模型的安全性

# 痛点1：缺失内容

**具体描述**：当提出的问题无法从可用文档中找到答案时，RAG系统可能会回答"对不起，我不知道”，但如果问题与内容相关却没有答案，系统可能会被误导提供错误的回答。

**解决方案**：
- 数据清洗，构建高质量的数据

- prompt优化
    - 例如，通过设置提示：“如果你对答案不确定，就告诉我你不知道”，可以鼓励模型承认其局限性，并更透明地表达不确定性。虽然无法保证百分百的准确率，但在数据清洗之后，设计恰当的提示是提高回答质量的有效手段之一。

# 痛点2：关键文档被遗漏

**具体描述**：问题的答案在文档中，但排名没有足够高以返回给用户。（理论上所有文档都会被排名并用于下一步，但实际上只返回排名最高的K个文档）。

**解决方案**：
- 通过调整`chunk_size`和`similarity_top_k`两个参数优化检索效果
    - [通过 LlamaIndex 实现超参数自动调整中](https://medium.com/gitconnected/automating-hyperparameter-tuning-with-llamaindex-72fdd68e3b90?sk=0a29f2025b965040b9b1defd9525e4eb)
    - [对 RAG 进行超参数优化](https://docs.llamaindex.ai/en/stable/examples/param_optimizer/param_optimizer.html)

- 检索结果的优化排序（Rerank算法）
    - [优化排序的前后差异](https://docs.llamaindex.ai/en/stable/examples/node_postprocessor/CohereRerank.html)
    - 另外，通过各种嵌入技术和排序器，可以对检索性能进行评估和提升，详见[提升 RAG 性能：挑选最优的嵌入技术和排序模型](https://blog.llamaindex.ai/boosting-rag-picking-the-best-embedding-reranker-models-42d079022e83)，由 Ravi Theja 撰写。
    - 定制化的排序器经过微调后能够实现更优的检索性能，具体实施方法请参阅[通过微调 Cohere 排序器与 LlamaIndex 提升检索效果](https://blog.llamaindex.ai/improving-retrieval-performance-by-fine-tuning-cohere-reranker-with-llamaindex-16c0c1f9b33b)

# 痛点3：文档整合的长度限制一超出上下文

**具体描述**：数据库检索到包含答案的文档，但这些文档没有被包含在生成答案的上下文中。当数据库返回许多文档时，需要进行整合处理以检索答案。

**解决方案**：
- 调整检索策略：比如：基础检索、知识图谱检索等

- 微调嵌入技术：如果使用的是开源嵌入模型，对其进行微调是提高检索准确度的有效手段。[嵌入模型微调指南](https://docs.llamaindex.ai/en/stable/examples/finetuning/embeddings/finetune_embedding.html)

# 痛点4：提取困难

**具体描述**：答案存在于上下文中，但大型语言模型未能提取出正确的答案。这通常发生在上下文中存在太多噪声或矛盾信息时。

**解决方案**：
- 数据清洗

- 提示信息压缩，可以参考：[LongLLMLingua.pdf](https://rzv2chzpkh.feishu.cn/wiki/FIMJwPPyGiKJmAkuhJGcBrAonKe#CZQ6dQ58HotZswxkNTrcqZiwnGf)

- LongContextReorder（长内容优先排序）：[研究](https://arxiv.org/abs/2307.03172)发现，当关键数据被放置在输入内容的开始或结尾时，往往能够获得最佳的性能表现。为了解决信息在输入中间部分“迷失”的问题，LongContextReorder 应运而生，它通过重新排序检索到的节点来优化处理，特别适用于需要处理大量顶级结果的情形。

# 痛点5：格式错误

**具体描述**：问题涉及以特定格式（如表格或列表）提取信息，但大型语言模型忽略了指令。

**解决方案**：
- prompt优化

- 输出解析：可以在以下方面帮助确保获得期望的输出：为任何提示/查询提供格式化指令；对大语言模型的输出进行“解析”
    - [Pydantic程序](https://docs.llamaindex.ai/en/stable/module_guides/querying/structured_outputs/pydantic_program.html)

```python
from pydantic import BaseModel
from typing import List

from llama_index.program import OpenAIPydanticProgram

# Define output schema (without docstring)
class Song(BaseModel):
    title: str
    length_seconds: int


class Album(BaseModel):
    name: str
    artist: str
    songs: List[Song]

# Define openai pydantic program
prompt_template_str = """\
Generate an example album, with an artist and a list of songs. \
Using the movie {movie_name} as inspiration.\
"""
program = OpenAIPydanticProgram.from_defaults(
    output_cls=Album, prompt_template_str=prompt_template_str, verbose=True
)

# Run program to get structured output
output = program(
    movie_name="The Shining", description="Data model for an album."
)
```
# 痛点6：缺乏具体细节

**具体描述**：回答在响应中返回，但不够具体或过于具体，无法满足用户的需求。当RAG系统设计者对给定问题有预期结果时（例如针对学生的教学内容），就会发生这种情况。

**解决方案**：
- 进阶检索技巧：从小到大的信息聚合检索、递归式检索方法
    - 参考文章：[如何通过高级检索 LlamaPacks 优化您的 RAG 流程，并利用 Lighthouz AI 进行性能基准测试](https://towardsdatascience.com/jump-start-your-rag-pipelines-with%E9%AB%98%E7%BA%A7%E6%A3%80%E7%B4%A2-llamapacks-%E5%92%8C-lighthouz-ai-80a09b7c7d9d?sk=14e50a68f9ef825aaa6634365c7d9617)

# 痛点7：回答不全面

**具体描述**：回答不完整并不是错误的，但遗漏了一些信息，尽管这些信息在上下文中可用并且可以提取。例如，对于"这些文件A、B和C的关键点是什么？"这样的问题，更好的方法是分别询问这些问题。

**解决方案**：
- 查询变换的技巧：一个有效提升凡AG处理能力的策略是增设一个查询理解层，即在实际检索知识库之前进行一系列的查询变换。具体来说，我们有以下四种变换方式：
    - 路由：在不改变原始查询的基础上，识别并定向到相关的工具子集，并将这些工具确定为处理该查询的首选。
    - 查询改写：在保留选定工具的同时，通过多种方式重构查询语句，以便跨相同的工具集进行应用。
    - 细分问题：将原查询拆解为若干个更小的问题，每个问题都针对特定的工具进行定向，这些工具是根据它们的元数据来选定的。
    - ReAct 代理工具选择：根据原始查询判断最适用的工具，并为在该工具上运行而特别构造的查询。
    
- 参考文章：[提升 RAG 性能的高级查询变换技巧](https://towardsdatascience.com/advanced-query-transformations-to-improve-rag-11adca9b19d1)

# 痛点8：数据摄入的可扩展性问题

**具体描述**：在RAG系统中，数据摄入的可扩展性问题指的是系统在有效管理和处理大规模数据时面临的挑战，这可能导致性能瓶颈甚至系统故障。这类问题可能会造成数据摄入时间过长、系统过载、数据质量下降以及可用性受限等现象。

**解决方案**：
- 加速文档处理：并行摄取技术，参考：LlamaIndex 提供的[详细教程](https://github.com/run-llama/llama_index/blob/main/docs/examples/ingestion/parallel_execution_ingestion_pipeline.ipynb?__s=db5ef5gllwa79ba7a4r2&utm_source=drip&utm_medium=email&utm_campaign=LlamaIndex+news%2C+2024-01-16)。

# 痛点9：结构化数据的问答

**具体描述**：解读用户的查询并准确检索到所需的结构化数据是一项挑战，尤其是当面对复杂或模糊的查询请求时。这一挑战由于文本到SQL的转换不够灵活，以及当前大语言模型在处理这些任务上的局限性而更加复杂。

**解决方案**：
- 链式思维表格包：适合处理包含多重信息的复杂表格单元问题，通过逐步精确地切割和解析数据，直至找到所需的数据子集，显著提高了对表格数据的查询效率，参考：[链式表格原论文](https://arxiv.org/abs/2401.04398)

- 混合自洽（多数投票法）查询引擎包：原文见[重新思考大语言模型如何理解表格数据](https://arxiv.org/pdf/2312.16702v1.pdf)

# 痛点10：从复杂PDF中提取数据

**具体描述**：在从复杂PDF文档，如嵌入表格的文档中提取数据进行问答时，传统的检索方法往往无法达到目的。需要一个更高效的方法来处理这种复杂的PDF数据提取需求。

**解决方案**：
- 嵌入式表格检索技术：LlamaIndex 提出了一种解决方案——`EmbeddedTablesUnstructuredRetrieverPack

# 痛点11：备用模型策略

**具体描述**：在使用大语言模型过程中，可能会担心模型可能会遇到问题，比如遇到OpenAI模型的访问频率限制错误。这时候就需要一个或多个备用模型作为后备，以防主模型出现故障。

**解决方案**：
- Neutrino Router：Neutrino路由器能够通过一个预测模型，智能地选择最适合输入问题的大语言模型。

- OpenRouter：OpenRouter提供了一个一站式的API接口，能够接入任何大语言模型。不仅能找到市场上任一模型的最低价格，还能在首选的服务提供商遇到问题时，自动切换到其他选项。

# 痛点12：LLM的安全性：

**具体描述**：在设计和开发 AI 系统时，如何有效防止恶意输入、确保输出安全、保护敏感信息不被泄露等问题，都是每位 AI 设计师和开发者需要面对的重要挑战。

**解决方案**：
- Llama Guard：保护大语言模型的新工具，借鉴7-B Llama 2的技术，Llama Guard旨在通过评估输入（例如分析提问的内容）和输出（即对回答的分类）来帮助大语言模型 (LLMs) 鉴别内容是否安全。它本身也使用了大语言模型技术，能够判断某个问题或回答是否安全，若发现不安全的内容，还会详细列出违反的具体规则。