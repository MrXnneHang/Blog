# yutto-uiya v1.0.0 Released!

这是一个简单的功能介绍，主要是说明怎么部署。<br>

如果你是一个成熟的`linux`或者`mac`用户，欢迎直接到 github 进行手动部署: [yutto-uiya](https://github.com/MrXnneHang/yutto-uiya)<br>

这里的部署只是针对为打包的 windows 一键包。<br>

## 功能介绍:

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261656090.png)<br>

![image-20241126171028821](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261710560.png)

目前支持正在播放视频列表，用户收藏夹，用户合集，番剧的下载。详细用法参见每个功能页面的说明。<br>

请在碰到问题时先阅读一遍常见问题页面:<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261656032.png)

也欢迎向我反馈问题。<br>

## 怎么运行它。<br>

### 1. SESSDATA 的获取:<br>

首先运行`1.获取SESSDATA.bat`:

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261656937.png)

在打开的`chrome`界面登陆你的`bilibili`账号。<br>

当你登陆完成后就可以关闭那个终端窗口了。<br>

再次运行`1.获取SESSDATA.bat`，这次看那个终端窗口:<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261656979.png)

复制你自己的`SESSDATA`,然后到`configs`文件夹下用你喜欢的文本编辑器(比如记事本)打开`args.yaml`文件，将它写入到双引号中。注意检查一遍，尤其是多余的空格。<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261656939.png)

如果你是大会员的话，你还应该同时把`vip_strict: false`改成`vip_strict: true`。避免有时候被当作普通用户拦截，当然当你的大会员过期了，你也应该回到这里把它改成`false`，否则它会拦着你不让你下载。<br>

下载默认会放在`downloads`文件夹下，你可以在`args.yaml`中修改这个路径。<br>

### 2. 运行 webui:<br>

双击`2.运行.bat`，然后等待即可。<br>

不出意外它会自己弹出来，如果没有的话你也可以自己手动复制这个网址然后到你的浏览器中打开。<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261656166.png)

详细的使用方法各位查看各个功能页面的说明即可，需要注意的是，每个板块我都有提供示例链接，如果你的网址对应的页面类型不同，可能会造成一些意想不到的结果。<br>

![image-20241126170349687](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411261703077.png)

之后，祝各位使用愉快！

## 更多待开发:

- [ ] 把音频，视频，弹幕作为勾选项，放在 webui 中选择。<br>

