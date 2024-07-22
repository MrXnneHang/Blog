只要引用了moviepy就会这样。并且手动添加imageio_ffmpeg.biaries无效。

## Issue:

```cmd
FileNotFoundError:Package has no location <module 'imageio_ffmpeg.biaries'(namespace)>
```



## Solve:

在项目根目录Create

hook-imageio_ffmpeg.py:

```python
from PyInstaller.utils.hooks import collect_all
datas, binaries, hiddenimports = collect_all('imageio_ffmpeg')
```

在Pyinstaller调用中加入.

**--additional-hooks-dir=. **

如果你和我一样用auto-py-to-exe

**Advance - additioal-hooks-dir**

然后添加项目根目录(对于我是Yasumi-Clock):


![image-20240722211150700](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910936689.webp)