---
title: 人工神经网络
date: 2022-10-03 10:26:08
tags: 
  - 人工智能
categories:
  - 高级人工智能学习笔记
description: 国科大《高级人工智能》笔记2 —— 人工神经网络
cover: https://s3.bmp.ovh/imgs/2022/09/03/f51220518329aecb.jpg
---
# 人工神经网络ANN
## 结构
{% asset_img p1.png p1 %}
## 学习规则
- 能量最小（ENERGY MINIMIZATION）
- 对人工神经网络，需要确定合适的能量定义；可以使用数学上的优化技术来发现如何改变神经元间的联接权重。
# 多层感知机
多层感知机是ANN的一种。指具有至少三层节点，输入层，一些中间层和输出层的神经网络。给定层中的每个节点都连接到相邻层中的每个节点。输入层接收数据，中间层计算数据，输出层输出结果。
{% asset_img p2.png p2 %}
## 特点
多层感知机特性：
- 多层感知机层间神经元全连接；
- Can represent AND, OR, NOT, etc., but not XOR；
- 若训练数据集是线性可分的，则感知机模型收敛。

## 感知机存在的问题
- 噪声（线性不可分）
- 泛化性
### 单层感知机不可解决异或问题
{% asset_img p4.png p4 %}

> 传送门[https://blog.csdn.net/buracag_mc/article/details/89254808?utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-2.nonecase]
# 权重学习算法：BP算法
## 介绍
BP算法全称叫作误差反向传播(error Back Propagation，或者也叫作误差逆传播)算法。其算法基本思想为：将输出误差以某种形式反传给各层所有的单元，各层按本层误差修
正各单元连接权值。【有监督学习，采用梯度下降法调参】

### 特点
{% asset_img p3.png p3 %}
## BP遇到的困难，为什么会出现梯度消失
### 困难
- 梯度消失，梯度爆炸
- 局部极小
- 只能用于标注数据
### why梯度消失
因为BP算法采用链式法则，从后层向前层传递信息时，若每层神经元对上一层神经元偏导乘以w均小于1，多次链式法则,多级导数权值相乘结果会越来越小，导致loss传递到越前方越小。
