## #行宽:

don't do these:

1.

```python
# ❌
a = 1;b = 2

# ✔
a = 1
b = 2

# 不要用;把两句合并成一句
```

2.

```python
# ❌
a = b * c + c * d + d * e \
  + c * f + e * g + e * l

# ✔
a = (b * c + c * d + d * e  
   + c * f + e * g + e * l )

# 不要用\来显示续行，使用(),[],内部可以隐式续航。
# 写长的if , while 也是同理。
```

```python
# 长字符串也应该用()续行，会被识别到同一句中。
a = ('你好啊'
     '这是一个很长的字符串.')
print(a)
```

output:

```cmd
你好啊这是一个很长的字符串.
```





之后请继续 : [Google python 编写规范](https://github.com/zh-google-styleguide/zh-google-styleguide/blob/master/google-python-styleguide/python_style_rules.rst)

3.

```python
# ❌
a = a_long_line().of_chained_methods(
    ).that_eventually_provides(
    ).an_answer()
# ✔
a = (a_long_line()
     .of_chained_methods()
     .that_eventually_provides()
     .an_answer())

# 只要有一个(),内部就可以任性换行。并且也建议用()来管理长表达句。
```

```python
# ❌
if a is not None and b != a and c == a and b * c < 1000 \
   and c != 0:

# ✔
if (a is not None
    and b != a
    and c == a
    and b * c < 1000
    and c != 0):
    
# if,while同理,()永远是你大哥。
```



##  #括号

上面提到的括号都是用于隐式续行。

但是()也是元组的意思。

don't do these

1.

```python
# ❌
if (x):
  print(True)
# ✔
if x:
  print(True)
```

2.

```python
# ❌
return (a,b)

# ✔
return a,b

# 除非你明白你想要返回元组，否则请不要这么做。
```



## #缩进

请不要混合使用tab和空格。

请不要使用tab。标准缩进是四个空格，隐式续行需要左对齐，字典需要四缩进。

```python
# ✔
a = (value1,
     value2,
     value3)

b = {"long_value":
         long_value,}

# 禁止2缩进,禁止无悬挂缩进(b左对齐。)
```



## #！（shebang）行

它可以帮助python找到解释器。一般写在main_files中，因为它在import的那些file中其实并不生效。有些鸡肋，但可以用来区分那个是运行的main_file。

```python
#!./env/python
```



## #注释字符串。""""""

python中除了用#注释外，""""""也是常用的，但是我是乱用的。

### 用法1:文件开头做文档说明。

```python
"""文件功能的一行概述，以句号结尾。

留一个空行，接下来写总体描述。

经典使用案例（用法）：

main_window = MainWindowResponse()
main_window.show()
"""
```

### 用法2:定义的类或者函数的说明文档。

满足任意条件：

> 1.公开的API
>
> 2.长度过长
>
> 3.逻辑不能一目了然

同样道理，一样是一句描述，一个经典用法。描述应该用陈述句或者祈使句，不因过长。可以陈述返回，或者做了什么处理。

```python
class ManualSelectionWindow(QtWidgets.QWidget):
    """复刻一个传入Window的布局，并且能够在上面画矩形来确定x,y,w,h.
    
    Args:
    [
    select_window_pos:[int,int,int,int]
    要复刻的window的x,y,w,h.
    
    child_button_pos:[Button1_pos,Button2_pos ...]
    你要复刻的Window上面的所有Button Pos
    
    ]
    
    Usage:
    child_button_poses = self.list_main_button_pos()
    self.window1 = ManualSelectionWindow(self.main_window_Pos,child_window_pos)
    self.window1.show()
    """
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



[之后请继续.](https://github.com/zh-google-styleguide/zh-google-styleguide/blob/master/google-python-styleguide/python_style_rules.rst)

## #代码注释

```python
# ✔

# 我们用加权的字典搜索, 寻找 i 在数组中的位置. 我们基于数组中的最大值和数组
# 长度, 推断一个位置, 然后用二分搜索获得最终准确的结果.

if i & (i-1) == 0:  # 如果 i 是 0 或者 2 的整数次幂, 则为真.
    
    
# ❌

# 不好的注释: 现在遍历数组 b, 确保每次 i 出现时, 下一个元素是 i+1
```

### 代码注释什么时候写：

* 涉及特殊语法，i & (i-1) 在其他语言中可能都没有，别人读到会很懵。说实话，我也每读懂。
* 涉及复杂 for 循环操作，或者多个 if elif else。
* 特殊变量赋值，说明用处。

必须假设看代码的人水平没有自己高，让对方能够最快地读完整个代码。

## #字符串：

### 1.多用fstring,少用+，为了清晰。

```python
# ✔

x = f'名称: {name}; 分数: {n}'

# ❌

x = first + ', ' + second
x = '名称: ' + name + '; 分数: ' + str(n)
```



### 2.不要在 for 循环中用字符串加法，为了时间复杂度。用.join(list)。

```python
# ✔
items = ['<table>']
for last_name, first_name in employee_list:
    items.append('<tr><td>%s, %s</td></tr>' % (last_name, first_name))
items.append('</table>')
employee_table = ''.join(items)

# ❌

employee_table = '<table>'
for last_name, first_name in employee_list:
    employee_table += '<tr><td>%s, %s</td></tr>' % (last_name, first_name)
employee_table += '</table>'
```

这个我踩大坑。

似乎字符串加法会产生n^2的复杂度,而.join是n。

### 3.统一单双引号，但避免反转义"和'。

```python
# ✔

Python('为什么你要捂眼睛?')
Gollum("I'm scared of lint errors. (我害怕格式错误.)")
Narrator('"很好!" 一个开心的 Python 审稿人心想.')

# 仅当内部需要用到单引号或者双引号的时候才考虑变换外部符号。我一般统一用""

# ❌

Python("为什么你要捂眼睛?")
Gollum('格式检查器. 它在闪耀. 它要亮瞎我们.')
Gollum("伟大的格式检查器永在. 它在看. 它在看.")
```



### 4.多行字符串。

缩进：

```python
# ✔
long_string = """如果你可以接受多余的空格,
    就可以这样."""

long_string = ("如果你不能接受多余的空格,\n" +
               "可以这样.") # 这个 + 可以省略

long_string = ("如果你不能接受多余的空格,\n"
               "也可以这样.")


# ❌

    long_string = """这样很难看.
不要这样做.
"""
```

另外请使用",而非'。



## #Log和Error



我还没学Log info,ValueError,OSError。

之后碰到合适的项目再像argparser那样独立开一个。



## #TODO注释

在临时、短期和不够完美的代码上添加 TODO (待办) 注释.

```python
# TODO(crbug.com/192795): 研究 cpufreq 的优化.
# TODO(你的用户名): 提交一个议题 (issue), 用 '*' 代表重复.

# 我觉得放在main_file里面是OK的
```



## #import

### 1.除from impoort外,各自成一行。

```python
# ✔

import os
import sys
from typing import Any, NewType


# ❌

import os,sys
```

### 2.顺序

1.标准库

```python
import sys
import os
```

2.第三方模块和包.

```python
import torch
```

3.第三方库的子包

```python
import matplotlib.pyplot as plt
```

4.自己代码中的子包

```python
from util import write_long_line
```



## #语句

if 那些事：

```python
# ✔
if a: b = 2

# ❌
if a: b = 2
else: b = 1
```

鸟事情真多。

其他没啥。

## #命名。



### 驼峰法：

 UserName

我才知道这个叫驼峰。

### 下划线:

 user_name/User_Name

还有一点区分:

对于局部变量/函数/函数参数，用小写下划线:user_name

全局变量/类变量,用大写下划线User_Name



我经常混着用两个，实际上两者有点区分。

驼峰一般用来做类名，文件名，下划线一般用来做变量名。





## #MainFile

通常只在这里面def main():

但我习惯性在很多个文件里面写:

```python
if __name__ == '__main__':
    function()
```

因为要测试我的函数，并且也避免主程序在调用到函数文件时导致函数文件执行一遍的问题。我这里就不作改变了，依然只写一个def main，其他文件都会依然保持原来的可执行方式。



## #函数

重头戏。

### 1.函数长度不应该超过40行

函数应该小巧且专一.

若一个函数超过 40 行, 应该考虑在不破坏程序结构的前提下拆分这个函数.

要一目了然.

### 2.命名。

小写下划线:

write_long_line()



## #类

### 1.变量类型声明

```python
def __init__(self,name: str) -> str:
```

同理这个也应该被函数采用。

### 2.类名/变量名

驼峰:

class FunASRModel(nn.Moudle)

实例:

ASR_Model = FunASRModel()

### 3.文档字符串

参上。

要求，简洁，明了，陈述句。描述必要的args，和return。

以及经典用法。





## 在最后：



啊~（咆哮ing...）

我这个顶级拖延症成功自主地分多次完成了一件事情，虽然每次隔得都有点久，但每次像水一下我就会来写这个。

我又觉得自己写得那个字幕生成V2像勾巴了。

总结自己以前的毛病:

>
>
>1.变量名随便取。驼峰也用，小写下划线也用，大写下划线也用，不分场景的，就单这一项直接判处死刑。
>
>2.函数巨长。而且中间引用其他函数的时候一点注释都没有。
>
>3.函数，类的类型声明一点没有，没有input类型，没有return类型。
>
>4.文档字符串一点不写，一点没给以后的自己考虑。
>
>5.排版看得就糟心。
