- [x] 如何给Qwidget背景对象设置圆角。
- [x] 如何给窗口绘制阴影。
- [x] 解决 给Widget设置背景，子控件会受到影响的问题

## 1.QT Designer中的样式表并不是只针对该对象。

我最初以为样式表是针对该对象的。
但是样式表针对这些：该对象的不同状态（:hover,:selected）,该对象的不同子组件(::tab),**以及，该对象的子对象。(#background)**又由于Qwidget可以支持一个套一个。

如果我在最外层的Qwidget写上

```css
background-image:url(:/bg/resource/draw/bg.jpg);
background-repeat:no-repeat;
background-position:center;
background-size: contain;
```

就会碰上下面所有对象都会是以这个背景图片为背景的现象。

正确的写法:

```css
QWidget#background{
background-image:url(:/bg/resource/draw/bg.jpg);
background-repeat:no-repeat;
background-position:center;
background-size: contain;}

/*实测你也可以用#background来指定，这个是变量名。*/
```

用类#示例名称来指定你要赋予的对象。

当然你也可以在这里赋予子对象属性。

当我这么写:

```css
QWidget#background{
background-image:url(:/bg/resource/draw/bg.jpg);
background-repeat:no-repeat;
background-position:center;
background-size: contain;}

#tab_1{
background-image:url(:/bg/resource/draw/bg.jpg);
background-repeat:no-repeat;
background-position:center;
background-size: contain;}

```

就会让tab_1和background具有相同属性。

但是要想让他生效，需要该类具有相同的属性，或者说它们是相同类，那么这种qss是可以继承的。

更多参考:
[https://blog.csdn.net/qq_56720262/article/details/131733284](https://blog.csdn.net/qq_56720262/article/details/131733284)



## 2.给窗口制作圆角。

有了上面那个Background的Widget作为背景，我们其实只需要把background image给做个圆角，然后隐藏窗口上面QMainWindow自带的最小化关闭那一栏隐藏了就行。

qss做倒角:

```css
QWidget#background{
background-image:url(:/bg/resource/draw/bg.jpg);
background-repeat:no-repeat;
background-position:center;
border-radius: 6px;}
```

是radius那一行，如果你喜欢圆润一点，可以设的更大，比如20px。

然后Python代码隐藏菜单栏。

```python
from PyQt5.QtCore import Qt

# 隐藏菜单栏
self.setWindowFlags(Qt.FramelessWindowHint)
self.setAttribute(Qt.WA_TranslucentBackground)

# self是QMainWindow对象。
```





## 3. 给窗口制作阴影（投影）。

```python

from PyQt5.QtWidgets import QGraphicsDropShadowEffect

# 创建阴影效果
shadow = QGraphicsDropShadowEffect()
shadow.setBlurRadius(20)  # 模糊半径
shadow.setXOffset(0)  # X方向的偏移
shadow.setYOffset(0)  # Y方向的偏移
shadow.setColor(QColor(0, 0, 0, 160))  # 阴影颜色 (黑色，160透明度)
# 将阴影效果应用到窗口(实际上是应用到背景图像的widget上)。
self.background.setGraphicsEffect(shadow)
```

这里的self.background是窗口最外层包含background的widget。qss等见#1.





## 写在最后。



似乎就算我一天看两部电影，几乎没怎么去想那个项目，最后解决的问题和做的进度和我一整天盯着也没差多少。

甚至可能更多一些。

我觉得更多是钻牛角尖。

比如这个窗口阴影的问题，我最初跟着别人做，他用ps制作了投影效果。我也跟着做，但是他做的是非弹性窗口，就是不会缩放的，不用管布局，只要使劲叠就行。但我要做窗口可以接受缩放就得考虑布局。

他用QLabel来做，我这里QLabel放下去整个布局就填满了，不存在叠加性。至少我没学会。布局里面一般我常用水平和垂直，不存在叠加。我先是考虑了半天叠加，无果，又试着用qwidget的stylesheet。但是有个问题。图片加载进来，只加载到了有rgb值的位置，投影部分实际是透明的，也就没能加载进来。结果加载进来的图片没有阴影。我大概在这里钻牛角尖钻了一个中午。

在我去看了一部电影后我打算换实现。

就是我不用画好的图片，而是用一张没有阴影没有圆角的纯背景，然后用qss做倒角，用QGraphicsDropShadowEffect来做阴影。可以说倒角和阴影合起来才花了我二十分钟的时间，还顺便学会了#1,我想不明白我为什么要花那么久时间掉在一个实现里面。换实现呗。

我觉得自己前几天也很多次陷入这种坑里面。这个时候换个事情做或许会比较好。

到这里我的QT即使不算专精，也是能够自由自定义了。

但我依然在想，难道不能像psd里面叠加图层那样叠加渲染吗？布局还是有点死板的。