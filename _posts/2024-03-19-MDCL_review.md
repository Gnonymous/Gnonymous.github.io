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

- **Background：**之前的检测工作大都把人脸伪造作为基于`交叉烱损失(cross-entropy)`的**二分类**问题，这样确实可以在训练集上取得不错的效果，但是在面对未见过的数据集时，该类模型强调**类别级差异**而非**真假人脸之间的本质差异**，限制了模型在不可视域的泛化。

- **Address：**提出了一种新的人脸伪造检测框架，命名为`Dual Contrastive Learning`。具体来说，结合**硬样本选择策略**(hard sample select)，首先提出了**实例间对比学习**`( Inter-Instance Contrastive Learning，Inter-ICL )`，通过特别构建实例对来促进任务相关的判别性特征学习。此外，为了进一步探索本质差异，引入**实例内对比学习**`( Intra-Instance Contrastive Learning，Intra-ICL )`，通过在实例内部构建局部区域对来关注伪造人脸中普遍存在的**局部内容不一致性**。

- **QAQ：**
  - ~~inter-ICL除了硬样本选择和之前的CL工作类似，主要还是添加了intra-ICL，这里大概是泛化提点的主要原因~~，而且DCL框架搭的很好，两个一组合故事能讲的蛮漂亮（有逻辑）。后面的`CADDM`也是在玩这方面的工作，似乎从`SBI`切进去的。
  
    > 这里把DCL的对比框架想简单了，QK查询键值对的对比思路使得在inter-ICL中也提高了泛化性，具体见[section: inter-ICL](#inter-ICL)
  
  - New pipeline：设计一个framwork关注挖掘特定feature，在original的数据集的基础上专项处理Input进framework训练，提升训练效果，`SBI` 、`CADDM`的处理思路一致，是一种启示。

#### Novelty：

* 我们提出了一种新颖的双重对比学习( Dual Contrastive Learning，DCL )用于一般的人脸伪造检测，该方法专门构造正负数据对，并在不同粒度下进行对比学习，以进一步提高泛化性；
* 我们专门设计了基于样本间实例对和样本内局部区域对的实例间对比学习和实例内对比学习，以学习与任务相关的本质特征；
* 大量的实验和可视化证明了我们的方法优于SOTA；

#### Method：

* **Framework**：
  ![image-20240320113404337](https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/DCL_framwork.png)

  DVG模块生成增强数据，views随机打乱混合输入2个流（带label，有监督的CL），双流Decoder share weight（参考MoCo MAE更新权重)，在f<sub>q</sub>后面外接全连接层output_2分类（L<sub>ce</sub> ）。`inter_ICL `的维持hard R/F队列，进行“查询”CL（L<sub>inter</sub>）；`intra_ICL`在pixel level上做CL，通过对比特征内的self-similarity来利用伪造人脸的不一致性。

* **DVG：**

  * **只对Fake**进行<u>随机概率下随机组合的增强</u>
  * 生成**一对一**的view，双流输入，在`inter-ICL`中进行QK“查询键值对“（**single image‘s different view contrast**）

* **<span id="inter-ICL">inter-ICL：</span>**

  * ICL：得益于VAG模块构造的不同view：细看DVG的构造，**v1和v2是一对一的**，inter-ICL的contrast是通过**键q值k对（v1-v2）**对应来对比学习的，也就是<u>**single  image的不同view对比（拉近）+硬样本对比（拉远）**</u>。这样只最大化了single image不同view的不变性（而不是同类目的不变性），在拉进single image views的距离的同时，同类目下不同image间仍保持一定距离——**保持了各类目（F/R）的方差**，保证了可迁移性（提高泛化）。

    可视化参见[Feature distribution](#Feature distribution)

    <div align=center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/DCL_inter-ICL.png"</div>

  * hard sample select ：维持一个R/F 的特征队列，挑选那些最像real的fake、最像fake的real入列（设定阈值）。作为hard sample进行CL。

    <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/DCL_hardsample1.png"</center>

* **intra-ICL：**

  * **Fake**：首先将fake img进行像素级分割——（**依据fake与对应real的差距生成的feature_mask,切割分为real_pixel and fake_pixel)**。L<sup>f</sup><sub> intra</sub> 依旧使用InfoNCE将真假pixel分开。

    <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/intra_fake_loss.png"</center>

  * **Real：**因为都为real pixel，希望保持self-similarity，采取前人做法（**转置**）：

    <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/intra_real.png"</center>

* **Loss：**

  分为3个部分：cls+inter+intra

  <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/DCL_loss.png"

#### Expriment：

* Cross-dataset evaluation:

  <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/DCL_cross.png">

* <span id ="Feature distribution">Feature distribution:</span>

  <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/DCL_feature%20distribution.png">







