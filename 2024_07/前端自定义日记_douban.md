![](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p1933116042.webp)
## 关于前置（一些废话）：

说来曲折，我最初的想法是在windows上面直接预览前端页面的，但是为了能够看到加载博客后的实际效果，我走上了一条歪路:企图在虚拟机上面

运行和调试整个项目。

最初我盯上了已经有的环境，virtualbox和deepin20.9，在高兴地装完之后，发现每次开机都会进入installer界面，进入桌面每次都要手敲一段代码，很艰难。

随后迎面而来的是bt面板启动失败，python环境不全。我果断扔了deepin。随后转战ubuntu，结果在virtualbox上先后碰到了终端无法打开(ctrl+alt+t无效，手动点击终端也不行。)，和用户不在sudoer list中。又一次果断抛弃。

然后我投入vmware的怀抱，这一次非常顺利，终端顺利打开，代理工具都顺便装上了，bt面板没有任何报错，一切顺利得难以言说。但是最终我又走上了一条歪路：我折腾vmtools。在经过大半天的努力，我安装了官方自带的，开源的，中途干净卸载几次，重装系统几次，系统降到了12.04的时候，我突然发现虚拟机原来可以这么快。但是最后我兜兜转转还是只能共享剪切板文本而不能拖拽文件，现在想想还是感觉不甘心，但是折腾这个真没什么鸟用。

在我配置完所有:jdk,redis,mysql,nodejs,maven以及各种镜像之后。我即将接近了目标的时候，**我室友冷不丁地说了一句，你为什么不在服务器上面搞。**

我那一瞬间就像五雷轰顶，我发现我在服务器上面只要新开三个端口，然后像最初那样配置公网ip访问就可以。而且还不影响到我的域名下的网站，而且还可以直接共享数据库，连数据库的导入都省了。

于是，以上就是我曲折的经历。

最终,**在服务器上面新开端口来调整前端，适当的时候合并代码对我而言是最佳选择。**

那是我的最终答案。

## 一些修改:

### /2024/6/28:

* 删除Header下方两个wave波浪图片。

* 将原本的Header的三个Bg图换成css中全局替换的背景图x1。

打算做的修改:

* 如果可以，我想改一下字体，但是似乎字体好像分成好多种系统来分别显示？我有点不敢下手。
* 将博客面板调成半透明，但同时保持字体清晰。
* 自己设计一个Logo

**修改前:**

![Snipaste_2024-06-28_23-17-46](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2909971706.webp)

**修改后:**

![Snipaste_2024-06-28_23-16-38](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2909971707.webp)

最初我是因为三个伊蕾娜才果断说什么都要部署这个博客的，可以说死磕了好几天。但是我换掉伊蕾娜却这么果断。我太不是人。

我是参照着:[Echo小窝](https://www.liveout.cn/message/)  去改的。

但是我却觉得自己越改越像:[xyz's blog](https://xzynet.com.cn/)

可能不会天天改，想起来改一点。因为比起修改博客，维护博客同样重要，修改博客大概算是一种心灵调剂吧，有时候写博客写着写着会觉得同质化，我就想要做点不一样的改变一下。



### /2024/6/29:

* 给自己设计了一个Logo。

![white_bg](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910033270.webp)
> 什么时候搞丢了可以在github的Blog_img/2024_06下面找回。上传时间为2024/6/30/16/22

* 透明视窗改了半天发现还不如原来纯白色的好看，太花里胡哨。暂时放弃，字体的原来也越看越顺眼（给懒惰的自己找的借口。）



---



写于/2024/6/30,这大概不出意外是六月份的最后一次博客更新。

**想要做的:**

在home位置，Introuduction下方加上一个类似于XXX还剩下多久的那种倒计时，可以是横条的，也可以是圆形的。

比如，暑假还剩下X天结束。这个月还剩下X天结束。

> 今天是周X，顺便给自己写一句应景的话，像页脚那样子存在数据库里面随机或者固定出现。当然，如果办不到，就直接手动实现。

以及我不太想面对的，距离毕业还剩下X%。wtf，想想就抓狂，也许这样可以时时促进自己考研的心，也让自己学学英语。哦对了，今天是星期日哦，虽然明天要上课会让你很低落，但是请不要忘了学十五分钟英语。呃，这么说好像有点不应景。我好久没学英语了。





### 2024/7/6:

* live2d看板娘，虽然不知道我到底能不能带的动，但是，live2d看板娘啊。

参考：[为你的个人博客添加一个Live2D看板娘](https://www.bilibili.com/video/BV1TD421M7zm/?spm_id_from=333.999.0.0&vd_source=d7601f0fc447d708fff71aa75186ea10)

饭都喂嘴里了总不能不吃吧。

* 夜间模式和白间模式的区分，夜晚自动切换成暗色。





## 待实现汇总：（避免太忙时没实现就给忘了）

* Introuduction下方加上一个类似于XXX还剩下多久的那种倒计时：可以是横条的，也可以是圆形的，或者高级点的沙漏，但横条一般画风兼容度更高。
* live2d看板娘，虽然不知道我到底能不能实现，但是，live2d看板娘啊。
* 夜间模式和白间模式的区分，夜晚自动切换成暗色。