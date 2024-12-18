---
title: OpenAI官方的Prompt工程指南
tags:
  - 大模型
  - Prompt Engineering
categories:
  - 大模型
  - Prompt
description: Prompt相关笔记记录
cover: 'https://s3.bmp.ovh/imgs/2024/04/08/2368e73f30658fe8.png'
date: 2024-11-26 15:44:30
---

> 原文：https://platform.openai.com/docs/guides/prompt-engineering
> 参考：https://www.jiqizhixin.com/articles/2023-12-18-17

**共六个策略**

# 策略一：写清楚指令

## 包含更详细的查询信息
提示中**尽量包含更详细的查询信息（要求）**，从而获得更相关的答案。如下图，同样是总结会议记录，采用这样的提示「用一个段落总结会议记录。然后写下演讲者的 Markdown 列表以及每个要点。最后，列出发言人建议的后续步骤或行动项目（如果有）。」结果会比较好。

<!-- {% asset_img p1.png p1 %} -->
<img src="1.png"  style="zoom:60%;" />

## 提供示例
用户可以**提供示例**。例如，当你想让模型模仿一种难以明确描述的回答风格时，用户可以提供少数示例。

## 指定步骤
第三点是**指定模型完成任务时所需的步骤**。对于有些任务，最好指定步骤如步骤 1、2，显式地写出这些步骤可以使模型更容易地遵循用户意愿。

## 指定输出长度
第四点是**指定模型输出的长度**（自己的经验：规定几句比规定多少字，效果更好）。用户可以要求模型生成给定目标长度的输出，目标输出长度可以根据单词、句子、段落等来指定。

## 使用分隔符划分各部分
第五点是**使用分隔符来明确划分提示的不同部分**（自己的经验：**一般使用`****`来表示强调，`"""`或者`###`来分段**）。`"""`、`XML 标签`、`小节标题`等分隔符可以帮助划分要区别对待的文本部分。

## 角色扮演
第六点是**让模型扮演不同的角色**，以控制其生成的内容。比如说：你是一位Python专家，你是一个专业的审稿人等。

# 策略二：提供参考文本

语言模型会时不时的产生幻觉，自己发明答案，为这些模型提供参考文本可以帮助减少错误输出。需要做到两点：

## 提供参考回答
首先是指示模型**使用参考文本回答问题（其实就是RAG）**。如果我们可以为模型提供与当前查询相关的可信信息，那么我们可以指示模型使用提供的信息来组成其答案。比如：使用由三重引号引起来的文本来回答问题。如果在文章中找不到答案，就写「我找不到答案」。

## 从参考文本中引用答案
其次是指示模型**从参考文本中引用答案**。如果输入信息中已经包含了相关知识，就可以直接要求模型在回答问题时引用所提供的文件中的段落。值得注意的是，输出中的引用可以通过在所提供的文件中匹配字符串来进行验证。(如下表)

| **Role**|**Prompt**|
|------------|--------------|
|SYSTEM| You will be provided with a document delimited by triple quotes and a question. Your task is to answer the question using only the provided document and to cite the passage(s) of the document used to answer the question. If the document does not contain the information needed to answer this question then simply write: "Insufficient information." If an answer to the question is provided, it must be annotated with a citation. Use the following format for to cite relevant passages ({"citation": ...}). |
|USER| `"""<insert document here>"""` <br> **Question**: `<insert question here>`|

# 策略三：将复杂的任务拆分为更简单的子任务

正如软件工程中将复杂系统分解为一组**模块化**组件一样，提交给语言模型的任务也是如此。复杂的任务往往比简单的任务具有更高的错误率，此外，复杂的任务通常可以被重新定义为更简单任务的工作流程。包括三点：

## 意图分类
使用**意图分类**来识别与用户查询最相关的指令。

## 总结或过滤长信息
对于需要很**长对话**的对话应用，**总结或过滤**以前的对话。

## 分段递归总结长文档

即：**分段总结长文档**并**递归的构建完整摘要**。由于模型具有固定的上下文长度，因此要总结一个很长的文档（例如一本书），我们可以使用**一系列查询来总结文档的每个部分**。章节摘要可以连接起来并进行总结，**生成摘要的摘要**。这个过程可以递归地进行，直到总结整个文档。如果有必要使用前面部分的信息来理解后面的部分，那么另一个有用的技巧是在文本（如书）中任何给定点之前包含文本的运行摘要，同时在该点总结内容。OpenAI 在之前的研究中已经使用 GPT-3 的变体研究了这种过程的有效性。

# 策略三：给模型时间去思考（慢思考）

对于人类来说，要求给出 17 X 28 的结果，你不会立马给出答案，但随着时间的推移仍然可以算出来。同样，如果模型立即回答而不是花时间找出答案，可能会犯更多的推理错误。在给出答案之前采用思维链可以帮助模型更可靠地推理出正确答案。需要做到三点：

## 
首先是指示模型在急于**得出结论之前找出自己的解决方案**。

其次是使用**内心独白（inner monologue）**或**连续提问**来**隐藏模型的推理过程**。
前面的策略表明，模型有时在回答特定问题之前详细推理问题很重要。对于某些应用程序，模型用于得出最终答案的推理过程不适合与用户共享。例如，在辅导应用程序中，我们可能希望鼓励学生得出自己的答案，但模型关于学生解决方案的推理过程可能会向学生揭示答案。

>**inner monologue** 是一种可以用来缓解这种情况的策略。inner monologue 的思路是指示模型将原本对用户隐藏的部分输出放入结构化格式中，以便于解析它们。然后，在向用户呈现输出之前，将解析输出并且仅使部分输出可见。

| **Role** | **Prompt** |
|----------|------------|
|SYSTEM| 按以下步骤回答用户问题：<br>第 1 步 - 首先独立解决问题。不要依赖学生的答案，因为可能有误。将此步骤的所有内容用三重引号 (""") 包围。<br>第 2 步 - 将你的解答与学生的答案比较，判断学生的答案是否正确。将此步骤的所有内容用三重引号 (""") 包围。<br>第 3 步 - 如果学生答案有误，想出一个不直接透露答案的提示。将此步骤的所有内容用三重引号 (""") 包围。<br>第 4 步 - 如果学生答案有误，给出第 3 步的提示（不用三重引号）。用“提示：”替代“第 4 步 - ...”。 |
|USER| 问题陈述：\<插入问题陈述> <br> 学生解答：<插入学生解答>|


最后是**询问模型在之前的过程中是否遗漏了任何内容**（灵魂拷问，真有论文写过😂）。

# 策略五：使用外部工具（工具调用）

通过向模型提供其他工具的输出来弥补模型的弱点。例如，文本检索系统（有时称为 RAG 或检索增强生成）可以告诉模型相关文档。OpenAI 的 Code Interpreter 可以帮助模型进行数学运算并运行代码。如果一项任务可以通过工具而不是语言模型更可靠或更有效地完成，或许可以考虑利用两者。包括以下：

- 使用基于嵌入的搜索实现高效的知识检索；（**RAG**）
- 调用外部 API；（**Agent中工具调用**，比如调用代码解释器）
- 赋予模型访问特定函数的权限（调用函数）。

# 策略六：系统的测试变化

有时，很难确定某个更改，比如新的指令或设计，是否真的改善了系统。观察几个案例可能会有所帮助，但在样本量较小的情况下，很难判断这是真正的改进还是偶然的幸运。可能某些更改在特定输入上提高了性能，但在其他情况下则降低了性能。

评估程序对于优化系统设计非常有用。有效的评估特点是：
- 能够代表现实世界中的使用情况（或至少具有多样性）
- 包含众多测试案例，从而拥有更强的统计能力
- 可以轻松自动化或重复

评估可以由计算机、人工或两者结合进行。计算机可以自动化那些具有客观标准的评估（例如，有唯一正确答案的问题），也可以用于某些主观或模糊标准的评估，在这种情况下，模型输出由其他模型查询进行评估。**OpenAI Evals** 是一个开源软件框架，提供创建自动化评估的工具。