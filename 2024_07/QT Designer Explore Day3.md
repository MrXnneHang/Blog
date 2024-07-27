## 1.高dpi导致窗口看起来异常。

表现，在1080p上面表现和QTDesigner中一样。

但是放在4k屏幕上面就挤成一坨。

Issue:

[https://linux.do/t/topic/156959](https://linux.do/t/topic/156959)

>QT Designer中预览窗口正常，控件也会自适应缩放，有用spacer和layout。
>最终在4k屏幕和1080p上面表现差异巨大。
>在1080p下和预览的一致。
>而4k下屏幕和控件缩成一团。
>用resize把main窗口缩放两倍后，控件排版正常了，但是所有字体都很小。大概也只有预览的一半大小。
>
>应该如何解决？字体几乎都是写在stylesheet里面的。

Solve:

[https://doc.qt.io/qt-6/highdpi.html](https://doc.qt.io/qt-6/highdpi.html)

具体解法:

创建app之前设置高dpi自适应，以及pixmap高Dpi自适应。

```python
QtWidgets.QApplication.setAttribute(Qt.AA_EnableHighDpiScaling, True)
QtWidgets.QApplication.setAttribute(Qt.AA_UseHighDpiPixmaps, True)
app = QtWidgets.QApplication(sys.argv)
```

我的env:

```cmd
PyQt5==5.15.4 
PyQt5-Qt5==5.15.2 
PyQt5-sip==12.12.1
pyqt5-tools==5.15.4.3.0.3
```



## 2.学会了弹性窗口和自定义布局。

### Layout

之前我一创建空项目（只有一个MainLayout）就会开始往里面拖Layout。

其实并不建议直接拖动Layout。

我现在是把Widget当作Layout用。因为Widget可以在Layout中自动填充，自适应缩放。并且Widget可以内置Layout。可以实现Widget嵌套Widget。

以前手动拖动Layout真的不是人干的事情。

具体操作可以参考:

[https://www.bilibili.com/video/BV1gT4y1e7Pj/?spm_id_from=333.1007.top_right_bar_window_history.content.click](https://www.bilibili.com/video/BV1gT4y1e7Pj/?spm_id_from=333.1007.top_right_bar_window_history.content.click)

大概看个三四分钟，就能明白，其他操作都是一样的。



### Spacer

Spacer

建议使用max和min。

我没弄明白expanding的意义是什么。

经常可以一个max一个min。

然后ctrl+r看看window顺便再缩放一下。能达到预期效果就行。

**碰到问题。**

QT Designer自动生成.py的时候,spacer的命名会被格式化，而并不是用我的命名，不清楚为什么。其他的Object基本上都能够正常。

这样子就存在一个问题。

比如我之前碰到在写聊天气泡的时候，

实现左边发送和右边发送，就是靠删掉一个spacer。但是由于spacer不存在，命名不同，就很难用生成的代码进行交互。

这里的解决依然是，用widget当spacer就好。



## 3.并不是所有的窗体都是指定死的。

比如说聊天气泡。随着发送得越多，每次发送会自动产生一个内容。

这种程序性生成也是很重要的。

而且很巧，只要留一个widget给它。

它可以自己在里面写layout。然后比如这次的气泡。就是一列下来。

## 在做好了几乎所有的准备后。

我在想明天一天能否完成功能汇总和美化？

这几天总觉得自己没做什么东西。因为都零零碎碎的。

