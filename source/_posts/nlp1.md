---
title: NLP基础
date: 2021-02-12 20:21:32
tags: 
- 自然语言处理
- python
categories: 
- 自然语言处理
description: 自然语言处理学习笔记1——基础知识
---

> 参考书籍：
> - 《自然语言处理理论与实战》
> - 《Python 自然语言处理》
> - 《Python 自然语言处理实战：核心技术与算法》


## NLP基本概念
### 什么是NLP
NLP定义：自然语言处理，用计算机处理、理解以及运行的人类语言（如中文、英文等）

NLP分类：
- 自然语言理解
  - 音系学：语法中发讯的系统化组织
  - 词态学：研究单词构成及相互关系
  - 句法学：给定文本哪部分是语法正确的
  - 语义句法学：给定文本的含义是什么？
  - 语法学：文本的目的是什么？
- 自然语言处理
  - 自然语言文本

NLP的研究任务
- 机器翻译：计算机具备将一种语言**翻译成另一种语言**的能力
- 情感分析：计算机能够判断用户评论**是否积极**
- 智能问答：计算机能够正确**回答输入的问题**
- 文摘生成：计算机能够准确**归纳、总结并产生文本摘要**
- 文本分类：计算机能够采集各种文章，进行**主题分析**，从而进行**自动分类**
- 舆论分析：计算机能够**判断目前舆论的导向**
- 知识图谱：知识点相互连接而成的**语义网络**


## NLP常用术语
- 分词（segment）
  - 将句子分成独立的有意义的语言成分（词）
- 词性标注（part-of-speech tagging）
  - 对词的词性进行标注（例：动词、名词、形容词）
- 命名实体识别（NER, Named Entity Recognition）
  - 从文本中识别具有特定类别的实体（例：人名、地名、机构名等）
- 句法分析（syntax parsing）
  - 分析句子各个成分的依赖关系（例：主从关系），最后可生成句法分析树
- 指代消解（anaphora resolution）
  - 代词指代
- 情感识别（emotion recognition）
  - 本质是分类问题，经常应用在舆情分析等领域，可分为两类（正面、负面）或者三类（+中性）
- 纠错（correction）
  - 在搜索技术和输入法中利用很多，具体做法可以根据N-Gram、字典树等等方法
- 问答系统（QA system）
  - 类似机器人的人工智能系统


## 语料库
- 中文维基百科
- 搜狗新闻语料库
- IMDB情感分析语料库
- 等

## 三个层次
- 词法分析
- 句法分析
- 语义分析
