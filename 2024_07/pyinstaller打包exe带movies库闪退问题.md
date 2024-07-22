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


![image-20240722211150700](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407222111729.png)