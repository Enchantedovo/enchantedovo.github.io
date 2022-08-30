---
title: PTA习题2-7 
date: 2021-04-20 23:59:22
tags:
 - PTA
categories: 刷题
description: 🥦🐔刷题记录
---

# 弹球距离

设有一个球从高度为h米的地方落下，碰到地面后又弹到高度为原来p倍的位置，然后又落下，再弹起，再落下…。请编写函数求初始高度为h的球下落后到基本停下来（高度小于给定阈值TOL）时在空中所经过的路程总和。

函数接口定义：
```
double dist( double h, double p );
```
其中h是球的初始高度，p是球弹起高度与弹起前落下高度的比值；函数dist要返回球下落后到基本停下来时在空中所经过的路程总和。注意：当弹起的高度小于裁判程序定义的常数TOL时，弹起的距离不计算在内。

裁判测试程序样例：

```
#include <stdio.h>
#define TOL 1E-2

double dist( double h, double p );

int main()
{
    double h, p, d;
    scanf("%lf %lf", &h, &p);
    d = dist(h, p);
    printf("%.6f\n", d);
    return 0;
}

/* 你的代码将被嵌在这里 */

```

输入样例：
```1.0 0.4```
输出样例：
```2.319680```

**递归实现**
```
double dist( double h, double p ){
    if(p*h<TOL) return h;
    else return h+h*p+dist(h*p,p);//重复的
}
```
**非递归实现**
```
double dist( double h, double p )//右一个注意点两次触地时走过的距离为 h+2*h*p
{
    double mix=h*p;
    double sum = h;//第一次落下走过的距离是h
    while(mix>=TOL)
    {
        sum+= 2*mix;//第二次落下走过的是 h+2*h*p，依次类推
        mix = mix*p;
    }
    return sum;
}
```