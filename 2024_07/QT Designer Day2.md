## 1.Tab:

![Snipaste_2024-07-21_20-30-36](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407212105380.jpeg)

不打开新窗口，但切换不同页面。

QT Designer自带Tab，拖进去即可。

这里贡献一份美化。

```css
QTabWidget::pane {
    border-top: 2px solid rgba(0, 0, 0, 1); /* 黑色 */
}

QTabBar::tab {
    font-family: "Microsoft YaHei";
    font-size: 12px;
    font-weight: 500;
    padding: 10px 25px; /* 增加内边距 */
    border-bottom: none;
	min-width: 45px;
   
}

QTabBar::tab:selected {
    color: rgba(123,190,254, 1); /* 淡蓝色 */
    border-bottom: 2px solid rgba(123,190,254, 1); /* 淡蓝色 */
}

QTabBar::tab:!selected {
    margin-bottom: 2px;
}

```

关于qss的写法。

现在看来和C++的类很像。每个对象的qss只能编辑自己，但之所以会有多个不同的{}，是因为一个对象有不同的子对象::，和不同的状态:。

常见状态,hover,selected。

### 还需要补充:

* 如果创建三个以上的Tab，默认只有两个。

## 2.用Button链接Tab。

RSP:

```python
class RSP_Tab(QMainWindow,Ui_Tab):
    def __init__(self)->None:
        super().__init__()
        self.setupUi(self)
        self.t1_switch_t2_button.clicked.connect(self.switch_to_tab2)
        self.t2_swicth_t1_button.clicked.connect(self.switch_to_tab1)
    
    def switch_to_tab1(self):
        util.switch_tab_index(self.TabWidget,1)
    def switch_to_tab2(self):
        util.switch_tab_index(self.TabWidget,2)
```

util:

```python
def switch_tab_index(TabWidget,index):
    TabWidget.setCurrentIndex(index-1)
```



### 值得注意的是，tab的index是从0开始的。

## 3.可以被触发的Label。

因为Label可以直接贴上贴图，甚至可以播放动画,gif。所以我当时死磕要把Label当成Button用。因为开始界面我想做点动态的。

ClickQLabel:

```python
from PyQt5.QtWidgets import QLabel
from PyQt5.QtCore import pyqtSignal, QPropertyAnimation, QRect, Qt



class ClickQLabel(QLabel):
    """可以被触发的Label
    
    @func:
    animate_geometry:缩放Label。
    
    """
    clicked = pyqtSignal()

    def __init__(self, parent=None):
        super().__init__(parent)
        self.initial_geometry = None  # 记录初始几何位置
        self.setAlignment(Qt.AlignCenter)  # 设置文本居中
        self.init_animation()



    def init_animation(self):
        self.anim = QPropertyAnimation(self, b"geometry")
        self.anim.setDuration(30)  # 动画持续时间30毫秒

    def enterEvent(self, event):
        if self.initial_geometry is None:
            self.initial_geometry = self.geometry()  # 记录初始几何位置
        self.animate_geometry(scale=1.12)
        super().enterEvent(event)

    def leaveEvent(self, event):
        self.animate_geometry(scale=(1))  # 恢复到原始大小
        super().leaveEvent(event)

    def mousePressEvent(self, event):
        if event.button() == Qt.LeftButton:
            self.clicked.emit()
        super().mousePressEvent(event)

    def animate_geometry(self, scale):
        if self.initial_geometry is None:
            return

        width = self.initial_geometry.width()
        height = self.initial_geometry.height()
        new_width = width * scale
        new_height = height * scale
        new_x = self.initial_geometry.x() - (new_width - width) / 2
        new_y = self.initial_geometry.y() - (new_height - height) / 2
        new_geometry = QRect(new_x, new_y, new_width, new_height)
        
        self.anim.stop()
        self.anim.setStartValue(self.geometry())
        self.anim.setEndValue(new_geometry)
        self.anim.start()


```

目前把代码缩减了，只剩下缩放动画，以及我依然屈服地使用了hover。可恶，因为qss读不进来。QT Designer生成QLabel，我将QLabel替换成ClickLabel。按理说它有SetStyleSheet，但是我读不到。

以及hover的一个毛病，它不是渐变色，看上去变色变得挺着急的。

但没事，逼急了，我直接做个动画，然后把动画拿去播放。关于那个阴影问题。

动画:
把图片饱和度调高一丢丢

把图片亮度调低一丢丢

把边框由黑色调成白色

变化时长0.1s.

### 有待解决:

* 每次缩放动画的时候字体都会抖一下，但增大缩放倍率，也不会抖得更厉害，幅度永远是一丢丢，但这一丢丢我看得很难受。

* 制作渐变色
* 制作动画



## 4.纸飞机，快速回到定位的某一个段。

```python
def scroll_to_last_self_message(self):
if self.self_message_indices:
    last_self_index = self.self_message_indices[-1]
    self.message_list_lswdg.scrollToItem(self.message_list_lswdg.item(last_self_index), QAbstractItemView.PositionAtTop)
    self.current_message_index = last_self_index

def locate_previous_message(self):
if self.self_message_indices:
    current_pos = self.self_message_indices.index(self.current_message_index) if self.current_message_index in self.self_message_indices else -1
    if current_pos > 0:
        previous_index = self.self_message_indices[current_pos - 1]
        self.message_list_lswdg.scrollToItem(self.message_list_lswdg.item(previous_index), QAbstractItemView.PositionAtTop)
        self.current_message_index = previous_index

def locate_next_message(self):
if self.self_message_indices:
    current_pos = self.self_message_indices.index(self.current_message_index) if self.current_message_index in self.self_message_indices else -1
    if current_pos < len(self.self_message_indices) - 1:
        next_index = self.self_message_indices[current_pos + 1]
        self.message_list_lswdg.scrollToItem(self.message_list_lswdg.item(next_index), QAbstractItemView.PositionAtTop)
        self.current_message_index = next_index
```

化身无情调试机器和gpt对话一个小时最终解决。  

效果还是相当惊艳的。

如果消息够长，它总是能够把自己的消息定位到最上层。

![Snipaste_2024-07-21_20-54-11](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407212105419.jpeg)

如果间隔不够长，也会把两条都定位出来，并且不会再重复点击两次才切换。

![Snipaste_2024-07-21_20-54-24](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407212105223.jpeg)

我可能那时候真有点累，午睡就睡了一小会儿。

不得不说，早上刚刚起床的时候，是觉得，20分钟太短了，经常偷偷不计时然后干个四五十分钟。

但下午的时候，特别是到了六点，差不多只要十分钟左右就感觉有点昏了。

所以说专注也是个体力活。跑步还是得跑的。又偷懒了两天。

## 5.编辑资源。

在QT Designer中特别贴心地把它放在很显眼的位置。

右下角资源浏览器->resource root -> 添加。

而且似乎这么做的话最终引用的图片都会生成一个.py文件里面会有png什么hard编码。

![Snipaste_2024-07-21_21-01-56](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407212105055.jpeg)



之后会整理一个QT Designer Explore,所有的待补充和待完善都会集中在那里。

以及这里又是一次忠告。当我们一个实现行不通的时候，一定记得换一个实现。不要徒增bug，徒耗时间。