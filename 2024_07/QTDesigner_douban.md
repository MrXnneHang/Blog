## 为什么再用QT Designer

之前写的关于[PyQt使用总结](http://xnnehang.top/blog/58)

里面提到的yaml管理窗口位置和资源，诶感觉有点神奇哈。虽然我有完全的掌控感，这感觉真的很不错。

但是有个人说如果项目大的话尽量使用QT Designer。

> Ask:[一个较大的项目用PyQT要如何管理页面布局。](https://linux.do/t/topic/152217)
>
> Answer:肯定是用designer，代码是练手时候用的，习惯了以后用界面没事的

我依然相信，在快速开发的环境下，或者自己的项目，我会坚持使用yml来管理。

但是现在，毕竟项目不是我的，我也只是其中的一环，我服个软，我去学个QT Designer。但问题不大，上次主要的功夫都花在了如何安全地开多线程，以及绘制动画上。

（嘿嘿，我看了学长的QT项目，里面的多线程是不安全的，可能会影响程序进入下一个函数，和程序退出，以及Threading的进程级线程会和整个main.py竞争资源。如果不关，应用就会变慢。）

多线程得继承QThread，并且调用它类下的run()，以及写好CloseEvent才算安全。



## QT Designer安装。



```cmd
pip install PyQt5
pip install pyqt5-tools

designer
```



报错:

```cmd
'designer' 不是内部或外部命令，也不是可运行的程序
```



其实每次打开都用终端也挺麻烦的。

我这里直接定位了我虚拟环境里面的designer.exe

我的环境名为UIDes

path:

```cmd
D:\miniconda\envs\UIDes\Lib\site-packages\qt5_applications\Qt\bin\designer.exe
```

把它搞个快捷方式放到桌面，每次用的时候打开，看上去还是有点麻烦。

因为打开后需要open folder,open file。

我很烦那个。

于是，另一种解法，是在.ui的打开方式中，默认选择定位的designer.exe。

这样一劳永逸。