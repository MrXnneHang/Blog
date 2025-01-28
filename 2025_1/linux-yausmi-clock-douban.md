# 热身： 尝试在 linux 上面打包我的 Yausmi-Clock v1.2

主要的难点在于:<br>

- pyqt5 兼容性适配。
- 排查之前用的 moviepy 是否支持 linux。
- 排查是否有用到 win32 底层 api 的库。
- 用 pyinstaller 打包。

其实两个排查在写的时候就应该考虑，但是，我之前压根没想到。这里需要额外的注意。<br>

为什么不考虑用 docker 打包呢？<br>

我以前在学 docker 的时候就尝试用 docker 进行一个打包，但是，存在很大的问题，在于 docker 一般的 ubuntu 和 python 镜像都是无头模式，或者说无图形界面底层库的，它用来做一些后端服务打包，是非常优秀的，但是，对于 pyqt 的打包只能说是依赖库问题很大，而且当时我修了两天也没能成功，最终的解法是，用 vnc 远程桌面连接我的 docker。<br>

emmm,那打包了还不如不打。<br>

## pyqt5 兼容性适配

requirements.txt:<br>

```txt
pyaml
pyqt5
PyQt5-Frameless-Window
opencv-python
PyQt-Fluent-Widgets
moviepy==1.0.3
```

这么直接安装后启动会报错：<br>

```shell
QObject::moveToThread: Current thread (0x5574adcd3a20) is not the object's thread (0x5574ae5364f0).
Cannot move to target thread (0x5574adcd3a20)

qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/home/xnne/miniconda3/envs/yasumi/lib/python3.10/site-packages/cv2/qt/plugins" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb, linuxfb, minimal, offscreen, vnc, webgl.
```

在 gpt 的建议下，我用这个方法解决了：<br>

```shell
pip uninstall PyQt5 opencv-python-headless
pip install PyQt5 opencv-python-headless --no-cache-dir
```

问题应该是在于 opencv-python-headlessopencv-python-headless 和 PyQt5 的冲突。<br>

于是乎环境修正为这个:<br>

```txt
pyaml
pyqt5
PyQt5-Frameless-Window
opencv-python-headless
PyQt-Fluent-Widgets
moviepy==1.0.3
```

因为包不是很多，所以我们尝试手动进行`锁版本`。<br>

我不建议用 pip freeze > requirements.txt，因为它会把所有的包都写进去，而且这样的环境的一个兼容性是很差的，只要保证核心的包是锁定的就好，让别人可以围绕这个微调。<br>

```shell
pyaml==25.1.0
pyqt5==5.15.11
PyQt5-Frameless-Window==0.4.5
opencv-python-headless==4.11.0.86
PyQt-Fluent-Widgets== 1.7.4
moviepy==1.0.3
```

至少下次别人调环境的时候知道哪些包版本可以上下浮动。<br>

## 排查 moviepy 是否支持 linux

[ModuleNotFoundError: No module named 'moviepy.editor' ](https://github.com/harry0703/MoneyPrinterTurbo/issues/535)<br>

> so guys after the November release of moviepy they removed editor module, they have changed some dir's<br>
> to fix this error temporarily you can downgrade to Moviepy 1.0.3

所以这里打包的一个重要性就体现出来了，我上次更新大概是半年前，但是这半年里，moviepy 更新了，并且移除了 movipy.editor 的模块。<br>

这个时候就只能降级到 1.0.3 了。但如果通过我自己的 requirements.txt 来安装，就会下到最新的版本然后出错。<br>

我记得 uv 似乎可以锁版本，但是我不会用，下次可以到官网观察观察。<br>

## 排查是否有用到 win32 底层 api 的库

有的。<br>

```python
# 获取屏幕物理分辨率
def get_real_screen_resolution()->dict:
    hDC = win32gui.GetDC (0)
    width = win32print.GetDeviceCaps (hDC,win32con.DESKTOPHORZRES)
    height = win32print.GetDeviceCaps (hDC,win32con.DESKTOPVERTRES)
    return {"width":width,"height":height}
```

之前用它来获取分辨率，确实很简单，但是它不通用。<br>

## 在 linux 下打包

这个时候就不得不感谢以前的自己了。<br>

[pyinstaller 打包 exe 带 movies 库闪退解决](https://xnnehang.top/blog/69)<br>

一次成功。<br>

但，强大的应该不是我，而是 pyinstaller。<br>

## What's more: 给我们的软件一个可爱的皮套。

因为我注意到打包时加入的 icon 只是负责了任务栏的图标，而不是软件的图标。<br>

而在 linux 里面加图标就比较容易了，我更倾向于，在 desktop 文件里面加入图标。<br>

创建 Yasumi-Clock.dekstop:<br>

```shell
[Desktop Entry]
Name=Yasumi-Clock
Comment=一个简单的时钟
Exec=/home/xnne/local/software/Yasumi-Clock-v1.2-linux-amd64/yasumi_clock
Icon=/home/xnne/Pictures/yasumi-icon.jpeg
Terminal=false # 设置成 false 可以 headless 运行
Type=Application
Categories=Utility;
```

### 出现的问题： 可以从程序源目录启动，但是从桌面启动不了。

报错是找不到 config.yaml 和资源文件。<br>

初步判断，是因为程序的工作目录不是程序所在目录。<br>

而这个工作目录是我的 desktop 文件所在目录。<br>

需要改一点代码。<br>

ok 啦，以为只要改一点，结果改了两个多小时。<br>

不过大致还算顺利。<br>

[https://github.com/MrXnneHang/Yasumi-Clock/releases/tag/v1.2-linux](https://github.com/MrXnneHang/Yasumi-Clock/releases/tag/v1.2-linux)<br>

顺利 release 啦～<br>

![alt text](https://image.baidu.com/search/down?url=https://img2.doubanio.com/view/photo/l/public/p2917826521.webp)

![alt text](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2917826520.webp)

![image-20250128132813843](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2917826519.webp)

下午又修了个 bug,倒计时结束闪退的问题。<br>
