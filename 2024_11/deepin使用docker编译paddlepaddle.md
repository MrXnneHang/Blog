# Deepin 使用 docker 编译 paddlepaddle

## Reference:

- [Linux 下使用 make 从源码编译](https://www.paddlepaddle.org.cn/documentation/docs/zh/develop/install/compile/linux-compile-by-make.html)<br>
- [Git 使用之(pathspec master did not match any file(s) known to git) ](https://blog.csdn.net/wankui/article/details/53328369)

## 前言：

这里只是记录一下从 docker 拉取 paddlepaddle 镜像，然后编译 paddlepaddle 的过程，Docker 的安装，如何配置 docker_proxy,如何在容器内访问 github,这里默认你已经会了。<br>

当然这里指个方向，我以前写过类似的:<br>

- [Deepin 的 curl 代理有效而 docker search pull 的代理无效。](https://xnnehang.top/blog/148)

另外是一点建议，请保证你使用的节点是稳定的，在 github clone 的过程中尽量不要出现一些奇怪的报错。我先前用一个很快节点，但是四十分钟里失败了好几次，导致我的 third-party 不全。<br>

每一次折腾依赖都非常费时间，特别是需要排查你是不是有某些 third-party 没装上。<br>

## 步骤和思路：

我们的代码代码在本地编写，然后在 docker 容器内编译，容器映射到代码目录，编译生成 whl 文件直接生成在源代码根目录 build 文件夹下。再把编译好的 whl 文件用本地的 miniconda 虚拟环境安装和测试。

## clone paddlepaddle 源码：

你需要保证使用 clone 而不是 download zip，因为编译的时候需要一些 git hash 等信息。

你可以使用你的本地 fork 仓库，但是需要保证某些 tag 是一样的，比如 develop,不然编译的时候会出现一些问题。

```bash
git clone https://github.com/PaddlePaddle/Paddle.git
```

## 拉取 paddlepaddle 镜像：

```bash
docker pull paddlepaddle/paddle:latest-dev
```

## 进入源代码根目录运行 paddlepaddle 镜像：

```bash
cd Paddle/
docker run --name paddle-test -v $PWD:/paddle --network=host -it paddlepaddle/paddle:latest-dev /bin/bash
```

`--network=host`代表我们 localhost 使用主机的 localhost 而不是容器的。

我的 http_proxy,https_proxy 放在主机的 localhost:7890,然后通过这个共享到容器内。

`-v $PWD:/paddle`代表我把当前目录映射到容器的 /paddle 目录下,我共享的是源代码根目录。

在容器内可以直接用`cd /paddle`就可以看到源代码根目录。后续，build 的目录，和 third-party 以及最终生成的 whl 文件都会在这里，同时这些生成文件也会同步到主机的源代码根目录。下次再进来编译，就不需要反复下 third-party,初次编译的缓存也都会存在，可以直接进行二次编译，时间大概从一个小时缩短到四五分钟甚至几十秒。<br>

## 配置 http_proxy,https_proxy:

分别是用户变量和 git config，这里的代理是我自己的，你需要根据自己的代理配置。

```bash
export http_proxy=http://localhost:7890
export https_proxy=http://localhost:7890
git config --global http.proxy http://localhost:7890
git config --global https.proxy http://localhost:7890
```

## 安装一些 make 需要的依赖：

```bash
pip install protobuf
apt install patchelf
```

以及很重要的，networkx,这个在官方文档里面没有提到，但是缺失的话编译最后会报错。<br>

保证你在根目录：<br>

```bash
λ xnne-PC /paddle ls
AUTHORS.md             paddle/         security/
build/                 patches/        SECURITY_cn.md
cmake/                 pyproject.toml  SECURITY_ja.md
CMakeLists.txt         python/         SECURITY.md
CODE_OF_CONDUCT_cn.md  r/              setup.py
CODE_OF_CONDUCT.md     README_cn.md    test/
CONTRIBUTING.md        README_ja.md    third_party/
doc/                   README.md       third_party.tar.gz
LICENSE                RELEASE.md      tools/
```

然后:<br>

```bash
pip install -r python/requirements.txt
```

> `IMPORTANT:`<br> > `注意！！！`<br>
> docker 环境下有多个 python,你 python 一下看看默认使用哪个 python<br>
> 我是默认使用 3.9,在最后 make -j 的时候，而我构建的是 310 的 paddle<br>
> 就报错建议运行:pip install -r python/requirements.txt<br>
> 但是反复运行都会缺一个 networkx。<br>
> 然后我尝试指定了 pip3.10 install -r python/requirements.txt,就没再报错一次编译完成。<br>

> 如果你编译的 whl 和你机器 python 是相同版本，那么放心使用 pip install -r python/requirements.txt,如果是不同版本，那么参照你的版本指定 pip.<br>

## 切换到 develop 分支：

如果像文档里直接 `git checkout develop`,会导致后面找不到 develop tag 的 hash,但是 grep 是可以找出那个 hash 的。

当然我建议你也先这么运行:<br>

```bash
git checkout develop
```

因为我流程里有这个操作，我也不清楚缺少它是否会产生某些影响。<br>

查看当前分支：<br>

```bash
λ xnne-PC /paddle git branch -a
* (HEAD detached at origin/develop)
  develop
  remotes/origin/1.8.5
  remotes/origin/HEAD -> origin/develop
  remotes/origin/a246d2cc8876d9efb0b733f0ae02e1bd973
  remotes/origin/add_kylinv10
  remotes/origin/ascendrelease
  remotes/origin/cinn-trivalop-fuse
  ...
```

你可以找找有没有`remotes/origin/HEAD`，如果没有，可以运行一下`git fetch`。<br>

当然我建议运行`git fetch`，因为似乎编译后面会检查一些 tag hash,如果没有 fetch,可能会缺少一些 tag hash。<br>

这时候我再运行了：<br>

```bash
git checkout origin/develop
```

它会弹出来一些 Note 像这样:<br>

```bash
λ xnne-PC /paddle git checkout origin/develop
Note: switching to 'origin/develop'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

```

如果看到 note 应该就是成功了，然后再运行。你就可以进行下一步。<br>

## 编译 paddlepaddle：

保证你在根目录：<br>

```bash
cd /paddle
mkdir build/
cd build/
```

### 然后运行 cmake,你需要确认你的 DPY_VERSION，它决定你编译出来 whl 的 python 版本。<br>

```bash
git config --global --add safe.directory '*' # 这个是解决中断输入的
time cmake .. -DPY_VERSION=3.10 -DWITH_GPU=OFF
```

第一次运行这个会很久，和你的网速有关，同时也是先前我提到，请保证你的节点是稳定的，不然会出现一些奇怪的报错。<br>

以及中间可能会需要中断出来输入一些指令，比如:<br>

```bash
git config --global --add safe.directory /paddle/third_party/absl
```

而且不少，你可以留意一下，当然如果你找到解决方法也可以告诉我，不然一直中断会很烦。<br>

ps:我这里找到了一种方法:<br>

```bash
git config --global --add safe.directory '*'
```

之前用的`git config --global --add safe.directory /paddle`在这里并不起效果。

以及结束后可以二次运行看是否缺少某些 third-party，如果依然缺少，你可以手动 clone 它。<br>

### 多核编译(我这里使用 10 核):<br>

```bash
make -j10
```

同样需要很久很久...

## 验证编译结果:

如果前面都没问题的话，你应该会在源代码根目录的 build 目录下看到一些 whl 文件。<br>

这个时候我们用 miniconda 来安装它:<br>

```bash
conda activate paddle-test # 自己创哈，保证DPY_VERSION和你的环境一致
pip install -U build/python/dist/paddlepaddle-0.0.0-cp39-cp39-linux_x86_64.whl
```

```bash
python

>>> import paddle
/home/xnne/miniconda3/envs/paddle/lib/python3.9/site-packages/paddle/utils/cpp_extension/extension_utils.py:686: UserWarning: No ccache found. Please be aware that recompiling all source files may be required. You can download and install ccache from: https://github.com/ccache/ccache/blob/master/doc/INSTALL.md
  warnings.warn(warning_message)
>>> paddle.utils.run_check()
Running verify PaddlePaddle program ...
I1105 10:44:10.085186 70641 pir_interpreter.cc:1492] New Executor is Running ...
I1105 10:44:10.085837 70641 pir_interpreter.cc:1515] pir interpreter is running by multi-thread mode ...
PaddlePaddle works well on 1 CPU.
PaddlePaddle is installed successfully! Let's start deep learning with PaddlePaddle now.
>>> exit()
```

第一个警告似乎可以通过`apt install ccache`解决，但是我没有试过。<br>

你看到`PaddlePaddle is installed successfully! Let's start deep learning with PaddlePaddle now.`就代表你编译成功了。<br>

## 二次编译：

- 修改底层的头文件：paddle/fluid/platform/enforce.h
- 修改 Op 的 cc 文件：paddle/fluid/operators/rank_loss_op.cc
- 修改 python 文件：python/paddle/tensor/math.py

这些俺打算放到下篇博客讲。<br>
