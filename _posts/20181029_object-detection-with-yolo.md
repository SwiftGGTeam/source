 title: "使用 YOLO 进行实时目标检测"
 date: 2018-10-29
 tags: [机器学习]
 categories: [machinethink]
 permalink: object-detection-with-yolo
 keywords: 计算机视觉，目标检测，YOLO，机器学习
 description: 本文详细介绍了使用 YOLO 进行实时目标检测的原理，以及在 iOS 上的应用。
---
> 作者：Matthijs Hollemans，[原文链接](http://machinethink.net/blog/object-detection-with-yolo/)，原文日期：2018/03/28
> 译者：[阳仔](https://github.com/YangGao1991)；校对：[numbbbbb](http://numbbbbb.com/)，[小铁匠Linus](http://linusling.com)；定稿：[Forelax](http://forelax.space)
  
 
 
 
 
 
 

 <!--此处开始正文-->

目标检测是计算机视觉中的经典问题之一：

识别一幅图像中有哪些目标，以及它们在图像中的位置。

检测是一个比分类更复杂的问题，因为分类也可以识别目标，但不能准确判断目标在图像中的位置——并且分类不能适用于包含多个目标的图像。

![](http://machinethink.net/images/yolo/ClassificationVsDetection.png)

[YOLO](https://pjreddie.com/darknet/yolo/) 是一个实时有效的目标检测神经网络。

在这篇文章中，我将阐述如何使用 Metal Performance Shaders 来将“简化版” YOLOv2 运行在 iOS 设备上。

<!--more-->

在继续阅读之前，请先 [观看这个令人惊叹的 YOLOv2 介绍视频](https://www.youtube.com/watch?v=VOC3huqHrss)。

## YOLO 的工作原理

你可以使用 [VGGNet](http://machinethink.net/blog/convolutional-neural-networks-on-the-iphone-with-vggnet/) 或 [Inception](https://github.com/hollance/Forge/tree/master/Examples/Inception) 这样的分类器，通过一个小的滑动窗口对图像进行遍历，从而形成一个目标检测器。每一步遍历运行一次分类器，来对当前窗口中的目标进行分类。使用这样的滑动窗口，会对一幅图像输出成百上千的检测结果，但你只需要保留分类器最确定的那些结果。

这种方法是可行的，但很显然会很慢，因为需要运行很多遍分类器。一种效率稍微高一点的方法是，首先判断图像的哪一部分包含了有效的信息——也就是候选区域 (region proposals)——然后仅仅在这些区域中运行分类器。这种方法比滑动窗口的方法能减少分类器的运行次数，但仍然很多。

YOLO 采用了一种完全不同的方法。它并不是将传统的分类器改造成检测器。YOLO 实际上只对图像进行一次操作（也就是它名字的由来：You Only Look Once），但是是以一种聪明的方式。

YOLO 将图像分成 13×13 的网格单元：

![](http://machinethink.net/images/yolo/Grid.png)

每一个网格单元负责预测 5 个检测框。一个检测框是包含一个目标的矩形区域。

YOLO 同时会给出一个置信度，描述了某个检测框确实包含了某目标的确定程度。这个值和检测框中是什么目标毫无关系，只和检测框的形状大小匹配程度有关。

预测出的检测框看起来和下图类似（置信度越高，框越粗）：

![](http://machinethink.net/images/yolo/Boxes.png)

对每个检测框，对应的网格单元还给出了一个分类的预测。这和分类器的工作相似：它给出一个全部可能的类别的概率分布。我们使用的这个版本的 YOLO 是用 [PASCAL VOC dataset](http://host.robots.ox.ac.uk/pascal/VOC/) 训练的，可以检测 20 种不同的类别，比如：

- 自行车
- 船
- 汽车
- 猫
- 狗
- 人
- 等等

检测框的置信度和分类预测最终被整合成一个最终得分，来告诉我们该检测框内包含一个特定类型的目标的概率。例如，左边这个又大又粗的黄色框告诉我们，85% 的概率这里面包含一只狗：

![](http://machinethink.net/images/yolo/Scores.png)

因为整幅图像包含 13×13 = 169 个网格单元，且每个网格单元预测 5 个检测框，我们最后总计能得到 845 个检测框。实际上，这其中大多数的检测框的置信度都很低，所以，我们只需要保留最终得分大于等于 30% 的检测框就行了（你也可以根据需要的检测准确度改变这个阈值）。

最终的检测结果：

![](http://machinethink.net/images/yolo/Prediction.png)

从 845 个检测框中，我们只保留了这三个，因为它们给出的结果最好。尽管我们有 845 个检测框，但它们都是同时得到的——神经网络只需要运行一次。这也是 YOLO 强大而快速的原因。

（以上图片来自 [pjreddie.com](https://pjreddie.com/)）

## 神经网络

YOLO 的结构只是一个简单的卷积神经网络：

```
Layer         kernel  stride  output shape
---------------------------------------------
Input                          (416, 416, 3)
Convolution    3×3      1      (416, 416, 16)
MaxPooling     2×2      2      (208, 208, 16)
Convolution    3×3      1      (208, 208, 32)
MaxPooling     2×2      2      (104, 104, 32)
Convolution    3×3      1      (104, 104, 64)
MaxPooling     2×2      2      (52, 52, 64)
Convolution    3×3      1      (52, 52, 128)
MaxPooling     2×2      2      (26, 26, 128)
Convolution    3×3      1      (26, 26, 256)
MaxPooling     2×2      2      (13, 13, 256)
Convolution    3×3      1      (13, 13, 512)
MaxPooling     2×2      1      (13, 13, 512)
Convolution    3×3      1      (13, 13, 1024)
Convolution    3×3      1      (13, 13, 1024)
Convolution    1×1      1      (13, 13, 125)
---------------------------------------------
```

该神经网络具有典型的结构：一个 3×3 卷积核的卷积层，采样窗口 2×2 的池化层。没有花哨的东西。YOLO 中没有全连接层。

> 注意：我们使用的“简化版” YOLO 只有 9 个卷积层和 6 个池化层。完全版 YOLOv2 模型分层数是这个的三倍，并且会更复杂一些，但仍然是一个常规的卷积神经网络。
> 

最后一个卷积层的卷积核为 1×1，是为了将参数降维至 13×13×125。其中的 13×13 看起来很熟悉：这就是图像被划分成的网格单元的数量。

因此，每个网格单元有 125 个通道。这 125 个通道包含了检测框的数据，以及分类预测的数据。为什么是 125 呢？因为每个网格单元预测 5 个检测框的结果，每个检测框由 25 个数据元素来描述：

- 检测框矩形的 x，y，宽，高
- 置信度
- 在 20 个分类上的概率分布

YOLO 的使用很简单：输入一幅图像（大小为 416×416 像素），它运行一次卷积网络，输出一个 13×13×125 的张量，描述网格单元以及检测框。你需要做的只是计算每个检测框的最终得分，并抛弃低于 30% 的那些。

> 小提示：要了解更多有关 YOLO 的工作原理，以及 YOLO 是如何训练的，请 [观看对它的发明者之一的访谈](https://www.youtube.com/watch?v=NM6lrxy0bxs)。这个视频介绍的是 YOLOv1，由于是老版本，结构稍有不同，但主体思想是相同的。很值得观看！
>

## 转换成 Metal

上文描述的是简化版的 YOLO，也是我们将在 iOS 应用中使用的版本。完全版 YOLOv2 的神经网络有三倍的分层数，因为太大，所以在当前的 iPhone 设备上不能快速运行。简化版的 YOLO 使用更少的分层数，因此运行会更快，但准确度也稍差。

![](http://machinethink.net/images/yolo/CatOrDog.png)

YOLO 使用 Darknet 编写，这是 YOLO 作者自己编写的深度学习框架。能下载到的都是 Darknet 格式。尽管 [Darknet 是开源的](https://github.com/pjreddie/darknet)，我也不想花很多时间去弄清它的工作原理。

幸运的是，[有人](https://github.com/allanzelener/YAD2K/) 已经做了这件事，将 Darknet 模型转换成了我所使用的深度学习工具 Keras。我所要做的，就是运行这个“YAD2K”脚本，将 Darknet 转换成 Keras 格式，然后用我自己写的脚本将 Keras 转换成 Metal。

但是，有一点小麻烦。YOLO 在它的卷积层后，使用了一种叫做”批标准化“的规整化方法。

“批标准化”的思想是，当数据是干净的时候，神经网络能够达到最好的工作效果。理想情况下，一个层级的输入数据的平均值为 0，且方差较小。每个做过机器学习的人都应比较熟悉这一思想，因为我们经常使用一种称为“特征缩放”或“白化”的技术来处理我们的输入数据，以达到这一目的。

批标准化对层间数据做了类似特征缩放的处理。这种处理能防止数据在神经网络中传递时退化，从而有效提升神经网络的性能。

为了让你直观感受到批标准化的作用，以下是第一个卷积层分别在应用和未应用批标准化的情况下的输出直方图：

![](http://machinethink.net/images/yolo/BatchNorm.png)

批标准化在训练一个深度网络时很重要，但事实上，我们在进行推断的时候可以不需要这一处理。不需要进行批标准化的计算，有助于使我们的应用运行更快。在任何时候，Metal 都没有一个 `MPSCNNBatchNormalization` 层。

批标准化通常发生在卷积层之后，激活函数（YOLO 中的 ReLU 函数）之前。卷积操作和批标准化操作都是对数据进行线性变换，所以我们可以将批标准化层的参数和卷积权重相结合。这称为将批标准化层“折叠”到卷积层。

长话短说，使用一些数学方法，我们可以省略批标准化层，但需要改变前序卷积层的权重。

快速阐述一下卷积层的计算过程：设 `x` 是输入图像中的像素，`w` 是卷积层权重，那么，经过卷积层计算，输出的每个像素的值为：

```
out[j] = x[i]*w[0] + x[i+1]*w[1] + x[i+2]*w[2] + ... + x[i+k]*w[k] + b
```

即输入像素矩阵和卷积核权重的点积，再加上偏差项 `b`。

以下是对卷积层输出进行批标准化处理的计算过程：

```
        gamma * (out[j] - mean)
bn[j] = ---------------------- + beta
            sqrt(variance)
```

批标准化首先对每个像素的输出值减去平均值 `mean`，再除以标准差，乘以一个缩放系数 `gamma`，再加上一个偏移值 `beta`。这四个参数 —— `mean`，`variance`，`gamma`，`beta` —— 是在网络训练过程中，批标准化层学习得到的。

为了省略批标准化，我们可以将这两个公式进行稍微的整合，来为卷积层计算新的权重和偏差项：

```
           gamma * w
w_new = --------------
        sqrt(variance)

        gamma*(b - mean)
b_new = ---------------- + beta
         sqrt(variance)
```

利用这些新的权重和偏差项对输入 `x` 进行卷积操作，能够得到与原来卷积层加上批标准化处理后同样的结果。

现在，我们可以去掉批标准化层，只使用卷积层，但参数是经过调节的权重和偏差项 `w_new` 和 `b_new`。我们对网络中所有的卷积层都重复这一过程。

> 注意：事实上，YOLO 中的卷积层没有使用偏差项，因此上述公式中 b 为 0。但请注意，经过整合批标准化的参数后，卷积层就有了偏差项。
> 

一旦我们将所有的批标准化层整合到了它们的前序卷积层，我们就可以将权重转换到 Metal 了。简单地将该数组（Keras 中存储的顺序和 Metal 不同）进行转置，再将其写进 32 位浮点数的二进制文件。

如果你对这些操作感到好奇，你可以查看转换脚本 [yolo2metal.py](https://github.com/hollance/Forge/blob/master/Examples/YOLO/yolo2metal.py) 来获得详细信息。为了验证整合批标准化的效果，脚本创建了一个不包含批标准化层，但使用了调节后的权重的模型，并将其与原始模型的预测结果进行比对。

## iOS 应用

我理所当然地使用 [Forge](https://github.com/hollance/Forge) 来编写我的 iOS 应用。😂你可以在 [YOLO](https://github.com/hollance/Forge/tree/master/Examples/YOLO) 文件夹中找到源代码。如果想尝试一下的话，可以下载或者 clone Forge，在 Xcode 8.3 以上版本中打开 **Forge.xcworkspace**，在 iPhone 6 以上设备上运行 **YOLO**。

最简单的测试方法是将你的 iPhone 对准某个 [YouTube 视频](https://www.youtube.com/watch?v=e_WBuBqS9h8)：

![](http://machinethink.net/images/yolo/App.png)

**YOLO.swift** 中有一些有趣的代码。首先，这里创建了卷积网络：

```
let leaky = MPSCNNNeuronReLU(device: device, a: 0.1)

let input = Input()

let output = input
         --> Resize(width: 416, height: 416)
         --> Convolution(kernel: (3, 3), channels: 16, padding: true, activation: leaky, name: "conv1")
         --> MaxPooling(kernel: (2, 2), stride: (2, 2))
         --> Convolution(kernel: (3, 3), channels: 32, padding: true, activation: leaky, name: "conv2")
         --> MaxPooling(kernel: (2, 2), stride: (2, 2))
         --> ...and so on...
```

摄像头的输入图像被调整为 416×416 像素大小，接着被输入到卷积层和池化层。这和其他卷积神经网络的操作是非常类似的。

真正有意思的是对输出的操作。回想一下，我们的输出是一个 13×13×125 的张量：图像中每个网格单元有 125 个通道。这 125 个数字包含了检测框的数据，以及分类预测的数据。我们需要将这些数据通过某些方法进行整理。这些是通过 `fetchResult()` 实现的。

> 注意：fetchResult() 函数在 CPU 中运行，而非 GPU。这种实现方式比较简单。有人说 GPU 的并行性会对嵌套循环的运行比较有利。也许我在将来会重新写一个 GPU 的版本。
> 

以下是 `fetchResult()` 的工作原理：

```
public func fetchResult(inflightIndex: Int) -> NeuralNetworkResult<Prediction> {
  let featuresImage = model.outputImage(inflightIndex: inflightIndex)
  let features = featuresImage.toFloatArray()
```

卷积网络的输出是一个 `MPSImage` 格式的数据。我们首先将其转换成一个 `Float` 的数组，即 `features`，以便处理。

`fetchResult()` 的主体部分是一个大的嵌套循环。它对所有的网格单元以及每个网格单元的 5 个预测结果进行遍历：

```
  for cy in 0..<13 {
    for cx in 0..<13 {
      for b in 0..<5 {
         . . .
      }
    }
  }
```

在每个循环中我们对网格单元 `(cy, cx)` 计算出其检测框 `b`。

首先，我们从 `features` 数组中读取出检测框的 x，y，宽，高，以及置信度：

```
let channel = b*(numClasses + 5)
let tx = features[offset(channel, cx, cy)]
let ty = features[offset(channel + 1, cx, cy)]
let tw = features[offset(channel + 2, cx, cy)]
let th = features[offset(channel + 3, cx, cy)]
let tc = features[offset(channel + 4, cx, cy)]
```

`offset()` 函数的作用是帮助在数组中寻找读取数据的合适位置。Metal 将其数据以每 4 个通道为一组进行存储，这意味着这 125 个通道的数据并不是连续存储的，而是分散的。（请查阅代码以获得更详尽的解释）

我们仍然需要对这 5 个数据 `tx`，`ty`，`tw`，`th`，`tc` 进行处理，因为它们的格式有一点奇怪。如果你好奇这些公式是从哪里来的，它们是在 [这篇文章](https://arxiv.org/abs/1612.08242) 中提出的（这是网络训练的副产物）。

```
let x = (Float(cx) + Math.sigmoid(tx)) * 32
let y = (Float(cy) + Math.sigmoid(ty)) * 32

let w = exp(tw) * anchors[2*b    ] * 32
let h = exp(th) * anchors[2*b + 1] * 32

let confidence = Math.sigmoid(tc)
```

现在，`x` 和 `y` 代表在 416×416 大小的输入图像中，检测框的中心点的坐标。`w` 和 `h` 是检测框的宽和高。`tc` 是检测框的置信度，我们用 logistic sigmoid 函数将其转换成百分制的形式。

我们现在有了检测框，并且我们知道 YOLO 有多确信其中包含了某个目标。下一步，让我们看一下分类预测的结果，看看 YOLO 认为检测框中的目标是什么物体：

```
var classes = [Float](repeating: 0, count: numClasses)
for c in 0..<numClasses {
  classes[c] = features[offset(channel + 5 + c, cx, cy)]
}
classes = Math.softmax(classes)

let (detectedClass, bestClassScore) = classes.argmax()
```

回想一下，`features` 数组中的 20 个通道的数据包含了该检测框的分类检测结果。我们将这些读取到一个新的 `classes` 数组中。像分类器的一般做法一样，我们采用 softmax 函数来将数组转换成一个概率分布。然后，我们挑选分数最高的分类作为结果。

现在，我们可以对检测框计算最终得分了 —— 例如，“我有 85% 的确信度相信这个检测框包含了一只狗”。一共会有 845 个检测框，而我们只想保留最终得分超过某一阈值的那些。

```
let confidenceInClass = bestClassScore * confidence
if confidenceInClass > 0.3 {
  let rect = CGRect(x: CGFloat(x - w/2), y: CGFloat(y - h/2),
                    width: CGFloat(w), height: CGFloat(h))

  let prediction = Prediction(classIndex: detectedClass,
                              score: confidenceInClass,
                              rect: rect)
  predictions.append(prediction)
}
```

对所有网格单元重复上述代码。当循环结束后，我们就有了一个 `predictions` 数组，一般来说，其中会包含 10 到 20 个预测结果。

我们已经过滤掉了那些最终得分非常低的检测框，但剩下的检测框中，仍然可能会存在相互重叠特别严重的情况。因此，我们在 `fetchResult()` 中做的最后一件事就是用一种称为*非极大值抑制*的方法来减少这种重复的检测框。

```
  var result = NeuralNetworkResult<Prediction>()
  result.predictions = nonMaxSuppression(boxes: predictions,
                                         limit: 10, threshold: 0.5)
  return result
}
```

`nonMaxSuppression()` 函数所使用的算法很简单：

1. 从最终得分最高的那个检测框开始。
2. 将其他与该检测框重叠率超过一定阈值（比如超过 50%）的检测框移除。
3. 返回第一步，重复直到遍历完所有的检测框。

该算法移除了与更高得分的检测框有太多重叠的其他检测框，只保留了最好的那些。

以上就是所有的过程了：一个常规的卷积网络，以及后续对结果的一些处理。

## 运行效果如何？

[YOLO 官方网站](https://pjreddie.com/darknet/yolo/) 宣称精简版 YOLO 最快每秒可处理 200 帧图像。但那是在性能优秀的笔记本上的运行结果，而不是在移动设备上。那么，它在 iPhone 上能够运行多快呢？

在我的 iPhone 6s 上，它处理一幅图像大约需要 0.15 秒。那也只有 6 FPS，几乎不能称之为实时。如果你将手机对准一辆开过的汽车，你会看到一个检测框拖在汽车后面一些。尽管如此，它能够生效已经使我印象深刻了。😁

> 注意：正如上文解释，检测框的处理是在 CPU 上进行，而不是 GPU。如果 YOLO 完全运行在 GPU 上，会变得更快吗？也许会，但 CPU 代码运行时间只有 0.03 秒，占 20% 的运行时间。将其中一部分工作交给 GPU 做当然是可行的，但考虑到卷积层的计算仍然占据了 80% 的时间，我不确定是否值得这样做。
> 

我认为导致变慢的主要原因在于输出通道为 512 和 1024 卷积层。经过实验，`MPSCNNConvolution` 在通道较多的小图片上的表现比在通道较少的大图片上会更差。

另一件我比较感兴趣的事情是采用另一种不同的网络结构，例如 SqueezeNet，对其重新进行训练，以在最后一层进行检测框的预测。换句话说，采用 YOLO 的思想，并将其应用在一个更小更快的网络上。这样以精确度的损失换来速度上的提升是否值得呢？

> 注意：顺便说一句，最近的 [Caffe2](http://caffe2.ai/) 框架也是通过 Metal 的支持，运行在 iOS 设备上。[Caffe2-iOS project](https://github.com/KleinYuan/Caffe2-iOS) 是一个针对 YOLO 的精简版本。看起来它运行得会比纯 Metal 版稍慢，大约 0.17 秒/帧。
> 

## 后记

想要了解更多 YOLO 相关的知识，可以查看 YOLO 作者的以下论文：

- [You Only Look Once: Unified, Real-Time Object Detection](https://arxiv.org/abs/1506.02640) by Joseph Redmon, Santosh Divvala, Ross Girshick, Ali Farhadi (2015)
- [YOLO9000: Better, Faster, Stronger](https://arxiv.org/abs/1612.08242) by Joseph Redmon and Ali Farhadi (2016)

我的实现一部分基于 TensorFlow Android demo [TF Detect](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/android)，Allan Zelener 的 [YAD2K](https://github.com/allanzelener/YAD2K/)，以及 [Darknet 源代码](https://github.com/pjreddie/darknet)。
> 本文由 SwiftGG 翻译组翻译，已经获得作者翻译授权，最新文章请访问 [http://swift.gg](http://swift.gg)。