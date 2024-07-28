## 加载自定义字体并应用到指定widget上。

```python
from PySide2.QtWidgets import QApplication
from PySide2.QtGui import QFontDatabase, QFont

def load_custom_font(font_path):
    font_id = QFontDatabase.addApplicationFont(font_path)
    if font_id == -1:
        print("字体加载失败")
        return None
    font_family = QFontDatabase.applicationFontFamilies(font_id)[0]
    print(f"字体 {font_family} 加载成功")
    return font_family

def set_global_font(family, weight=400):
    """设置全局字体家族和权重"""
    font = QFont(family)
    font.setWeight(weight)
    QApplication.setFont(font)

def set_font_family(widgets, family):
    """设置字体家族"""
    for widget in widgets:
        font = widget.font()
        font.setFamily(family)
        widget.setFont(font)

def set_font_weight(widgets, weight):
    """设置字体权重"""
    for widget in widgets:
        font = widget.font()
        font.setWeight(weight)
        widget.setFont(font)

def set_font(widgets:list,font_path:str,weight:int):
    font_family = load_custom_font(font_path=font_path)
    set_font_family(widgets=widgets,family=font_family)
    set_font_weight(widgets=widgets,weight=weight)
```



调用:

```python
from font_util import set_font

set_font([self.global_tab],"./window/resource/fonts/ZCOOLXiaoWei-Regular.ttf",800)
```

有时权重过大或者过小都会不生效，这取决于下载的字体是否支持，一般都在100~1000

之前加载不生效，是因为QFont不支持直接输入字符串。虽然微软雅黑，Arial这些自带的可以。但是自己的字体需要通过load_custom_font转换成family。

还有一些时候加载不生效，应该考虑注释所有qss中写的字体类型，qss的优先级似乎比代码高，很诡异。但qss又不支持加载外部字体，就更诡异。



## 全局设置字体。

```python
def set_global_font(family, weight=400):
    """设置全局字体家族和权重"""
    font = QFont(family)
    font.setWeight(weight)
    QApplication.setFont(font)
```

同样，不生效应该考虑注释掉qss里面指定的font-family。

调用的位置位于app创建后，mainwindow创建前:

```python
app = QtWidgets.QApplication(sys.argv)
# Chat
font_family = load_custom_font("./window/resource/fonts/NotoSansSC-VariableFont_wght.ttf")
if font_family:
# 设置全局字体
set_global_font(font_family, 800)  # 示例设置字体权重为 400
main_window = RSP_WeChat()
```

