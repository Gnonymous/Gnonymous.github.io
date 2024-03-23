---
layout: post
title: "SBI的回顾与细节"
date:   2024-03-23
tags: [paper]
comments: true
author: Gnonymous
---

<center><img  src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/SBI.png">

>K. Shiohara和T. Yamasaki, 《Detecting Deepfakes with Self-Blended Images》, 收入 *2022 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*, New Orleans, LA, USA: IEEE, 6月 2022, 页 18699–18708. doi: [10.1109/CVPR52688.2022.01816](https://doi.org/10.1109/CVPR52688.2022.01816).

#### <span id="Motivation">Motivation:</span>

* **Background:**
  1. 目前大多模型在未知伪造方法的测试集上泛化效果不佳。
  2. 该问题最有效的解决方案之一是使用合成数据训练模型，这鼓励模型学习通用的表示来进行深度伪造检测。而目前大多方法处理此步骤时开销太大（需要先landmark matching后再blend）。

* **Address:**

  1. 提出了一种新的合成训练数据，称为**自混合图像( SBIs )**来检测深度伪造。SBI完全由**真实的单张图片（自己和自己blend）**混合生成，再现了常见的Artifacts(例如：混合边界和统计不一致性)。SBIs的核心思想是，*更一般和难以识别的伪造样本鼓励分类器学习通用和稳健的表示，而不会过度拟合操纵特定的伪造。*

     * 我们分析了伪造的人脸，并定义了4个典型的artifacts，这些artifacts源自先前的工作(例如,混合边界 ,源特征不一致和频域统计异常）

       > 更能帮助学习提高泛化性的 伪造痕迹——>让模型学习到

       <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/SBI_artifacts.png">

       ​		

     * 通过将源图像和目标图像与掩模进行混合，得到了SBI。SBI的训练鼓励模型学习通用表示，因为模型学习我们在[STG](#STG)中主动生成的伪造痕迹。

  2. 以往的方法融合两个不同的面孔，并根据选择的源图像和目标图像之间的差距生成伪影。相比之下，我们的方法仅从单张图像中混合了轻微变化的面孔，并通过变换主动生成伪影。

     <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/SBI_efficient.png">

* **QAQ:**
  * 总得来说，就是生成具有**更明确伪造痕迹的训练集**，训练模型**更关注这些Atifacts**，从而提升鲁棒性。Self-Blend里的这个`Self`很有创意（因为只用real进行处理训练），paper写的也很好，nice~
  * 后面的[Method](#Method)主要在说如何生成SBIs.

#### Novelty:

参见[Motivation模块](#Motivation)吧，里面写的蛮清楚了。

#### <span id="Method">Method:</span>

* **Framework:**

<center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/SBI_framwork.png">

~~~python
# 算法 1: SBIs 生成伪代码
# 输入: 大小为 (H, W, 3) 的基础图像 I，大小为 (81, 2) 的面部标记 L
# 输出: 大小为 (H, W, 3) 的自混合图像 ISB

def T(I):
    # 源-目标增强
    I = ColorTransform(I)  # 颜色转换
    I = FrequencyTransform(I)  # 频率转换
    return I

if Uniform(min=0, max=1) < 0.5:
    Is, It = T(I), I
else:
    Is, It = I, T(I)

p = RandomResizeTranslate(Is)  # p: 参数
L = LandmarkTransform(L)
M = ConvexHull(L)
M = ParameterizedResizeTranslate(M, p)
M = MaskDeform(M)
r = Uniform({0.25, 0.5, 0.75, 1, 1, 1})
M = r * M
ISB = Is ⊙ M + It ⊙ (1 − M)
~~~

* **<span id ="STG">STG:</span>**

  * 用于融合的伪源图像和目标图像。对源图像和目标图像进行增强，生成它们之间的统计不一致性(例如,颜色和频率)。源图像也被重新缩放和翻译，以再现混合边界和地标不匹配。

* **MG:**

  * 生成带有一些变形的灰度掩模图像。

  * `MG`提供了一个灰度掩膜图像来融合源图像和目标图像。为此，`MG`根据BI [ 40 ]中使用的界标变换对掩模进行变形。为了增加混合掩模的多样性，掩模的形状和混合比例是随机变化的。首先，根据文献[66][1]中的方法，掩模被弹性变形。其次，利用两个不同参数的高斯滤波器对掩模进行平滑。第一次平滑后，小于1的像素值变为0。这意味着如果第一个高斯滤波器的核尺寸大于第二个高斯滤波器的核尺寸，则掩模被腐蚀，反之则被膨胀。

    >如果第一个高斯滤波器的核尺寸大于第二个高斯滤波器的核尺寸，则会导致混合遮罩的效果类似于腐蚀操作。这是因为第一个高斯模糊会使得遮罩的边缘变得更加模糊，而第二个高斯模糊会进一步模糊遮罩。因此，模糊后的遮罩中的边缘部分将会受到更强的影响，从而使得遮罩向内收缩，类似于腐蚀操作。
    >
    >相反，如果第一个高斯滤波器的核尺寸小于第二个高斯滤波器的核尺寸，则会导致混合遮罩的效果类似于膨胀操作。这是因为第一个高斯模糊会使得遮罩的边缘部分变得模糊，而第二个高斯模糊会进一步模糊遮罩但是在更大的范围内。这会导致遮罩的边缘部分受到更强的影响，从而使得遮罩向外膨胀，类似于膨胀操作。

  ~~~python
  def get_blend_mask(mask):
        # 获取输入遮罩的形状
        H, W = mask.shape
        
        # 随机生成新的遮罩大小
        size_h = np.random.randint(192, 257)
        size_w = np.random.randint(192, 257)
        
        # 将输入遮罩调整到新的大小
        mask = cv2.resize(mask, (size_w, size_h))
        
        # 随机生成两个高斯滤波器的核大小
        kernel_1 = random.randrange(5, 26, 2)
        kernel_1 = (kernel_1, kernel_1)
        kernel_2 = random.randrange(5, 26, 2)
        kernel_2 = (kernel_2, kernel_2)
        
        # 对遮罩进行第一次高斯模糊
        mask_blured = cv2.GaussianBlur(mask, kernel_1, 0)
        mask_blured = mask_blured / (mask_blured.max())  # 归一化
        mask_blured[mask_blured < 1] = 0  # 将小于1的像素值设为0
        
        # 对模糊后的遮罩再进行一次高斯模糊
        mask_blured = cv2.GaussianBlur(mask_blured, kernel_2, np.random.randint(5, 46))
        mask_blured = mask_blured / (mask_blured.max())  # 归一化
        
        # 调整遮罩大小为原始输入遮罩的大小
        mask_blured = cv2.resize(mask_blured, (W, H))
        
        # 将遮罩转换为三维张量（加入了通道维度）
        return mask_blured.reshape((mask_blured.shape + (1,)))
  ~~~

* **Blending:**

  * ISB ← Is ⊙ M + It ⊙ (1 − M )

    >在图像处理中，两幅图像的相乘和相减操作是指逐像素进行的操作。这些操作通常用于实现图像混合或融合的效果。
    >
    >1. **图像相乘**：将两幅图像的对应像素相乘。在图像混合中，一般将一个图像乘以一个遮罩，遮罩中的值决定了源图像和目标图像在混合中的权重，值越大，对应像素在混合结果中所占比例越大。
    >
    >2. **图像相减**：将两幅图像的对应像素相减。在图像处理中，可以用来减少图像的背景或者执行一些边缘检测等操作。
    >
    >在下面的代码中，源图像（source）和目标图像（target）通过混合遮罩（mask_blurred）进行了线性混合。混合遮罩中的值确定了每个像素在混合结果中所占的比例，即它们的权重。通过相乘和相减操作，可以根据混合遮罩的不同值，实现源图像和目标图像的逐像素混合，最终得到混合后的图像。

  ~~~python
  def dynamic_blend(source, target, mask):
      # 获取模糊后的混合遮罩
      mask_blured = get_blend_mask(mask)
      
      # 定义混合比例列表
      blend_list = [0.25, 0.5, 0.75, 1, 1, 1]  # 论文里MG模块的混合比例
      
      # 随机选择混合比例
      blend_ratio = blend_list[np.random.randint(len(blend_list))]
      
      # 将模糊后的混合遮罩乘以混合比例
      mask_blured *= blend_ratio
      
      # 进行图像混合
      img_blended = (mask_blured * source + (1 - mask_blured) * target)  # 论文中SBI混合的最终函数
      
      return img_blended, mask_blured
  ~~~

#### Expriment:

* **Cross-Test:**

  <center><img src="https://raw.githubusercontent.com/Gnonymous/Gnonymous.github.io/master/images/SBI_test.png">

[1]: https://openaccess.thecvf.com/content/ICCV2021/html/Zhao_Learning_Self-Consistency_for_Deepfake_Detection_ICCV_2021_paper.html "Zhao, Xiang Xu, Mingze Xu, Hui Ding, Yuanjun Xiong, and Wei Xia. Learning self-consistency for deepfake detection. In ICCV, pages 15023–15033, 2021. 1, 2, 3, 4, 5, 6"
