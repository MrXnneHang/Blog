## 1.改用PySide2后pyuic生成的代码运行报错。（虚惊一场）

```cmd
Traceback (most recent call last):
  File "app.py", line 12, in <module>
    main_window = RSP_WeChat()
  File "D:\program\project\window\response\main.py", line 17, in __init__
    self.setupUi(self)
  File "D:\program\project\window\UI\main.py", line 18, in setupUi
    self.background = QtWidgets.QWidget(main)
TypeError: QWidget(parent: QWidget = None, flags: Union[Qt.WindowFlags, Qt.WindowType] = Qt.WindowFlags()): argument 1 has unexpected type 'RSP_WeChat'
```



似乎是在安装pyqt-tools的时候自动帮我装了PyQt5.

所以并没有报错No Module 'PyQt5'，反而报了一个奇怪的东西。

我debug半天，发现pull下来的版本是可以用的。但是生成一下就不能用了。我以为是我的.ui出了问题，特地把昨天最后一个版本拉过来，发现在PyQt5项目中生成可以用，在PySide2中就会出现上面那个错误。

我以为这是两个实现有点不同。但后面发现是个乌龙。

因为生成的文件里面默认import PyQt5。

这个报错的原因是混合使用Pyside2和PyQt5。

只要把PyQt5改成PySide2就可以了。



这里送上替换脚本,你可以自定义要替换哪些，我这里替换了PyQt5和我的资源文件的位置。

replace_pyqt5_with_pyside2.py

```python
import os

def replace_in_file_if_needed(file_path, old_string, new_string):
    with open(file_path, 'r', encoding='utf-8') as file:
        content = file.read()
    
    if old_string in content:
        new_content = content.replace(old_string, new_string)
        with open(file_path, 'w', encoding='utf-8') as file:
            file.write(new_content)
        print(f"Replaced in: {file_path} \n {old_string}--->{new_string}")
    else:
        # print(f"No '{old_string}' found in: {file_path}")
        pass

def replace_in_directory(directory, old_string, new_string, ignore_file):
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.endswith('.py') and file != ignore_file:
                file_path = os.path.join(root, file)
                try:
                    replace_in_file_if_needed(file_path, old_string, new_string)
                except Exception as e:
                    print(f"Failed to process {file_path} - {e}")

if __name__ == "__main__":
    ignore_file = os.path.basename(__file__)
    # 指定要替换的字符串
    old_string = "PyQt5"
    new_string = "PySide2"
    
    # 指定当前目录
    directory = "."
    
    # 执行替换操作
    replace_in_directory(directory, old_string, new_string,ignore_file)

    # 指定要替换的字符串
    old_string = "import resource_rc"
    new_string = "import window.resource_rc"
    
    # 指定当前目录
    directory = "."
    
    # 执行替换操作
    replace_in_directory(directory, old_string, new_string,ignore_file)
```

run_replace.bat:

```cmd
@echo off
D:\miniconda\python replace_pyqt5_with_pyside2.py
pause
```





## 2.连续高频启动QThread导致异常闪退。

之前解决过只要开启线程就会报错的原因，局部变量初始化Thread类导致线程开始后立刻就结束，但线程销毁了，线程还在跑就会报错:

```cmd
QThread: Destroyed while thread is still running
```

当时的解决方法，是给初始化线程处，把线程变成self属性，就变成了持久化变量。

```python
self.open_image_thread = None
def mouseDoubleClickEvent(self, event):
    if event.buttons() == Qt.LeftButton:  # 左键按下
            self.open_image_thread = OpenImageThread(self.image_path)
            self.open_image_thread.start()
```

而这次是频繁点击左键，导致线程反复初始化在同一个变量（不断销毁再赋值）。

就会发生线程销毁但线程还在跑的异常。

我们这么做。用is not None来判定，只有在没有线程在跑的时候我们才进行打开。

保证每次都只有一个线程再跑。

但其实这个也是有一点前提，我这里的thread是打开一张图片，执行完自动就销毁了，但如果是长耗时的工作，那么这么做就相当于加了个锁，当一个线程没执行完，另一个只能等。打不开的。

```python
self.open_image_thread = None
def mouseDoubleClickEvent(self, event):
    if event.buttons() == Qt.LeftButton:  # 左键按下
        if self.open_image_thread is None or not self.open_image_thread.isRunning():
            self.open_image_thread = OpenImageThread(self.image_path)
            self.open_image_thread.start()

def closeEvent(self, event):
    if self.open_image_thread is not None and self.open_image_thread.isRunning():
        self.open_image_thread.quit()
        self.open_image_thread.wait()
    super().closeEvent(event)
```



## 3.获取Qwidget的布局并且修改。

```python
# layout.addWidget(self.avatar, 0, Qt.AlignTop | Qt.AlignLeft)
self.avatar_layout.layout().removeWidget(self.avatar_image)
self.avatar_layout.layout().removeWidget(self.avatar_name_label)
# 删除子对象
self.avatar_image.deleteLater()
self.avatar_name_label.deleteLater()
```

这里的avatar_layout实际上是一个Qwidget，而只有Layout支持removeWidget。

但是在QT Designer中我都是把Qwidget当成Layout来用的。

而很贴心，只要用.layout()就能获取布局，然后移除，最后删除。

安全的做法是先remove在delete later。



## 4.Scroll滑动到底部不生效

```python
def set_scroll_bar_last(self):
    self.scrollArea.verticalScrollBar().setValue(
        self.scrollArea.verticalScrollBar().maximum()
    )

def set_scroll_bar_value(self, val):
    self.verticalScrollBar().setValue(val)

def verticalScrollBar(self):
    return self.scrollArea.verticalScrollBar()
```

这么写，是没错的。

但是，调用的时候可能会出错。

```python
if type_ == "text":
    bubble_message = BubbleMessage(reply, Type=MessageType.Text, is_send=False)
else:
    bubble_message = BubbleMessage(reply, Type=MessageType.Image, is_send=False)
self.message_widget.w1.add_message_item(bubble_message)
self.message_widget.scroll_to_last()
```

我在add完后立刻调用，想要起到发送消息后自动刷新窗口的效果，但是实际上没起作用。

这是因为我立刻调用的时候，窗口信息可能还没更新。

用Timer加一个延时，就可以了。

```python
def scroll_to_last(self):
    QTimer.singleShot(100, self.w1.set_scroll_bar_last)  # 设置定时器，100毫秒后调用
```

