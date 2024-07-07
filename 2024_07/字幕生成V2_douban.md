# 字幕生成V2.2:Bug  Fix,手动修改device，可选择文本标点恢复。

## 1.List out of index:（究极Boss，终于被我逮到了）

今天使用的时候也恰好碰到这个bug了，就修了一下。

![list_index_out_of_range](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2910312204.webp)

如果你们使用过程中再有碰到，就这么做:

### 如果你们碰到应该怎么做：

先将time_stamp.py中write_long_txt的debug设置成=True,默认时=False。

![再碰到怎么做](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2910312205.webp)

然后把这个截图发给我。

![Snipaste_2024-07-07_07-44-53](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2910312206.webp)

可以看到检测到了597个字符单元（一个汉字或者一个单词），但是时间序列长度只有591，所以会报错。

**这一次的原因是:you'll,can't,she'd，因为我的代码会被拆分成you ll, can t , she d。而把所有英文单词缩写都当成了两个单词**，我修改了一些判断逻辑，把缩写当成一个单词就没再报错。

当你碰到识别到一些奇怪的符号的时候，加到config.yml的标点list中就可以。

![Snipaste_2024-07-07_07-45-45](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910312207.webp)

最终确保识别到的word和时间序列长度一样即可。

![确保长度一样长，就不会报错。](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910312232.webp)



## 2.通过Config更改device。

cpu写cpu,用显卡则写cuda，只支持NVIDIA的显卡。

![cuda](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2910312234.webp)

## 3.在写入srt字幕文件的时候顺便写入only_text的文件。

有人提到会用这种来进行一个视频转文档，所以就顺便写了，不会额外增加识别时长。

![Snipaste_2024-07-07_10-14-35](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910312238.webp)