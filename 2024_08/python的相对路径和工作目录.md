## Python 执行 | 什么才是工作目录

### Issue:

不管我是否具有:

`shape_predictor_68_face_landmarks.dat`

这个文件。我都会报下面那个错误。导致错误的不是文件命名或者说相对路径的写法。而是我的工作目录。

```cmd
PS D:\program\Vtuber_Tutorial-master> python .\4\虚境.py
Traceback (most recent call last):
  File "D:\program\Vtuber_Tutorial-master\4\虚境.py", line 13, in <module>
    import 现实
  File "D:\program\Vtuber_Tutorial-master\4\现实.py", line 20, in <module>
    predictor = dlib.shape_predictor('../res/shape_predictor_68_face_landmarks.dat')
RuntimeError: Unable to open ../res/shape_predictor_68_face_landmarks.dat
```

常见编译编译器其实run是这么一行代码:

以vscode为例：

```cmd
PS D:\program\Vtuber_Tutorial-master> D:\miniconda\envs\vtuber\python.exe D:\program\Vtuber_Tutorial-master\4\虚境.py
```

```cmd
::open folder决定了D:\program\Vtuber_Tutorial-master
::编译器选择决定D:\miniconda\envs\vtuber\python.exe
::run的文件决定的是D:\program\Vtuber_Tutorial-master\4\虚境.py
```

而python中写相对路径，不是相对于被执行文件，也不是相对于open folder，而是决定于工作目录。

是这个:

```cmd
PS D:\program\Vtuber_Tutorial-master>
```

open folder可以让它初始化成项目根目录，每次run的时候也默认用这个作为工作目录，但是实际上可以更改。

所以可以理解为什么我这个会报这个错，虽然我有shape_predictor_68_face_landmarks.dat

```cmd
PS D:\program\Vtuber_Tutorial-master> python .\4\虚境.py
Traceback (most recent call last):
  File "D:\program\Vtuber_Tutorial-master\4\虚境.py", line 13, in <module>
    import 现实
  File "D:\program\Vtuber_Tutorial-master\4\现实.py", line 20, in <module>
    predictor = dlib.shape_predictor('../res/shape_predictor_68_face_landmarks.dat')
RuntimeError: Unable to open ../res/shape_predictor_68_face_landmarks.dat
```

**我之前理解偏差在，main file决定了工作路径和目录**，也就是当我直接执行

`Vtuber_Tutorial-master\4\虚境.py`

的时候，"../res/shape_predictor_68_face_landmarks.dat"指的是

`Vtuber_Tutorial-master\res\shape_predictor_68_face_landmarks.dat`

**这是错的，实际上不是这样。**

工作目录只是指的是:

`PS D:\program\Vtuber_Tutorial-master>`

当我以这个目录执行，那么在找"../res/"的时候会找`D:\program\res`下方的文件。

至于能不能跨越到项目之外找到。

**答案是可以跨项目，找到项目外的文件。**

但是意义不大，因为工作目录改变后很多文件引用，import都会报错，而且使用到的内容应该被封装到项目中，而不是想着用户能够自己脑补着还原出来你神奇的文件树。

So，最后得出解决方法:

### Solve:

改变工作目录

```cmd
PS D:\program\Vtuber_Tutorial-master> cd .\4\
PS D:\program\Vtuber_Tutorial-master\4> python 虚境.py
```



### 还应该告知运行Main File的工作目录：

另外这也提醒了我。

平时单纯只是发：

**运行：**

```cmd
python app.py
```

是不够的。

最好是

```cmd
D:\program\字幕生成V2>python app.py
```

工作目录也是需要告知的。

