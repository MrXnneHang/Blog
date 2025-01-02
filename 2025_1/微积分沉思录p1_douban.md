# 微积分沉思录. -p1 平均不需要微积分 - 深度学习框架中的 min,max,average 是怎么处理微分的.

## 一点琐事

> 今天课设完结了,之后就是属于自己的开发.
>
> 在全班人以及一群老师面前问 Qwen-7B-CHat "可以做爱吗?" 有一种莫名的刺激感.

## 前言:

我这次课程看的是[麻省理工微积分-比刷剧还爽](https://www.bilibili.com/video/BV1rY4y1P7er/).<br>

大概在我大一还在学高等数学的时候,我就看它"比刷剧还爽",但是当时看了两节就没往下看了.<br>

前两节都在举例说明什么是微分和积分.<br>

我记得在一本书还是一个视频里看过,大概是说,数学本身是一个抽象的过程,如果只以形象来解释抽象,比如一些视频里甚至做动画来演示,它会虚假地让我以为自己懂了,自己理解了.这是具体和形象的弊处.<br>

最终还是有必要从抽象层面理解它,那么到时候如果我试图形象地告诉别人,我就不会是回忆着用看到过的别人的形象的例子,而是随手捻起,就是一个具体例子,那很潇洒,但我不知道能不能做到.<br>

> 完了,我已经快写不下去了.真的不喜欢总结.开始胡扯.

## "平均不需要微积分"

微积分的对象是连续的瞬时对象,对于路程来说,就是知道每个时刻的位置,而不是一个小时后的位置,半个小时前的位置,那是用来求平均的,平均本身不具有瞬时性->平均是无所谓微积分的.<br>

有意思的是,在神经网络中,不少地方实际上都用到了 min,max,average 的方法.按理来说,单纯的这样的运算是不存在微积分的,也就是说,它是不存在梯度的.<br>

这就导致了一个结果: 链式求导法则在某个节点会发生中断.<br>

之前做的链式求导法则的考虑是建立在全连接层的.碰到激活函数还好,参数依然是可以求梯度的,但是如果碰到 min,max,average,这个梯度要怎么求?<br>

我记得之前剖析过 torch.autograd.grad.<br>

[第三次正经 PR】 对 paddle.grad 进行功能增强 - part1 理解自动微分框架](https://xnnehang.top/blog/169)<br>

实际上在每次要求梯度的时候,最终接受的是一个标量,一般用.sum(),或者用.average(),需要把它转成一个标量后才能求 grad, 求的过程就像开枝散叶一样.<br>

但是这里就扯到了一点对于.sum(),.average(),.max(),.max()这样的运算,按理来说不具有连续性和瞬时性,或者准确来说,它并不能被描述称一个连续函数.求导也就不会发生.<br>

那么对于这样的运算是如何"规定"导数值的?<br>

我之前的猜测是, sum(), average()后, 导数在传递的过程保持为 1,或者保持为加权值.<br>

但我们现在来研究一下.<br>

> 状态好多了,果然我还是更喜欢胡扯,想到哪里写到哪里,而不是总结,这样的行为对于看的人来说是灾难性的,对于我来说,也许也只是一种"徒劳",但是这样的徒劳,对于我本身而言,却是有难以言说的价值的. -- [<雪国> 二观后](https://xnnehang.top/blog/185).

## 哪些操作可能用到 max,min,average? 除了手动进行的部分.

> 然后这又是框架的内容了哈哈!
>
> 说来惭愧,之前被同一个同学问了两次 maxpooling 是什么,我都不能讲明白,我有必要好好补一下.

## 简单卷积,池化计算.

上次扯到了,在对于平均,以及 min,max 的操作,深度学习框架是如何对梯度进行处理的.<br>

在 `torch.autograd.grad` 中,求梯度的输入要求是一个标量,一般是 sum,或者 mean.<br>

### 卷积/Conv2d

```python
torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True, padding_mode='zeros', device=None, dtype=None)
```

这里重要的只有:<br>

```python
torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0)
```

in_channels,out_channels,是输入和输出的通道数.<br>

一般来说,对于一张图像,刚刚输入的时候 in_channels 是 3,然后 out_channels 是 n,可以自定义,一般是 64,128,256,512 这样的.<br>

简单理解是它把 RGB 进行不同维度的特征提取.<br>

我学过另外一个卷积,我已经把它忘了,似乎是同一个老师教的.<br>

kernel_size 定义的是卷积核的大小,一般是奇数,或者说,只能是奇数,因为卷积做的操作比较简单,比如对一个图像的 3x3 的一个区域和一个 3x3 的卷积核进行一一相乘,然后求和/求平均,把这个求和的值放在中心的那个像素.<br>

这么做和传统计算机视觉的滤波器或者说锐化和高斯模糊是一样的.计算过程是一样的.它本身也只是进行锐化或者模糊操作.<br>

区别在于,卷积是具有可学习的参数的.<br>

我们的卷积核是可更新的,这样子,就达到了一种,不同的卷积核,提取不同维度的特征,让模型在高维度下理解图像的目的,而不是既定的,那就叫滤波器了.<br>

stride 是步长,一般是 1,也就是每次移动一个像素,但是也可以是 2,3,4 这样的,这样子就会导致输出的图像变小.<br>

图像变小的过程,也是特征富集的过程,通常图像分类任务就是卷积一下,全连接一下,然后 RELU 一下,重复操作.确保每个维度都有足够的信息可以学习和支撑.<br>

padding 是填充.<br>

值得注意的一点,就是即使我们`stride=1`,只要我们的 `kernel_size!=1`, 那么我们最终卷积出来的图像就是会小一点,比如 3x3, 图像最后会小 2. 5x5, 图像最后会小 4.<br>

以前刚刚学用这个函数的时候这让我很困扰,因为我考虑不清楚输出的 size.<br>

但是实际上它就是一个滑块问题.<br>

而 padding 就是补齐由于滑块边缘和中心距离差的部分,让输出的 size 保持预期(简单的 x1,或者 x0.5)而补充的 1 或者 0.<br>

extra_practice:<br>

> 如果`stride=2`,`kernel_size=3`,`img_size=3x64x64`
> 如果我希望最后输出是 `16x32x32`,我的 padding,out_channels 应该怎么设置.

正确答案是:<br>

out_channels=16, padding=1, stride=2, kernel_size=3.<br>

padding 和 stride 无关,只和 kernel_size 有关.<br>

当 stride>kernel_size 的时候,也是允许的,但是会出现特征断层和信息丢失.<br>

![alt text](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2917012440.webp)

另外是一个有意思的操作: `kernel_size=1`<br>

kernel_size=1 的时候,模型相当于自己生成一个可以自我选择的特征,这个特征是可以学习的,但是不会改变图像的大小,也就是说,它是一个全连接层.<br>

![alt text](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2917012438.webp)

1x1 的卷积.<br>

对了,原本还要解析一下数学公式,但是卷积本身就只是根据学来的参数进行近似的一个函数,所以没必要写公式.<br>

它的计算方式和高斯模糊以及锐化一致,就是滤波器那个卷积.<br>

我记得还有个卷积是啥来着,和傅里叶一起学的.<br>

```txt
(f * g)(t) = ∫ f(τ)g(t - τ) dτ
```

这个,在信号与系统里学的.<br>

舒服了.<br>

### 池化/Pooling

我最经常听到的,MaxPooling.<br>

```python
torch.nn.MaxPool2d(kernel_size, stride=None, padding=0, dilation=1, return_indices=False, ceil_mode=False)
```

这里实际上没必要说的很多.<br>

Pooling 的计算方式,和卷积完全一致,只不过它不是可学习的,而是固定的.<br>

比如 MaxPooling,就是取一个区域的最大值,然后放在中心的那个像素.<br>

是最简单的锐化滤波器.<br>

它的作用是什么,是减少特征的维度,减少计算量,减少过拟合.<br>

一般都是在特征提取的最后一步.这一步就像在决定哪些信息是真正重要的,并且决定用这些进行分类.<br>

也是对模型的一种指导,应该把哪些特征提取出来.(设置成高应答)<br>

除了 MaxPooling,还有 AveragePooling,MinPooling.<br>

和滤波器一致,有机会可以做一个计算图的.<br>

表现一下计算的过程.<br>

## 关于卷积的 Extra-Thinking

对于灰度图的卷积,我们的`in_channels=1`,这个本身并不难理解.<br>

但是,对于我们的彩色图像进行卷积的时候,比如`in_channels=3,out_channels=16`.<br>

这个时候就会让我产生一个疑惑,那就是我要如何把 3 个通道映射到 16 个通道,我要如何分配?<br>

我是这么理解的:<br>

卷积核实际上也并不只是一层的,而是和 in_channels 一致,三个通道一起输入后卷积得到了一个 channels,然后以此类推得到 16 个 channels.<br>

有一点仍需确认,就是 Conv2d 和 Conv3d 的问题,我这么理解确实没有问题,但是它似乎是 3d 卷积核.但是我们函数调用的却是 Conv2d,而且也另外存在一个 Conv3d,那么我的理解是否正确,Conv3d 又是在做什么?<br>

首先得到确认的一点,卷积核确实是具有深度的:<br>

![image-20250102152546878](https://image.baidu.com/search/down?url=https://img2.doubanio.com/view/photo/l/public/p2917012441.webp)

另外,conv3d 和 conv2d 的区别不是在与卷积核,而是在于滑动方式,因为即使我们的 Conv2d 具有 3 维卷积核,但是滑动依然是在平面上的,而 Conv3d 是可以在 3 维滑动.<br>

![image-20250102152712257](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2917012443.webp)



暂时到这里.<br>
