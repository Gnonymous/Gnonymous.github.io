---
layout: post
title: "关于数字水印的简单了解"
date:   2024-03-21
tags: [paper]
comments: true
author: Gnonymous
---

### 水印嵌入原理：

> 水印作为隐藏信息，保护载体版权，查看是否被修改。

数据统计冗余：

> 压缩也是一种对冗余的修改——丢到冗余的信息
>
> 而水印就是将这些冗余的信息，修改为要添加的水印

* 空间冗余：白色像素与像素之间区别很小
* 时间冗余：相邻帧几乎没有区别

### 水印嵌入模型：

* 生成水印：W = F（I,W,K）

* 嵌入水印：I‘ = E （I,W）

* 提取水印：W‘ = D（I‘） 比对W与W‘，查看是否修改（盲水印，不需要原始I，就进行提取）
