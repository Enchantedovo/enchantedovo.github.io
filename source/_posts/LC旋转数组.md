---
title: LC旋转数组
date: 2021-05-20 23:23:42
tags:
 - LC
 - 数组
categories: 刷题
description: 🥦🐔刷题记录
---

# 旋转数组
给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。

> **示例 1:**
> - 输入: nums = [1,2,3,4,5,6,7], k = 3
> - 输出: [5,6,7,1,2,3,4]
> - 解释:
>   - 向右旋转 1 步: [7,1,2,3,4,5,6]
>   - 向右旋转 2 步: [6,7,1,2,3,4,5]
>   - 向右旋转 3 步: [5,6,7,1,2,3,4]

> **示例 2:**
> - 输入：nums = [-1,-100,3,99], k = 2
> - 输出：[3,99,-1,-100]
> - 解释: 
>   - 向右旋转 1 步: [99,-1,-100,3]
>   - 向右旋转 2 步: [3,99,-1,-100]
 
## 方法一：傻瓜方法

```c
void rotate(int* nums, int numsSize, int k){
    int length = numsSize;
    int list[length];
    for(int i=0;i<length;i++){
        list[i]=nums[i];
    }
    for(int j=0;j<length;j++){
        nums[(j+k)%length]=list[j];
    }
}
```

## 方法二：旋转旋转旋转

举个例子：

> nums = [1,2,3,4,5,6,7], k = 3
> 1. [7,6,5,4,3,2,1] (整个数组旋转)
> 2. [[5,6,7],[1,2,3,4]]


```c
void rotate(int* nums, int numsSize, int k){
    //先旋转整个数组-》旋转前一部分-》旋转后一部分
    k=k%numsSize;
    reverse(nums,0,numsSize-1);
    reverse(nums,0,k-1);
    reverse(nums,k,numsSize-1);
}

void reverse(int* nums, int start, int end){
    while(start<end){
        int tmp = nums[start];
        nums[start++]=nums[end];
        nums[end--]=tmp;
    }
}

```