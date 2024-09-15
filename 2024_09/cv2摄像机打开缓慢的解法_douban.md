```python
import cv2
import time

"""开始的时间"""
start = time.time()
end = None
cap = cv2.VideoCapture(0)
while True:
    ret, img = cap.read()
    img = cv2.flip(img, 1)
    cv2.imshow('self', img)
    """"第一次打开摄像头的时间"""
    if not end:
        end = time.time()
        print(f'打开摄像头用时：{end-start:.2f}秒')
    cv2.waitKey(1)
```

output:

```
打开摄像头用时：4.97秒
```



**加上cv2.CAP_DSHOW后:**

```python
import cv2
import time

"""开始的时间"""
start = time.time()
end = None
cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)
while True:
    ret, img = cap.read()
    img = cv2.flip(img, 1)
    cv2.imshow('self', img)
    """"第一次打开摄像头的时间"""
    if not end:
        end = time.time()
        print(f'打开摄像头用时：{end-start:.2f}秒')
    cv2.waitKey(1)
```

output:

```cmd
打开摄像头用时：0.63秒
```



我差不多一个月之前和我室友有过这样一个争执，

> "每次打开摄像头就要等个四五秒还没有单独线程程序无响应四五秒这东西能他妈验收。" -A

> “一次只能初始化一个摄像头不然会闪退，又不是每个人都像你那样写代码写那么好。” -B

捧杀，这是懂捧杀的。至少今天我发现只要加一个参数就能让摄像头打开快很多，但当时他是一脸抵触，说能用就行，要改你自己改。(A负责数据库，B负责UI，A也还没学QT，课设答辩前一天。)

我当时血压都高了。

我一年前的期末就跟他说你可以学pytorch，这样之后你也可以来写算法。他点头说，现在考试，等考试完了我就学。考试完后他开始LOL，确实有打开李沐那本电子版的《动手学深度学习》，但是隔了一个暑假回来后，我问他说学到哪里了，他说还没学。

于是这一次暑假前三周，我安排他去做一个python live2d渲染的项目，几乎不用基础:（暑假三周工训课程设计，比较忙确实代码写得少）

[https://github.com/RimoChan/Vtuber_Tutorial](https://github.com/RimoChan/Vtuber_Tutorial)

我跟他说，教程很乱，你先整理出来一个可以用的代码，然后写文档，main_files，一级调用函数功能介绍，可以的话流程图稍微画一下。

期间我手把手跟他说了git的各种报错怎么解决。

然后暑假放了二十来天，我终于甩掉了一个老师的锅。现在有时间做那个大锅。

我去拉了他的仓库，受到了降维打击：

[https://github.com/Caibospoem/vtuber](https://github.com/Caibospoem/vtuber)

![image-20240804125708695](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2911403327.webp)

first commit? 第一次更新?README写了什么？

文档？流程图？requirements?

然后在我部署了半个多小时的环境顺便还写了一份博客:[pip install dlib报错:缺少CMake | Windows](http://xnnehang.top/blog/85)

再运行我是心情跌入谷底。

```cmd
OSERROR: No Such File: ../res/shape_predictor_68_face_landmarks.dat
```

=-=

没有验证，没有测试，上传文件不完整，再也没有commit。

在三次课设的时候我也跟他起过类似的大大小小的争执。总体大概像这样:  

![剧情](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2911403324.webp)

![Snipaste_2024-08-04_10-14-52](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2911403323.webp)

直到现在，我彻底放弃扶起来一个室友和我组个组织写代码的想法。

也是我第一次感到，原来人类是如此具有多样性。如果说课设只是过家家，tmd这次这个项目预算都发了快一万下来，是要比赛的。结果还是只有一个人干。唉...



但我也碰到一些真的非常有意思的人:
![a1](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2911403325.webp)

![Snipaste_2024-08-04_11-48-21](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2911403326.webp)

而且，最开始时，两个人都不会写代码。第一个会一些，第二个是完全不会。

特别是第二个，他是个律师，python环境怎么部署可能都不清楚，但是也试着用gpt去写标点恢复。

相比之下，只剩下一声，唉....
