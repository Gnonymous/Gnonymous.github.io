---
layout: post
title: "DCL&&MDCL的联系与回顾"
date:   2024-03-19
tags: [paper]
comments: true
author: Gnonymous

---

### DCL：

![image-20240320104635957](https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/image-20240320104635957.png)

>K. Sun, T. Yao, S. Chen, S. Ding, J. Li和R. Ji, 《Dual Contrastive Learning for General Face Forgery Detection》, *AAAI*, 卷 36, 期 2, 页 2316–2324, 6月 2022, doi: [10.1609/aaai.v36i2.20130](https://doi.org/10.1609/aaai.v36i2.20130).

#### Motivation：

​	**Background：**之前的检测工作大都把人脸伪造作为基于`交叉烱损失(cross-entropy)`的**二分类**问题，这样确实可以在训练集上取得不错的效果，但是在面对未见过的数据集时，该类模型强调**类别级差异**而非**真假人脸之间的本质差异**，限制了模型在不可视域的泛化。

​	**Address：**提出了一种新的人脸伪造检测框架，命名为`Dual Contrastive Learning`。具体来说，结合**硬样本选择策略**(hard sample select)，首先提出了**实例间对比学习**`( Inter-Instance Contrastive Learning，Inter-ICL )`，通过特别构建实例对来促进任务相关的判别性特征学习。此外，为了进一步探索本质差异，引入**实例内对比学习**`( Intra-Instance Contrastive Learning，Intra-ICL )`，通过在实例内部构建局部区域对来关注伪造人脸中普遍存在的**局部内容不一致性**。

​	**QAQ：**

1. inter-ICL除了硬样本选择和之前的CL工作类似，主要还是添加了intra-ICL，这里大概是泛化提点的主要原因，而且DCL框架搭的很好，两个一组合故事能讲的蛮漂亮（有逻辑）。后面的`CADDM`也是在玩这方面的工作，似乎从`SBI`切进去的。
2. New

#### Novelty：

* 我们提出了一种新颖的双重对比学习( Dual Contrastive Learning，DCL )用于一般的人脸伪造检测，该方法专门构造正负数据对，并在不同粒度下进行对比学习，以进一步提高泛化性；
* 我们专门设计了基于样本间实例对和样本内局部区域对的实例间对比学习和实例内对比学习，以学习与任务相关的本质特征；
* 大量的实验和可视化证明了我们的方法优于SOTA；

#### Method：

#### Framework：

**DAG：**

**inter-ICL：**

**intra-ICL：**

**Loss：**

#### Expriment：

