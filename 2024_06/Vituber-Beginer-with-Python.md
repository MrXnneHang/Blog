## Vtuber-Beginer-with-Python





目前我们的项目分成两方向走。一步是继续开发和训练我们的基础模型，UGATIT。

而这里记录的是往功能性走（我们也会借助stable-diffusion来让图像更具调整性）。比如将二次元图像转换成live2d可面部捕捉的模型:

比如这个项目：

### 项目介绍:

完全基于python的vituber制作:[https://github.com/RimoChan/Vtuber_Tutorial](https://github.com/RimoChan/Vtuber_Tutorial)

给出了一种简单的，将.psd的多图层图像的分图层提取和渲染，进行面补的算法形成类似live2d模型的算法。

简要原理:

input:（分图层的图像）

<img src="https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406292246295.jpeg" alt="Snipaste_2024-06-29_22-38-20"  />

![Snipaste_2024-06-29_22-38-29](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406292246557.jpeg)

分图层渲染，并且将所有图层贴在一个类似球面上，渲染出一种伪立体感。

![Snipaste_2024-06-29_22-39-45](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406292248292.jpeg)

对于变形区域，比如嘴巴，我们是通过图层控制的。并且将嘴巴复制了一份（上嘴唇和下嘴唇），当不说话时重合，说话时则控制变形。

<img src="https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406292250417.jpeg" alt="Snipaste_2024-06-29_22-49-12" style="zoom:200%;" /><img src="https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406292252205.jpeg" alt="Snipaste_2024-06-29_22-49-39" style="zoom:200%;" />       

最终效果如下:

![6-5](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406292253366.webp)

### 我们接下来的主要工作:

眼睛眨动还需隐藏图层，暂时并未完成，嘴巴还未填色。

而目前主要的任务则是考虑**如何将png单图层拆分成理想图层**

这个要考虑到干净拆分，不管用精度多高的AI模型也还是不如画师提供的图层干净。主要的一点是，市面上也没有人特地为了画风差异巨大的二次元人物制作这种图像分割。

**但是，我们想到了解法。而且是极其简单的且高效的。**

如果我大一时没有学几个月板绘还不会想到这个方法，板绘分成平涂和厚涂。简单理解厚涂的就是那些写实风的，通常画法和油画一样的，用色很复杂。而平涂则是源自于早期日本动漫，初音未来那样的画风。我们看这样一个图片：

![Snipaste_2024-06-29_22-40-49](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406292259864.jpeg)

这是一个平涂图像，对于这样的图像来说，用色通常都很简单，基础色，阴影，光照色。

对于嘴巴来说，就只有亮暗之分。颜色为两种。（而且这两种就是没有任何跨度的，明确的两个rgb值，只取两个色，这就是平涂）

肤色也时同理，不管脸部手部肩部肤色都是同样，一个基础色一个阴影。头发通常还会有一个光照色。**而且基础色，阴影，光照色通常还是一个色相，不同亮度和饱和度（光照色亮度高饱和度低，阴影亮度低饱和度高。）**

**也就是说，一张平涂动漫画风的图像，通常情况下，颜色是可以数出来的。有限的**

我们的思路是：

* 先对图像进行一个规格化，将图像相邻像素之间变化不大的全部标准化。对于我们这个psd最终剩下的颜色区域不超过10种。
* 将每个颜色区域切分并且输出到我们的分类模型。（微调后的resnet18表现得就很好),分类输出class为hair，eye，body，mouth,others
* 将相同的class拼接到一起，并且将不联通的区域用canny检测轮廓进行联通。
* 这样我们就能够input:png和jpg图像 -> output：分图层的图像。
* 之后就可以继续我们上述面捕live2d步骤。