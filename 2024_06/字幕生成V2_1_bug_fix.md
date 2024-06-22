# 2024/6/21的更新:(主要是Bug-fix)

## 1.~~部分用户转成.wav文件时是大写的.WAV，被认为不是支持的wav~~

![Snipaste_2024-06-21_16-45-52](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406211836154.jpeg)

现在WAV会被视作wav同等处理。

## 2.~~偶尔的吞字现象。~~

将for n in range(cut_points[-1][1],len(sentence["text"])-1):中的-1去掉后，这个现象就消失了，但有点不可思议吞字发生概率不高，而我担心这个去掉后会报出**list out of range** 或者**list out of index** 的错误，如果你们有碰到，可以把整个输出都发给我，我来想想。

#### 调整前:

![Snipaste_2024-06-21_17-05-15](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406211841554.jpeg)

#### 调整后:

![玩得就很开心](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406211841446.jpeg)



## 3.~~cut_line未引用导致调整cut_line无效~~

sentences_methods.py中的seg_sentences的未引用。

#### 这是调整前的:

![before_seg](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406211839309.jpeg)

### 这是调整后的:

![seg](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406211842245.jpeg)

![seg2](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406211842501.jpeg)

并且你可以通过调整cut_line和combine_line来控制输出长句或者短句。

## 4.你可以在config.yml中调整cut_line和combine_line

```
## 单位是毫秒

cut_line: 600  # 如果一个句子中，有两个字的间隔超过1000毫秒，以此把句子切成两句，防止长句过长
combine_line: 400 # 如果两个字的间隔小于400毫秒，就把它们合并到一句，可以增加连贯性，同时合并一些语气词。
```

如果你发现读取失败了，请确保:后面有一个空格，以及:是英文的:
