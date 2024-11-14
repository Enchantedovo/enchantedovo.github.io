---
title: LLM for Security 论文记录 1114
tags:
  - 大模型
  - 网络安全
  - 大模型安全
  - 生成式人工智能
categories:
  - [大模型,大模型&安全]
  - [大模型,论文阅读]
  - [大模型,Agent]
description: LLM for Security 论文记录 1114
cover: 'https://s3.bmp.ovh/imgs/2024/04/08/2368e73f30658fe8.png'
date: 2024-11-14 10:57:54
---
> 更新一下论文阅读记录，一直写在了本地的编辑器上，现在更新到博客中（持续更新ing，打算以月为单位更新博客了）
# Agent in Security
<!-- {% asset_img p1.png p1 %} -->
<!-- <img src="p1.png"  style="zoom:55%;" /> -->

## PentestGPT
占坑

## AUTOATTACKER

AUTOATTACKER: A Large Language Model Guided System to Implement Automatic Cyber-attacks

> 论文：https://arxiv.org/abs/2403.01038
> 代码：https://github.com/binarybrain-009/AUTOATTACKER （核心代码放的是一个注释，差评）

<img src="autoattack.png"  style="zoom:80%;" />

**研究内容**
利用基于 LLM 的系统来模拟在各种攻击技术和环境下通常为人为操作或“手动键盘”攻击的攻击后阶段。

**研究意义**

- 首先，基于 LLM 的自动化漏洞后利用框架可以**帮助分析师快速测试**并持续改进其组织的网络安全态势，以抵御以前未曾见过的攻击。
- 其次，基于LLM的渗透测试系统可以通过有限数量的分析师来**扩展红队的有效性**。
- 最后，这项研究可以帮助防御系统和团队在实际使用之前学会**先发制人地检测新的攻击行为**。

**研究动机/挑战**
基于问题引入了独特的挑战，包括：
1）**复杂的攻击任务链**：高级攻击可能需要许多子任务，甚至一个失败的子任务会破坏整个链； 
2）**动作空间的高密度可变性**：bash或Metasploit中的命令具有许多参数，其中一些参数与系统信息或文件夹路径密切相关，其中一个拼写错误可能会破坏攻击命令。

**贡献**
1）提出了一种**模块化** Agent 设计，以在不同点利用LLM的不同功能，例如规划、总结和代码生成，即使在生成单个攻击命令时也是如此。通过这种设计，我们可以更好地利用 LLM 来得出精确的答案。 
2）借用**检索增强生成（RAG）**的想法，在生成下一个动作之前用先前攻击动作（称为经验）的知识库来增强LLM，因此成功攻击的机会会增加，因为它们的组合子任务可以重复使用。

**方法**
设计了 4 个模块，即**总结器**、**规划器**、**导航器**和**经验管理器**，以与 LLM 迭代交互。还精心设计了每个模块的**提示**模板，因此LLM的回答具有高度可控性。为了绕过使用策略，我们开发了一种 **LLM 越狱技术**来引出攻击命令。
