- [ ] 如何给Qwidget背景对象设置圆角。
- [ ] 如何给窗口绘制阴影。
- [x] 解决 给Widget设置背景，子控件会受到影响的问题

## 1.QT Designer中的样式表

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