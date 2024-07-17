Pyinstaller 我用起来很不适应，而且打包一次很久，调试很麻烦。

有人开源了GUI 版本的Pyinstaller:

[https://github.com/brentvollebregt/auto-py-to-exe](https://github.com/brentvollebregt/auto-py-to-exe)

具体用法:

```cmd
pip install auto-py-to-exe
auto-py-to-exe
```

支持python 3.6 ~3.12

极大地减少了我打包所花的时间。不过似乎资源文件不会被打包，需要手动拖到相应目录，模拟和运行main.py一样的布局。

包括:

```cmd
jpg,png,mp4 ...
yml,json ...
```

