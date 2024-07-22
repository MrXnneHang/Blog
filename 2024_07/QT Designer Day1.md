## 1.sipPyTypeDict() is deprecated

Solve:

[https://github.com/MrXnneHang/QT-Designer-Explore/blob/master/issuses.md](https://github.com/MrXnneHang/QT-Designer-Explore/blob/master/issuses.md)

Issue[1]

## 2. clicked.connect  误触发函数。

```python
# ✔
self.startdrawButton.clicked.connect(self.showDrawMainWindow)

# ❌
self.path_selector_1.clicked.connect(self.open_file_dialog(self.path_line_1))
```

正常传入函数地址，不会触发函数，只有点击时才会触发。

但是在传参进去后，会在执行绑定时触发。需要用lambda:

```python
self.path_selector_1.clicked.connect(lambda:self.open_file_dialog(self.path_line_1))
```



## 3. Path Select

### File:

```python
def open_file_dialog(self,line_edit):
    # 弹出文件对话框
    path = QFileDialog.getOpenFileName(self, "选择文件位置", "")
    # path-like:
    # ('D:/program/Designer/README.md', 'All Files (*)')
    if path:
        # 将选择的路径设置到LineEdit中
        line_edit.setText(path[0])  
```

Use

```python
line_edit.text()
```

来获取text。

### Directory:

Use:

```python
QFileDialog.getExistingDirectory
```



经过测试，都是只能单选，不能多选。



## 4.隐藏Layout下的所有项目。



```python
from PyQt5.QtWidgets import QVBoxLayout,QHBoxLayout

def hidden_layout(layout):
    """隐藏布局及其子控件

    layout: QVBoxLayout/QHBoxLayout ,可以传入列表

    hidden_layout(Layout1)
    hidden_layout([Layout1,Layout2])
    """
    if isinstance(layout, list):
        for i in range(layout):
            hidden_layout(i)
    for i in range(layout.count()):  # 遍历子控件
        item = layout.itemAt(i)
        if isinstance(item, QVBoxLayout) or isinstance(item, QHBoxLayout):  # 检查是否为QLayoutItem
            hidden_layout(item)
        else:
            widget = item.widget()  # 获取子控件
            widget.setVisible(False)  # 递归调用隐藏子布局
```



依然存在瑕疵，就是当我隐藏部分布局，和它同级的布局会因此而改变，我希望它保持不变。



## 5.Check Box



```python
class RSP_ChartSelector(QMainWindow,Ui_ChartSelector):
    def __init__(self):
        super().__init__()
        self.setupUi(self)

        # Connect the CheckBox signal to a slot
        self.line_chart.stateChanged.connect(self.on_checkbox_state_changed)
        self.bar_chart.stateChanged.connect(self.on_checkbox_state_changed)
        self.pie_chart.stateChanged.connect(self.on_checkbox_state_changed)
        # Set initial state of CheckBox
        self.bar_chart.setChecked(True)


    def on_checkbox_state_changed(self, state):
        if state == 0:
            print("CheckBox is unchecked")
        elif state == 2:
            print("CheckBox is checked")
```

上面适用于立即触发，我们到时候也有用，当勾选后立刻更新图片而非再次手动点击生成图片。

另外也可以通过这个方式来获取是否被选中:

```python
if self.line_chart.isChecked():
    print("Line Chart CheckBox is checked")
```

