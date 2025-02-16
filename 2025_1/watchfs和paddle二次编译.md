> 这篇是为数不多的搬运呀～

## 配置 Python 源码自动同步

Paddle 的开发流程中，源码和运行时使用的代码是分离的，一般有如下几种开发方式：

- 使用 wheel 包
  - 每次修改代码后重新编译，编译后重新安装 wheel 包，重新安装 wheel 包到当前环境（跑一次编译很麻烦）
  - 编译后直接编辑安装到的 `site-package` 下的 Paddle 源码（快，但是改的不是源码，如果改完代码没往源码同步就容易丢失修改）
- 使用 `PYTHONPATH` 将 `build/python` 目录添加进去，`export PYTHONPATH=/workspace/Paddle/build/python:$PYTHONPATH`
  - 当然你同样可以每次都重新编译，但不需要再跑 wheel 包安装这一步了
  - 编译后直接编辑 `build/python` ，问题和上面也类似

为了方便代码同步，达到修改 Python 源码就相当于修改运行代码的效果，随便写了个小工具 https://github.com/ShigureLab/watchfs，可以通过 `pip install watchfs` 安装，注意只支持 Python 3.12+，没 Python 3.12 自己想办法，不接受低版本支持需求

个人日常开发开三个终端：

1. 编译终端： `conda activate paddle-py310` 、 `cd build` ，之后按需 `make -j32` 即可，一般修改 C++ 代码或者切分支后需要触发
2. 文件同步终端： `watchfs python:build/python test:build/test --exclude 'python/paddle/_typing/libs/libpaddle/'` ，将 Python 源码同步到 build 下，修改 Python 代码后什么也不用做，这个终端一般不用再看了，除非他挂了（基本不会遇到）
3. 开发终端： `export PYTHONPATH=/workspace/Paddle/build/python:$PYTHONPATH` ，运行开发命令，主要是跑单测

---

需要用的指令合集整理：<br>

```shell
watchfs python:build/python test:build/test --exclude 'python/paddle/_typing/libs/libpaddle/'
```

注意 watchfs 的 python 环境应该是 3.12+

```shell
export PYTHONPATH=/paddle/build/python:$PYTHONPATH
```
