# PyQt的使用总结。

有了之前[期末设计-人脸识别系统](http://xnnehang.top/blog/55)的经验，其实只要改改内容就可以用了。

但是毕竟那不是我写的，我也得熟悉一下窗口初始化的一个流程。所以这边就用之前做的没有UI的番茄钟来练练手。最后也顺便学一下python的可执行文件打包。（我终于要有自己的release啦）。

但其实说是自己写，其实都是参考。严格地说就是缝，使劲缝。

## #窗口、控件关系和Pos的管理：

这里我用的是yaml。我存了一个yasumi_config.yml。

### yaml 列表的写法。

```yaml
# ❌
main_window_size: (600,600)
main_window_size: [600,600]

# ✔
main_window_size:
- 600
- 600
- pretty good
# 第一种写法默认解析成字符串。第二种才会被当作一个list。可以混合int和str，和列表差不多。
```

### yaml 写字典：

```yaml
yasumi_clock:
  main_window:
    window_pos:
    - 100
    - 100
    - 700
    - 450
    start_draw:
    - 277
    - 194
    - 166
    - 49
    start_fanqie:
    - 281
    - 333
    - 167
    - 50
```

之后我们window之间的关系都会反映在这里面。所以缩进很重要，也让自己看得清晰。上一次全都是写在源码里面实在是难以忍受。



## #窗口的响应和界面代码分离。

有点模拟前后端分离的写法。因为如果控件多的话，真的混在一起会乱。

做法就是，**响应的class继承界面的class。**python的继承很好写，简单理解就是子类拥有父类所有的变量和函数。

这里主要是为了增加代码易读性和易改性，不时兴套娃，各种库里面的类嵌套看得我是很头疼的。

这里举个例子,

### 分离前:

```python
class Main_Window(QtWidgets.QWidget):
    def __init__(self):
        super().__init__()
        self.config = load_config("./yasumi_config.yml")
        self.main_window = self.config["yasumi_clock"]["main_window"]
        self.main_window_pos = self.main_window["window_pos"]
        self.draw_button_pos = self.main_window["start_draw"]
        self.start_fanqie_pos = self.main_window["start_fanqie"]
        self.initUI()
    def initUI(self):
        self.setGeometry(self.main_window_pos[0],
                         self.main_window_pos[1],
                         self.main_window_pos[2],
                         self.main_window_pos[3],)
        self.setWindowTitle('Main Window')

        self.startdrawButton = PrimaryPushButton('draw_main_window', self)
        self.startdrawButton.setGeometry(self.draw_button_pos[0],
                                         self.draw_button_pos[1],
                                         self.draw_button_pos[2],
                                         self.draw_button_pos[3])
        self.startFanqieButton = PrimaryPushButton('Start', self)
        self.startFanqieButton.setGeometry(self.start_fanqie_pos[0],
                                           self.start_fanqie_pos[1],
                                           self.start_fanqie_pos[2],
                                           self.start_fanqie_pos[3])


        self.startdrawButton.clicked.connect(self.showDrawMainWindow)

    def showDrawMainWindow(self):
        child_window_pos = self.list_all_button_pos()
        self.selectionWindow = ManualSelectionWindow(self.main_window_pos,child_window_pos)
        self.selectionWindow.show()
    def list_all_button_pos(self):
        return [self.draw_button_pos,self.start_fanqie_pos]
```

可以看到Button初始化，设置位置和click是混合在一起的。

### 我们对它进行分离:

```python
# 没有响应的界面
class Main_Window_UI(QtWidgets.QWidget):
    def __init__(self):
        super().__init__()
        self.config = load_config("./yasumi_config.yml")
        self.main_window = self.config["yasumi_clock"]["main_window"]
        self.main_window_pos = self.main_window["window_pos"]
        self.draw_button_pos = self.main_window["start_draw"]
        self.start_fanqie_pos = self.main_window["start_fanqie"]
        self.initUI()
    def initUI(self):
        self.setWindowTitle('Main Window')
        self.setGeometry(self.main_window_pos[0],
                         self.main_window_pos[1],
                         self.main_window_pos[2],
                         self.main_window_pos[3],)
        
        self.startdrawButton = PrimaryPushButton('draw_main_window', self)
        self.startdrawButton.setGeometry(self.draw_button_pos[0],
                                         self.draw_button_pos[1],
                                         self.draw_button_pos[2],
                                         self.draw_button_pos[3])
        self.startFanqieButton = PrimaryPushButton('Start', self)
        self.startFanqieButton.setGeometry(self.start_fanqie_pos[0],
                                           self.start_fanqie_pos[1],
                                           self.start_fanqie_pos[2],
                                           self.start_fanqie_pos[3])

# 单纯的响应        
class Main_Window_Response(Main_Window_UI):
    def __init__(self):
        super().__init__()
        self.startdrawButton.clicked.connect(self.showDrawMainWindow)

    def showDrawMainWindow(self):
        child_window_pos = self.list_all_button_pos()
        self.selectionWindow = ManualSelectionWindow(self.main_window_pos,child_window_pos)
        self.selectionWindow.show()
    def list_all_button_pos(self):
        return [self.draw_button_pos,self.start_fanqie_pos]
    
# 调用时选择response
if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    mainWindow = Main_Window_Response()
    mainWindow.show()
    sys.exit(app.exec_())

```

相信我，在你的button超过十个之后，你会觉得，这是个好主意。



## #自己写draw_window的class。

在确定button的位置和大小的时候，如果直接更改x,y,w,h相当麻烦，需要反复确认。而当项目大一些运行一次PyQt启动就要四五秒。这能忍？于是手动写了一个可以画方框然后返回坐标的类，可以适用于主窗口和任意的子窗口。

### 为什么不用插件

PyQt是有插件的，支持手动画控件，然后最后直接生成一份UI的代码。这样很不好，因为不方便调试，我习惯写一部分，确认没有bug了再写下一部分。最终几乎不需要调试，那么生成最终调试的时间估计不会比我写代码短。

但是我看过我室友生成的那份代码，只能说相当炸裂，能自己写的还是不要自动生成了。而且插件很臃肿，安装是第一个关卡，摸索功能又是第二个关卡。有时间折腾，都自己实现出来了。

而我只需要，一个画框，然后返回x,y,w,h。

### 代码：

```python
from PyQt5 import QtCore, QtGui, QtWidgets
from qfluentwidgets import PrimaryPushButton
class ManualSelectionWindow(QtWidgets.QWidget):
    def __init__(self, select_window_pos,child_button_pos):
        super().__init__()
        
        self.pos = select_window_pos
        self.button_pos = child_button_pos
        self.isSelecting = False
        self.startPos = None
        self.initUI()
        
        
    
    def initUI(self):
        self.setGeometry(self.pos[0]+600,
                         self.pos[1]+150,
                         self.pos[2],
                         self.pos[3])
        self.setWindowTitle('Manual Selection Tool')
        self.setStyleSheet("background-color: white;")

        self.selectionLabel = QtWidgets.QLabel(self)
        self.selectionLabel.setGeometry(QtCore.QRect(0, 0, 0, 0))
        self.selectionLabel.setStyleSheet("border: 2px dashed red;")
        for pos in self.button_pos:
            self.selectButton = PrimaryPushButton(self)
            self.selectButton.setGeometry(pos[0],pos[1],pos[2],pos[3])


        self.setMouseTracking(True)
    

    def mousePressEvent(self, event):
        self.isSelecting = True
        self.startPos = event.pos()
        self.selectionLabel.setGeometry(QtCore.QRect(self.startPos, QtCore.QSize()))

    def mouseMoveEvent(self, event):
        if self.isSelecting:
            rect = QtCore.QRect(self.startPos, event.pos()).normalized()
            self.selectionLabel.setGeometry(rect)

    def mouseReleaseEvent(self, event):
        if self.isSelecting:
            self.isSelecting = False
            endPos = event.pos()
            rect = QtCore.QRect(self.startPos, endPos).normalized()
            self.selectionLabel.setGeometry(rect)
            print(f"Selected Rectangle: {rect.x()}, {rect.y()}, {rect.width()}, {rect.height()}")

```

这份代码我就没有手动去分离UI和response。

因为UI是根据传入的Pos自动生成的。

### 效果：

main window 中，单击draw_main_window就可以。

![main_window](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910589160.webp)

之后会跳出一个和main_window布局一样，但是支持手动画方框的窗口。

![draw_rec](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910589162.webp)

画出的方框会显示x,y,w,h在下方。

### 关于调用：

```python
# 连接到button
self.startdrawButton.clicked.connect(self.showDrawMainWindow)


def showDrawMainWindow(self):
    child_window_pos = self.list_main_button_pos()
    self.selectionWindow = ManualSelectionWindow(self.main_window_pos,child_window_pos)
    self.selectionWindow.show()
def list_main_button_pos(self):
    return [self.draw_button_pos,self.start_fanqie_pos]

```

布局会显示多少取决于传入的child_window_pos。



## #控件



### 0.Window 初始化和调用

```python
class Main_Window_UI(QtWidgets.QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()
    def initUI(self):
        ...
        
if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    mainWindow = Main_Window_Response()
    mainWindow.show()
    sys.exit(app.exec_())
```



### 1.设置Window Pos&Size&Tittle

```python
self.setWindowTitle('Main Window')
self.setGeometry(self.main_window_pos[0],
                 self.main_window_pos[1],
                 self.main_window_pos[2],
                 self.main_window_pos[3],)
```

### 2.Button(触发器,按钮)

```python
from PyQt5.QtWidgets import QPushButton
from qfluentwidgets import PrimaryPushButton

# QtWidgets自带的
self.startdrawButton = QPushButton('Draw Main Window', self)
self.startdrawButton.setGeometry(50, 50, 200, 50)

# qfluentwidgets提供的
self.startFanqieButton = PrimaryPushButton('Start', self)
self.startFanqieButton.setGeometry(self.start_fanqie_pos[0],
                                   self.start_fanqie_pos[1],
                                   self.start_fanqie_pos[2],
                                   self.start_fanqie_pos[3])

# 连接触发函数，注意不要传入()，只是传入地址:
self.startdrawButton.clicked.connect(self.showDrawMainWindow)
```

![有美化和无美化](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910589163.webp)

白色的是自带的，青色的是引入的。

主要是字体也变了。原本的字体似乎是windows自带的字体。我也忍不住跟着吐槽一句，比尔盖茨什么都好，就是没审美=-=。那个字体用来做代码看得是不错，但是用来当button，真的看不清，而且毛刺感很强，很扎眼睛，在纯白的背景下。放一组近距离对比：

![soft](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910589167.webp)![hard](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2910589165.webp)

UI这东西，感官体验上区别就是这么一点点积累起来的。

### 3.创建窗口并且用按钮打开。

其实PyQt中没有父子关系。可以理解为，就是另一个窗口被当前主窗口触发了。因此你也会发现关掉主窗口并不会让子窗口自动关闭，而是需要关掉所有窗口才会退出程序。

可能也存在真正的父子窗口，但这里我没用到。

```python
self.startdrawButton.clicked.connect(self.showDrawMainWindow)

def showDrawMainWindow(self):
    child_window_pos = self.list_main_button_pos()
    self.selectionWindow = ManualSelectionWindow(self.main_window_pos,child_window_pos)
    self.selectionWindow.show()
```

以这个为例，我的ManualSelectionWindow也是继承于QtWidgets.QWidget。

也同样就是初始化类，然后调用show()就可以了。

>
>
>我目前发现的一个父子关系，在于Button和Label创建时传入的self,这个会将该Button和self建立父子关系绑定在一起。
>
>如果传入其他Window的实例，则会和其他Window绑定在一起。一般用self。

### 4.Draw Picture in Window。

我先调了一下布局，让出了一些空间给贴图。如果config.yml用得好，那么改布局只需要改config就行了，很方便。

![调了一下布局](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910589168.webp)

确定了一下贴图的Pos和Size:

![确定要贴图的区域](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910589169.webp)

大概7:6，那我就截一张7:6的图：

![选一张差不多比例的图.](https://image.baidu.com/search/down?url=https://img2.doubanio.com/view/photo/l/public/p2910589171.webp)

这里我创了一个src.yml。

```yaml
example: "./src/img/example.jpeg"
```

Init UI中创建Label Object(和按钮们排在一起):

```python
self.animation_label = QtWidgets.QLabel(self)
self.animation_label.setGeometry(QtCore.QRect(self.animation_pos[0],
                                              self.animation_pos[1],
                                              self.animation_pos[2],
                                              self.animation_pos[3]))
self.animation_label.setText("")
self.animation_label.setObjectName("Animation")

# 在Init_UI中调用贴图函数
self.Draw_Image(Label=self.Animation_Label,
                path=self.example_img_path,
                Pos=self.animation_pos)
```

然后单独写一个贴图函数:

```python
def Draw_Image(self,Label,Pos,path=None,frame=None):

    img = Image.open(path)
    img = img.resize((Pos[2],Pos[3]),Image.BILINEAR)
    img = np.array(img)
    rgb_image = img

    h, w, ch = rgb_image.shape
    bytes_per_line = ch * w
    q_img = QImage(rgb_image.data, w, h, bytes_per_line, QImage.Format_RGB888)
    pixmap = QPixmap.fromImage(q_img)
    Label.setPixmap(pixmap)
```

这样就有了：

![这样就有了](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910589172.webp)

#### 进阶用法：

但其实它可以用来画动画，画摄像头的帧等等。

这里演示画动画。

但在这之前，我们插入一个Qt多线程工作的部分。当一个按钮耗时很长时，如果没有独立线程，就会出现这个按钮按下去后，窗口就无响应了，不接受任何消息，拖动，关闭，都不接受。而且要画动画的话是一个持续的过程，如果不独立线程就会把主线程塞住，while true里面出不来。

如果你用过Python自带的thread.Threading，你可能可以实现你想要的功能。但相信我它只会坑你，是个大坑。自带的Threading线程是进程级的，和你开的main window一个级别，当你关掉main window,你会发现你的程序依然没有退出。因为仍然卡在threading中，这会埋下很大隐患。

#### 安全地添加多线程：

这里介绍一种比较安全的Threading方法。

```python
import sys
from PyQt5.QtCore import QThread, pyqtSignal, QTimer
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QLabel
from time import sleep

# 继承自QThread的自定义线程类
class MainWindowWorkerThread(QThread):
    # 自定义信号，用于向主线程发送信息
    update_signal = pyqtSignal(str)

    def __init__(self):
        super().__init__()

    def run(self):
        # 模拟耗时操作
        for i in range(1, 11):
            sleep(1)  # 模拟耗时操作
            self.update_signal.emit(f"Task Progress: {i * 10}%")
        self.update_signal.emit("Task Complete")

class MainWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle('QThread Example')
        self.setGeometry(100, 100, 400, 200)

        self.label = QLabel("Task Progress: 0%", self)
        self.label.setGeometry(50, 50, 300, 30)

        self.startButton = QPushButton("Start Task", self)
        self.startButton.setGeometry(50, 100, 100, 30)
        self.startButton.clicked.connect(self.start_task)

        self.thread = None

    def start_task(self):
        if not self.thread or not self.thread.isRunning():
            self.thread = MainWindowWorkerThread()
            self.thread.update_signal.connect(self.update_progress)
            self.thread.start()

    def update_progress(self, msg):
        self.label.setText(msg)


if __name__ == '__main__':
    app = QApplication(sys.argv)
    mainWindow = MainWindow()
    mainWindow.show()
    sys.exit(app.exec_())
```



我模拟了一个耗时十秒的工作，如果不用多线程，你将在按下按钮后十秒内无法对窗口执行任何操作。

而上面是安全地添加线程的方法。我上次改Qt的时候，很大的一个不稳定因素就是来源于python自带的thread,它甚至会和一些button争抢资源，导致一个窗口刚刚打开就闪退。我之前一直不知道是这个原因。

#### 用多线程更新帧

那么我们学着做，开一个线程给Animation画图。

加入这些:

class Main_Window_UI(QtWidgets.QWidget):

```python
def __init__(self):
        super().__init__()
        self.animation_thread = None
        ...
        self.initUI()
def initUI():
    ...
    self.start_drawgif_task()
    
def start_drawgif_task(self):
    if not self.animation_thread or not self.animation_thread.isRunning():
        self.animation_thread = DrawAnimationThread()
        self.animation_thread.setup(path=self.example_gif_path,
                                          label=self.animation_label,
                                          pos=self.animation_pos,
                                          frame_speed=12)
        self.animation_thread.start()
```

Thread class:

```python
import sys
from PyQt5.QtCore import QThread, pyqtSignal, QTimer
from PyQt5.QtGui import QPixmap, QImage,QIcon
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QLabel
from time import sleep
import numpy as np
from util import split_gif_to_frames
from PIL import Image

# 继承自QThread的自定义线程类
class DrawAnimationThread(QThread):
    update_signal = pyqtSignal(np.ndarray)
    def __init__(self):
        super().__init__()
    
    def setup(self,path, label, pos, frame_speed=24):
        self.path = path
        self.label = label
        self.pos = pos
        self.frame_speed = frame_speed
    def run(self):
        self.running = True
        frames = split_gif_to_frames(self.path)
        while self.running:
            for frame in frames:
                if not self.running:
                    return
                rgb_image = frame.resize((self.pos[2],self.pos[3]),Image.BILINEAR)
                rgb_image = np.array(rgb_image)
                h, w, ch = rgb_image.shape
                bytes_per_line = ch * w
                q_img = QImage(rgb_image.data, w, h, bytes_per_line, QImage.Format_RGB888)
                pixmap = QPixmap.fromImage(q_img)
                self.label.setPixmap(pixmap)
                sleep(1 / self.frame_speed)

```

这样子可以复用多次，如果有多个地方需要动画的话，就初始化多个类实例即可。

最终会像这样，因为我这里不支持gif，我截了两帧:

![animation](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910589173.webp)

![animation2](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2910589176.webp)

好家伙，找不同。其实PyQt可以解析gif的图像，但是，学会多线程是有必要的，建议为所有耗时较久的控件都加上独立线程，而这个是一个很好的例子。另外，请避免使用thread.Threading。用Qthread中的run方法替代。

因为小问题可能会在某个瞬间爆发然后杀了你，与其到时候再修修补补，不如现在就规范地使用。



### 5.设置icon图标。

```python
if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    mainWindow = Main_Window_Response()
    mainWindow.setWindowIcon(QIcon(mainWindow.src_config["icon"]))
    mainWindow.show()
    sys.exit(app.exec_())
```

只需要在类示例那边加一个设置icon的方法即可，传入的是路径。一般只要是图片都支持。

![icon](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2910589177.webp)

### 6.Set text:

之前那个太丑了，之后用到了找一个美化过的。

## #一个想法：

之前在开启项目的时候要挺久，需要四五秒到五六秒，比如3dsmax开启很久，就会被人喷说它打开项目的时间都够blender做个模型了。

我对加载也是很没有耐心的。

但是我们如果开大的开不起来，可以先开小的。

就是，在打开我们主程序窗口之前，先开一个加载窗口，里面放上一些有意思的动画，或者加载动画。让人觉得时间没那么久。

我们现在就来试一下。

![加载动画](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910589178.webp)

### LoadingWindow:

```python
import sys
from PyQt5 import QtCore, QtWidgets
from PyQt5.QtWidgets import QApplication, QDialog, QLabel, QVBoxLayout, QMainWindow

from PyQt5.QtCore import Qt
from util import load_config
from MainWindowThread import DrawAnimationThread
# 定义加载窗口类
class LoadingWindow(QDialog):
    def __init__(self):
        super().__init__()
        self.windowconfig = load_config("./yasumi_config.yml")
        self.src_conifg = load_config("./src.yml")
        self.LoadingWindow = self.windowconfig["yasumi_clock"]["LoadingWindow"]

        self.animation_thread = None
        self.initUI()
    def initUI(self):
        self.setWindowTitle("Loading...")
        self.setWindowFlags(Qt.FramelessWindowHint | Qt.Dialog)
        self.setFixedSize(self.LoadingWindow["window_pos"][0],
                          self.LoadingWindow["window_pos"][1])  # 固定窗口大小
        self.setStyleSheet("background-color: white;")  # 设置背景色为白色
        
        # 隐藏最小化，关闭等按键
        self.setWindowFlags(Qt.FramelessWindowHint | Qt.WindowMinimizeButtonHint)  # 设置窗口标志位
        self.animation_label = QLabel(self)
        self.animation_label.setGeometry(QtCore.QRect(self.LoadingWindow["animation"][0],
                                                      self.LoadingWindow["animation"][1],
                                                      self.LoadingWindow["animation"][2],
                                                      self.LoadingWindow["animation"][3]))
        self.start_drawgif_task()


    def start_drawgif_task(self):
        if not self.animation_thread or not self.animation_thread.isRunning():
            self.animation_thread = DrawAnimationThread()
            self.animation_thread.setup(path=self.src_conifg["loading"],
                                              label=self.animation_label,
                                              pos=self.LoadingWindow["animation"],
                                              frame_speed=24)
            self.animation_thread.start()

```



### 重写Show，在打开主窗口后关闭Loading窗口

```python
class Main_Window_Response(Main_Window_UI):
    def __init__(self,loading_window):
        super().__init__()
        self.startdrawButton.clicked.connect(self.showDrawMainWindow)
        self.loadingwindow = loading_window
        

    def showDrawMainWindow(self):
        child_window_pos = self.list_main_button_pos()
        self.selectionWindow = ManualSelectionWindow(self.main_window_pos,child_window_pos)
        self.selectionWindow.show()
    def Show(self):
        self.show()
        self.loadingwindow.close()

    def list_main_button_pos(self):
        return [self.draw_button_pos,self.start_fanqie_pos]
```

### 加入延时，因为目前主窗口打开得太快了。

```python
if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    
    loading_window = LoadingWindow()
    loading_window.show()

    mainWindow = Main_Window_Response(loading_window)
    mainWindow.setWindowIcon(QIcon(mainWindow.src_config["icon"]))

    timer = QtCore.QTimer()
    timer.singleShot(3000, mainWindow.Show)  # Delay mainWindow's show by 1 second
    

    sys.exit(app.exec_())
```





所有代码已经传到github:

[MrXnneHang/Yasumi-Clock](https://github.com/MrXnneHang/Yasumi-Clock)

这一次算是总结了上次用到的所有PyQt的功能，并且更正了不安全的地方。明天可能会探索一下我要写番茄钟需要的功能，比如按时间缩减的沙漏。但其实根据今天的经验，只需要先做个动画，然后再多线程画上去就可以了。不过得探索一下怎么设置alpha通道透明背景。不然颜色不一样太扎眼了。
