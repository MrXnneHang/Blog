# ADL使用介绍:

![1-BERT-VITS2 Sovits 如何优雅而又偷懒地制作一份音频的数据集-1080P 60帧-AVC.Cover](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406231615290.jpg)



这里是在开发ADL V2的过程中写的文档。

同样也是为了给自己以后查询。

## 模块一:单人说话音频。

> 描述:比如一些游戏录屏，**只包含一个人的声音**，无法处理多人音频，无法排除游戏内人声（比如英雄的配音）的干扰。【这个之后有一个按说话人清洗模块可以解决。】

main_files:

single_person_step1.py:clip

single_person_step2.py:cut&text recognize

config.yml: 配置文件

### 功能：

* 1.clip:这步会把音频中的空白背景音全部去掉，并且每句话间隔1.5s，对于不怎么健谈的游戏主播来说，这一步可能会把原本半小时的音频变成五分钟。这步可以减少很多降噪的时间。
* 2.cut:这步会根据把每句话切分开，成为单独的wav
* 3.text recognize:识别切分后的wav，写入text和基本信息到./esd.list

### 重要参数config.yml:

![Snipaste_2024-06-23_17-17-48](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406231718237.jpeg)

你可以用任何你喜欢的文本编辑器编辑它,但修改前记得备份。

* **device**: 如果你没有nvidia显卡，可以用"cpu"
* **cut_line(ms):**如果识别到的一个句子中间间隔太长，超过cut_line，就把把它分成两句。设的越小，生成的句子越短。不应该短于combine_line。
* **combine_line(ms):**如果识别到的两个句子之间间隔短于combine_line，就把它们两句合并，设的越大，生成的长句越多，不应该长于cut_line。
* **ignore_line(ms):**如果run_cut过程中，这个句子短于ignore_line，就把它忽略掉。比如设成2.5s（2500），则切分的所有句子都会长于2.5s。

### 使用方法:

* **检查input:**

将需要处理的音频放到./raw_audio下。

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406231659924.jpeg)

命名规则不限，但是**不允许文件名中存在多个.**，最好还是简单些。

支持转录的格式为**wav**.

* **run_desk.bat **



![Snipaste_2024-06-23_16-55-55](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406231700827.jpeg)

你需要依次执行1和2。

在执行完1之后，你可以选择对音频进行降噪。这个时候音频会在./raw_audio下，显示为processed_wavname。**旧文件会被删除，注意备份。**

执行2后，文件会在./dataset/你取的名字下方，写入list文件为esd.list。

* **对音频进行精修(可选):**

对应run_desk.bat中的第三步。

![Snipaste_2024-06-23_17-03-31](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406231704015.jpeg)

进入local URL，一般情况下都是127.0.0.1:7860，端口占用的话一般会是7861或者7862等等。我会在模块2中介绍如何使用webui进行手动清理数据集。



## 模块二:数据集手动清洗。



![Snipaste_2024-06-23_17-06-04](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406231707749.jpeg)

* 如果你要对一条音频进行修改操作，先勾选右边的choose audio，否则修改无效,即使切换页面，也需要保存修改。
* 文本是可以手动修改的，记得保存文本。但我更建议检查一遍后调整一下hot_words.txt里的内容。

* 不建议使用分割和合并音频，因为我的模块一中可以自动分割生成长音频或者短音频。你可以通过更改cut_line和combine_line来控制生成句子的整体连贯性和长度。

* 修改完后记得保存修改。另外这里的保存修改不会直接改变./dataset/下的音频，一般只会影响esd.list。所以记得备份你的esd.list，并且和你的数据集放在一起。

* ![Snipaste_2024-06-23_17-12-52](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406231713427.jpeg)

  下方可以切换深色模式。



## 模块三: Speaker检测

### 用法一：用于单人音频中的清洗。(Building...)

* 1.录屏中，可能会有一些游戏中的角色讲话，哪怕像奥日与萤火意志，或者spiritfarer这种无对话配音的，与npc对话的时候精灵会发出一些声音，它虽然不成句子，但是正好碰到主播讲话的话，就会被切片到一起。这个在降噪中是没有办法被当作背景音降掉的，而被作为人声保留。

* 2.一些游戏结束时的音乐，可能会有人唱歌的声音，也会被切进来，当作数据集的一部分。

这种情况除非人工全部去听一遍，否则很难排除，但那真的不现实，对我来说。

**想到的方法:**

阿里的cam++提取整体的spk_embedding取中位值作为target

```
input = model([wav1,wav2,wav3...])
output = [spk1_embedding,spk2_embedding,spk3_embedding..]
```

eres2net

```
input = model([wav1,wav2])
output = {socre:0.68,text:"yes"}
```

ere2net的一个缺点，是必须选取一个target，但是target的选择，很难说，手动选择，不能够说代表target_spk的一个特征，求相似度会有很多问题。

### 用法二:用于电影，电视剧，动漫中的特定人物音频提取。

这个任务相对比较适合，因为重叠率低，而且一般情况下音色容易区分开，而且录音设备也都比较好。

但是之前在区分花城和谢怜的声线的时候让我原地爆炸，情感一飙升就变成新的spk，花城谢怜分不清，同时也让我对国内配音小小失望了一波。AI都分不开，总不能叫我能分开吧=-=。

### 用法三: 用于类似访谈，综艺，电台的说话人分类提取。

这类任务最难，因为相比电影和动漫那样的配音，以上这些场景中的说话人重叠几乎是灾难性的。现实中常常是一个人讲话打断另一个讲话，前一个被打断后还会说，好你说。重叠率几乎可以到20%到30%。



/2024/6/24：

如果可以的话我想还是想要保持使用阿里的说话人验证。

这是一个遗留的算法问题。

我认为分布应该是这样的，黑色的是(npc语音，混合语音等等，当然混合语音是否真的会再target之外有待考究。)，以及target_speaker应该占据80%~90%:

![Snipaste_2024-06-25_17-32-02](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406251732863.jpeg)

经过测试，以上是完全理想(幻想)的状态。
实际上会像这样，经过降维后投影:
 ![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406252219364.jpeg)

几个红色的点是中位数，但是很难说，我以为取了中位数就不会有混合语音，但是依然有碰到过混合。也就是说speaker的embedding不能够直接作为说话人分类的依据。至少，很难以正常的欧氏距离等正常数学逻辑去解释embedding。

目前可以考虑的策略:

利用带有说话人日志的模型，先检测音频片段中存在说话重叠的情况或者多个说话人的情况丢掉。

但在经过speaker embedding的聚类后我发现，这种聚类并不是毫无意义的，得到了target speaker的声线不一定就得到了可用的数据集。有时候target speaker会发出一些一惊一乍的声音，或者含含糊糊，总之就是一些奇怪的声音，这些也许被反映在emotion里面，然后就被检测得embedding离得很远。通过移除这些距离很远的speaker embedding可以排除掉一部分数据集噪声，这个在eres2net中的体现就是相似度很低。但其实一个好的模型应该具有一定的鲁棒性，应该是会自适应的，我也很难说完全无噪声的数据集就会更好。但这是一个embedding的使用方向。

```
input(model,[wav1,wav2,wav3,wav4])
output(spk1_embedding,spk2_embedding,spk3_embedding,spk4_embedding)
```

```
{'spk_embedding': tensor([[-1.8740e+00,  6.2234e-01,  4.8233e-01,  1.0637e+00,  4.2049e-02,
          1.9299e-01,  2.1912e-01,  1.5101e-03,  4.4940e-01, -9.7277e-01,
          3.1258e-01, -8.2188e-01,  1.6454e+00, -4.3867e-01, -5.6806e-01,
         -2.9532e-01, -6.8543e-01, -1.7338e-01,  3.8926e-01,  3.8491e-01,
         -9.5980e-01, -5.9897e-01,  3.2874e-01,  2.8249e+00, -1.1416e+00,
         -1.3728e+00,  8.0004e-01,  1.6199e+00, -4.6849e-01,  8.2824e-01,
         -2.2879e+00, -1.1475e+00, -1.8962e-01, -8.7403e-01, -1.0274e+00,
         -1.3192e+00,  1.9436e-01, -1.5972e+00, -1.0400e+00, -1.1597e+00,
          6.9777e-02,  9.8540e-01,  2.9216e-01,  4.7092e-01,  8.8029e-01,
          3.4071e-01,  1.0102e+00,  8.8783e-01,  4.5629e-01,  4.5567e-01,
          2.8913e-02,  7.0048e-01,  1.4592e-01, -3.9454e-01, -3.9745e-01,
         -5.6236e-01, -1.6243e+00, -7.9021e-02, -2.2990e-01, -8.7929e-01,
         -1.2565e+00, -1.3947e+00, -4.7982e-01, -4.4142e-01, -6.2106e-01,
          2.3665e-01, -7.4867e-01,  8.0137e-01,  4.7470e-01,  2.9406e+00,
          2.4534e-01, -2.7143e-01, -2.2567e+00, -1.3455e+00,  1.4965e-01,
         -2.1847e+00, -3.9912e-01, -4.8649e-01, -1.7075e+00,  1.0705e+00,
          2.8171e-01,  1.9936e+00, -4.2651e-01,  1.0114e+00, -4.0167e-01,
         -7.1657e-01,  5.0901e-01, -1.1157e+00,  6.1037e-01,  3.2058e-01,
          4.9221e-01,  1.2163e+00, -1.1269e-01, -7.4668e-01, -7.7454e-01,
          2.7185e-01, -9.0966e-02, -2.4865e+00,  2.6583e-01, -2.1350e-01,
         -9.4688e-02, -5.8378e-01, -2.3522e-01,  1.6224e-01,  1.0168e+00,
          1.9655e+00,  3.4811e-01,  1.7289e+00,  1.3207e+00, -1.4285e+00,
          1.4882e+00, -6.2862e-01, -7.8585e-01,  1.1489e+00, -7.1737e-02,
         -4.5864e-01, -1.7913e-01,  1.9200e-01,  6.3087e-01,  6.8130e-01,
          1.1074e-02, -1.2537e+00,  1.8152e-02, -1.4835e+00, -8.0153e-01,
         -1.1703e+00,  5.0475e-01, -1.5186e+00, -1.7251e+00, -1.6490e+00,
          4.8440e-01, -4.0540e-01, -5.5303e-01,  4.9784e-02, -5.0701e-01,
         -2.7794e-01, -1.1784e+00, -6.0908e-01, -2.7624e-02,  2.7073e-01,
          1.1687e+00, -1.0512e+00, -9.1582e-01,  4.2886e-01,  5.6452e-01,
          3.2301e-01, -3.1386e-01,  9.4958e-01,  2.3514e-01,  8.9597e-01,
         -2.4890e+00, -1.5485e+00,  2.3446e+00,  9.8548e-01,  4.4322e-01,
         -1.7362e+00, -7.6740e-01,  2.8641e-02, -3.9433e-01, -1.2189e+00,
          1.3161e+00, -3.8198e-01, -2.3017e-01, -5.8467e-01, -3.5604e-02,
         -3.9114e-02, -3.0936e-01,  1.2150e+00, -8.4436e-01, -1.2478e+00,
          5.6091e-01, -2.6313e-01, -2.7390e-01,  6.5159e-01, -1.2564e+00,
          1.0023e+00,  6.7826e-02, -1.3901e+00,  4.8017e-01, -1.7177e-01,
          1.1901e-01,  6.6517e-01,  8.3012e-01,  1.1645e+00,  1.3930e+00,
          7.4261e-01, -3.7814e-02,  7.7740e-01, -1.5404e+00,  4.3474e-01,
          1.0123e+00,  4.6408e-01]], device='cuda:0')}
{'spk_embedding': tensor([[-2.7514e-02,  1.1261e+00,  5.5841e-01,  1.1554e+00, -1.4768e+00,
          6.4574e-01,  8.7350e-01, -1.9143e-01, -1.3364e-01, -8.8304e-01,
          6.8537e-01,  4.3657e-01,  1.3226e+00, -2.7354e-01, -5.0915e-01,
          1.6072e-01, -1.7464e+00,  7.7550e-01, -8.5817e-01, -1.9341e-02,
         -1.5186e+00, -7.7464e-02,  1.5366e-01,  1.5422e+00,  3.5049e-01,
         -7.9827e-01,  4.8083e-01, -4.5937e-03, -8.0199e-01,  1.5080e+00,
         -2.1949e+00, -3.6382e-01,  1.5588e-01, -1.6743e+00, -1.2656e+00,
         -4.1569e-01, -5.4846e-01, -1.0400e+00, -6.9874e-02,  6.2443e-01,
          4.0864e-01, -5.0753e-01,  2.7794e-01,  9.2652e-01,  2.3766e-01,
          1.9581e+00, -3.2619e-01,  8.4147e-01,  3.3183e-01,  1.0091e+00,
          2.2862e-01,  1.5797e+00,  1.0374e-01, -3.5680e-01,  2.7965e-01,
          1.2092e-01,  6.2956e-01, -2.7444e-01, -1.1627e+00,  8.8098e-01,
         -5.3454e-01, -2.4151e-01,  8.7240e-01, -6.9140e-01, -7.6061e-01,
          1.3067e+00, -3.9895e-01,  1.2040e+00,  1.2649e+00,  2.8721e+00,
         -9.2992e-01, -3.5876e-01, -8.7203e-01,  2.0211e-01,  5.5605e-02,
         -9.8709e-01,  6.0764e-01,  5.5736e-01, -1.6029e+00,  2.2919e-01,
          4.7492e-01,  1.0839e+00,  1.1073e+00,  7.0862e-01,  2.1418e-03,
         -8.7193e-01, -1.6426e-02,  2.4173e-01,  8.5218e-01, -5.9665e-01,
          9.8587e-01,  2.5219e-01,  5.5474e-01, -4.9107e-02,  1.4255e+00,
         -8.3651e-01, -7.1446e-01, -1.5080e+00, -2.0367e-01, -9.6339e-01,
          2.1176e-01, -4.7981e-01,  4.5224e-01, -3.6631e-01,  4.7261e-01,
          1.7937e+00,  9.9674e-02,  7.9135e-01,  1.6991e+00, -9.8309e-01,
         -6.4238e-01,  2.3027e+00, -8.3260e-01, -8.7981e-02, -2.7149e-02,
         -7.1597e-01,  1.4575e+00,  4.4580e-01,  1.3920e+00,  4.8832e-02,
          3.7454e-01, -1.2451e+00, -1.1228e+00, -2.2184e+00, -1.3898e+00,
          2.4673e-01,  9.1907e-01, -8.8104e-01,  6.1713e-01, -5.7391e-02,
          9.7062e-01, -4.9618e-01, -1.2466e+00, -3.8462e-01,  1.8697e-03,
         -8.3686e-01, -1.2020e+00, -7.7435e-01, -1.3299e-01, -6.0682e-01,
          1.5708e+00,  2.8726e-01, -3.7564e-01,  1.9862e-01,  6.0458e-01,
          1.3790e+00, -3.9485e-01, -4.2943e-01, -1.7980e-01,  1.0117e+00,
         -2.1194e+00, -8.7259e-01,  1.7303e+00, -2.6986e-01,  1.2830e+00,
         -1.9565e+00, -1.1968e+00, -3.0146e-01, -4.4822e-01, -1.1815e+00,
          5.1121e-01, -3.8990e-01, -2.4577e-01, -9.4995e-01,  9.4058e-02,
         -6.7390e-04,  1.3361e+00,  9.8427e-01, -7.4307e-01, -1.0503e+00,
         -2.6243e-01,  1.5615e+00,  1.2226e+00, -3.8431e-01, -2.0331e-01,
          7.1395e-01,  2.3440e-01, -1.5474e+00,  4.2942e-01, -2.8634e-01,
         -1.2583e+00,  1.2354e-01, -1.5709e+00, -7.9544e-01,  1.3482e+00,
          6.8446e-01,  3.4728e-01,  2.8787e-01, -8.9656e-01, -4.5434e-01,
          9.1322e-01,  4.4513e-01]], device='cuda:0')}
 ...
```

有多少条音频就会输出多少个embedding。可以理解为这条音频的speaker标识符。

需要自己写一个相似度比对的算法，再加上strict系数来控制严格判定程度。

