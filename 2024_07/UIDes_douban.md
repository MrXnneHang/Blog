# 大体思路和布局：

## 用气泡替换文本框。

1.气泡+tag

两个接口，一个数据分析，一个总结。

chapter-1-tags

接口一

[(一)作业基本情况,(二)安监督查情况],

接口二

[总结一下]

左栏放LLM选项，传统选项，以及其他LLM模型选项。

右侧下面并不是输入文本框，而是tags，直接点击即可。

左边是回复，右边是发送的tags。

回复不用一次性给出，可以一段一段给，发气泡。

并且在右边给出一个像纸飞机一样的东西，可以直达上一个tag所在位置。

![image-20240719093357102](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910806102.webp)

## 进入界面

![image-20240719093651656](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910806103.webp)

动态效果，音效。直接明了。

特地去下了个CSGO。绑定了下b5，直接抄应该不会被告吧=-=

每个chapter做一个模块。

## 进入后的布局：

![UI设计](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2910806104.webp)



# 任务拆分（难点合集）

## 1.点击模块进去后不新建Window但更新布局。【★★★★】

以及还需要做一个log out的选项，能够回退到主界面进行再次选择。

如何Hidden，如何做动画。

## 2.聊天式。【★★★★★】

* 气泡随着字符长度变化而变化

* 以及查看历史记录的滚轮。
* 保留历史记录。
* 自动更新窗口

## 3.纸飞机，快速回到定位的某一个段。【★★★】

## 4.保留上一次选项中的excel路径。【★★】

## 5.做一个路径选择框。【★】





先不做整合，先每个Problem都做一个window出来，并且能够处理简单响应。