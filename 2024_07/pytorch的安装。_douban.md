## pip使用清华源来安装指定版本的pytorch-cuda



![Loading](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910499098.webp)

最近因为代理不支持pip下载的协议，所以不得已使用镜像源来进行pytorch下载。

这是官网给出的pytorch previous version的pip下载例子：

```cmd
pip install torch==2.2.2 torchvision==0.17.2 torchaudio==2.2.2 --index-url https://download.pytorch.org/whl/cu118
```

这个对下载地址是指定死的，每次下载必然从pytorch.org下。而且如果只写前面一段，那么就只是一个cpu的版本。即使能够用到镜像也没有cuda。

### 原理

思路是我们使用的是镜像源，但是我们指定的:

 https://download.pytorch.org/whl/torch_stable.html 

相当于是一个文件列表，告诉pip应该下载哪些需要的包和库，它会自己从清华源中下载，当然使用前需要先换源。

### 操作方法:

* 换源:

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

* 指定版本进行安装

```
pip install torch==2.2.0+cu118 torchvision==0.17.2+cu118 -f https://download.pytorch.org/whl/torch_stable.html
```



当然你需要下载不同版本的时候只需要更改==后的版本号就好。

当然这个torch和torchvision,torchaudio等等的版本需要对应，具体的查阅:

[https://pytorch.org/get-started/previous-versions/](https://pytorch.org/get-started/previous-versions/)