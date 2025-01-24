![](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2917690888.webp)

这里什么都没有哦。<br>

之后可能会有。<br>

一直以来觉得迁移起来太困难了所以操作系统崩了也懒得救。现在把需要做的事情列一下。

- [x]迁移我的梯子。
- [x]找一个稍微舒服一点的输入法。
- [ ]添加 ssh-key 到 github 写入权限账户
- [x]vscode 以及基本插件
- [ ]typora 以及 picgo 联动
- [ ]miniconda 安装和 cpu 版本最小 torch 包。
- [ ]docker 以及守护进程代理配置
- [ ]paddle cpu 版本的编译。

为啥只弄 cpu 版本的 paddle，因为我台式的 deepin 分配的空间相当小，另外我 windows 那边可以编译 gpu 版本的不过耗时过长，说起来也正是从我编译 gpu 版本的 paddle 开始我的一个 PR 积极性就逐渐下降了，二次编译半小时非常磨人。<br>

如果可以的话，之后尽量避免涉及 gpu 编译，生命诚可贵，远离 gpu 编译。<br>

## 迁移我的梯子。

我一直用的都是 qv2ray。<br>

它支持直接迁移分享连接，也支持直接的订阅，功能相当齐全。而且 C++ qt 写的界面看上去很硬核而且速度比起我自己写的 python qt 不是一般的快。<br>

碰到的一个问题是，我发现网页无法使用代理，但是尝试配置了一下 http_proxy，终端 curl 起来相当顺畅丝滑毫无阻力。<br>

最初我以为是我的代理有问题，还特地新开了一个梯子测试了一下。发现那是多余的。<br>

导致这个的原因，是我以前非常喜欢用的一个插件，`谷歌学术助手`，在没有梯子的时候，如何翻墙，我的做法是，把这个插件打包，然后用开发者模式直接解压到插件里面，然后开一下 vip，就能自给自足，这个操作起来比起其他的需要订阅或者分享要简洁的多，我很多设备上比如树莓派就是直接选择开了谷歌学术助手而不是自己挂代理。<br>

但是这个学术助手的 vip 过期了，过期一年了，也是我太久没有打开我台式机的这个操作系统。<br>

过期之后，它在 google 浏览器里的优先级和地位甚至比我的系统代理还要高。<br>

导致我开 b 站欸能看，github 都上不去，google 搜任何东西都无响应。<br>

### solve: 八谷歌学术助手插件禁用掉。

## 一个舒服的输入法。

输入法的舒适度很重要，不仅在切中英文的时候会不一顿一顿的，而且，打字就像流水，或者说水流。<br>

deepin 的可选输入法不多。如果可以我尽量不选择自己配置。<br>

我记得在之前 linux 操作系统课上，有一节实验课的内容是给 ubuntu 装上中文输入法。<br>

我嘞个老天，我当时装了一节课没装上后来放弃了，我选择直接用星火商店找个能一键安装的。<br>

那个时候我体会到了，一个`apt install`一直循环报错有多难受。而且，我也在想，如果你能提供一个更现代化的安装方式就好了，比如 zsh。<br>

每个人可能装输入法的方式不一样，但我总选择应用商店安装，除非有一天有个输入法能自己提供一个详细的安装指南给我并且告诉我它适配哪些操作系统。<br>

这次用的是华宇拼音输入，开门直接让我选择配置自定义短语。<br>

但是自定义短语依然存在 bug。<br>

我在尝试输出`![](https://image.baidu.com/search/down?url`的时候，它总是输出`!`。<br>

可惜了。<br>

不过`<br>`倒是出了，也许我也可以考虑用 html 的写法取代 markdown 的写法。<br>

等这个系列安顿下来我可能会选择去学一下双拼。<br>

据说全拼的效率很低，至少我需要敲击更多下才能够达到一样的效果。<br>

小鹤音形是啥来着，突然想起来这个名词。<br>

## vscode 以及插件

其实我目前正在用 vscode 写。<br>

我想想，目前依然需要做的有：<br>

- 装上 prettier,然后写个保存格式化的配置文件。
- copilot 和 github 验证。

hello,copilot.

It's been a while since I last saw you.<br>

I'm back.(太浪漫啦！)<br>

## 添加 ssh-key 到 github 写入权限账户

我最初在还没添加ssh-key的时候，我克隆仓库用的是http_proxy。<br>

然后在尝试push的时候出现了这个:<br>

```shell
(base) xnne@xnne-PC:~/code/Blog$ git push origin master Username for 'https://github.com': MrXnneHang
Password for 'https://MrXnneHang@github.com': 
```

以前也出现过一次这样的问题，但那个是因为仓库不存在或者说是私有的。<br>

这次是因为https_proxy不走ssh协议。我目前是这么理解的，目前我正重新克隆并且打算push。<br>