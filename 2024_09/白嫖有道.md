## 这是什么？

这是一个白嫖有道词典来记录单词的方法。

![image-20240910175636535](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409101758406.png)

它会被记录在这里:

![image-20240910175724248](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409101758218.png)

![image-20240910175746053](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/09/202409101757352.png)

后续会尝试使用sqlite【MySQL的本地版本】，因为yaml可能会有数据完整性问题，而且当一个数据损坏了，所有数据读取都会报错。

## 原理:

利用google-chrome模仿用户搜索单词，然后爬取网页源代码，然后正则得到我们想要的部分。



## 用途：

我正打算给自己写一个网页端的英文原版小说阅读器。并且希望能够记录一下单词以便后续乱序复习。



## 环境部署：

你需要chrome、chrome-driver.需要注意版本对应。

在`config.yml`中填入你的exe路径。

更改`./webrowser_config.py`中的line 22,把路径改成你的User Data路径，如果不存在，可以尝试运行一下你的测试版chrome。



## 免责声明： 

本软件仅供合法用途。任何因使用本软件进行非法活动所导致的后果，作者概不负责。使用者须遵守所在国家/地区的法律规定。