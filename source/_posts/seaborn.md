---
title: seaborn
date: 2021-02-03 11:12:39
tags:
 - python
 - 可视化
categories: 可视化
description: 数据可视化学习(≖ᴗ≖)✧
---

## seaborn简介
`seaborn`同`matplotlib`一样，是Python进行数据可视化分析的重要第三方包。但`seaborn`是在`matplotlib`的基础上进行了**更高级的API封装**。使用`seaborn`，可以在不需要了解那么多底层参数的情况下，同样能够画出比较好看的图表。
## seaborn使用
### 安装&导入
首先确定你的电脑已安装以下应用
- Python 2.7+ or Python 3
- Pandas
- Matplotlib
- Seaborn
- Jupyter Notebook(可选)

```cmd
jupyter notebook
```
出现jupyternotebook的界面，即可进行代码编写：
{% asset_img img1.png img1 %}

导入相关包：
```python
import numpy as np
import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
```
### seaborn绘图风格
首先定义一个简单的方程来绘制一些偏置的正弦波：
```python
def sinplot(flip=1):
    x = np.linspace(0, 14, 100)
    for i in range(1, 7):
        plt.plot(x, np.sin(x + i * .5) * (7 - i) * flip)
```
`matplotlib`默认会绘制如下：
```python
sinplot()
```
{% asset_img img2.png img2 %}
转换为`seaborn`默认绘图，可以简单的用set()方法。
```python
sns.set()
sinplot()
```
{% asset_img img3.png img3 %}
有五种seaborn的风格，它们分别是：
- darkgrid（默认）
- whitegrid
- dark
- white
- ticks


另外可以自己设置相关配置：
```python
sns.set_style("darkgrid", {"axes.facecolor": ".9"})
sinplot()
```
{% asset_img img4.png img4 %}

### 数据可视化demo
对数据的可视化操作，乍看起来很复杂，其实种类总结起来可以分为下面几种：
- 单变量分布可视化(displot)
- 双变量分布可视化(jointplot)
- 数据集中成对双变量分布(pairplot)
- 双变量-三变量散点图(relplot)
- 双变量-三变量连线图(relplot)
- 双变量-三变量简单拟合
- 分类数据的特殊绘图


#### 单变量分布
单变量分布可视化是通过将单变量数据进行统计从而实现画出概率分布的功能，**同时**概率分布有**直方图**与**概率分布曲线**两种形式。
利用displot()对单变量分布画出直方图(可以取消)，并自动进行概率分布的拟合(也可以使用参数取消)。

```python
sns.set_style('darkgrid')
x = np.random.randn(300)
sns.distplot(x);
```
{% asset_img img5.png img5 %}
还可以使用sns.kdeplot()画出核密度估计图。核密度估计是概率论上用来估计未知的密度函数，属于非参数检验，通过核密度估计图可以比较直观的看出样本数据本身的分布特征。
```python
x=np.random.randn(100)
#是否进行阴影处理
sns.kdeplot(x,shade=True,color="g")
```
{% asset_img img6.png img6 %}
二元kde图像，很少使用，稍微了解一下即可：
```python
#x,y
y=np.random.randn(100)
#cbar：参数若为True，则会添加一个颜色棒(颜色帮在二元kde图像中才有)
sns.kdeplot(x,y,shade=True,cbar=True)
```
{% asset_img img7.png img7 %}
#### 双变量分布
双变量分布通俗来说就是分析两个变量的联合概率分布和每一个变量的分布。
```python
import pandas as pd
mean, cov = [0, 1], [(1, .5), (.5, 1)]
data = np.random.multivariate_normal(mean, cov, 200)
df = pd.DataFrame(data, columns=["x", "y"])
sns.jointplot(x="x", y="y", data=df);
```
{% asset_img img8.png img8 %}
```python
# 同样可以使用曲线来拟合分布密度
sns.jointplot(x="x", y="y", data=df, kind="kde");
```
{% asset_img img9.png img9 %}

#### 数据集中成对双变量分析
```python
# 对角线化的是单变量的分布
iris = sns.load_dataset("iris")
sns.pairplot(iris);
```
{% asset_img img10.png img10 %}
可以进行一些配置：
- hue——》指定分类变量
- markers——》使用不同的形状


```python
g1 = sns.pairplot(iris, hue="species", markers=["o", "s", "D"])
```
{% asset_img img11.png img11 %}
还有其他配置如下:
- 使用调色板
- 使用 KDE
- 使用回归
  
```python
g2 = sns.pairplot(iris, hue="species", palette="husl")
g3 = sns.pairplot(iris, diag_kind="kde")
g4 = sns.pairplot(iris, kind="reg")
```

#### 双变量-三变量散点图
这里使用tips数据集：
```python
tips = sns.load_dataset("tips")
sns.relplot(x="total_bill", y="tip", data=tips);
# 除了画出双变量的散点图外，还可以利用颜色来增加一个维度将点分离开
sns.relplot(x="total_bill", y="tip", hue="smoker", data=tips);
```
**通过hue设置来进行区分**
{% asset_img img11.png img11 %}

#### 简单线性拟合
主要用regplot()进行画图，这个函数绘制两个变量的散点图，x和y，然后拟合回归模型并绘制得到的回归直线和该回归一个95％置信区间。
（不过一般这种工作可以用sklearn来做）
```python
sns.set_style('darkgrid')
sns.regplot(x="total_bill", y="tip", data=tips);
```
线性模型对某些数据可能适应不够好，可以使用高阶模型拟合，也可以删除部分数据：
```python
# 利用order2阶模型来拟合
sns.regplot(x="x", y="y", data=anscombe.query("dataset == 'II'"),ci=None,order = 2);
# 如果数据中有明显错误的数据点可以进行删除
sns.regplot(x="x", y="y", data=anscombe.query("dataset == 'III'"),ci=None);
plt.figure()
sns.regplot(x="x", y="y", data=anscombe.query("dataset == 'III'"),ci=None,robust = True);
```
#### 热力图
取出三个特征进行热力图的绘制figures.pivot() 其中第三个属性表示热力图上实际的值。
常用设置：
- annot：是否显示数值注释
- fmt：format的缩写，设置数值的格式化形式
- linewidths：热力图矩阵之间的间隔大小
- vmax,vmin：图例中最大值和最小值的显示值，没有该参数时默认不显示
- cmap：matplotlib的colormap名称或颜色对象；如果没有提供，默认为：
  - cubehelix map (数据集为连续数据集时)
  - RdBu_r (数据集为离散数据集时)
- xticklabels, yticklabels：绘制dataframe的行列名


```python
flights = sns.load_dataset('flights')
# 取出这三个属性画热力图，坐标点的位置是passengers
flights = flights.pivot('month', 'year', 'passengers')
ax = sns.heatmap(flights)
plt.show()

# 将实际的数值绘制到上面
flights = sns.load_dataset('flights')
# 取出这三个属性画热力图，坐标点的位置是passengers
flights = flights.pivot('month', 'year', 'passengers')
ax = sns.heatmap(flights, annot=True, fmt='d')
plt.show()
```
{% asset_img img12.png img12 %}

```python
plt.figure(figsize=(10,6))
ax = sns.heatmap(flights, fmt='d', linewidths=.5, cmap='YlGnBu')
```