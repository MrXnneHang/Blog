# 使用docker的简明教程

## 前置知识:

什么是镜像？什么是容器？什么是docker（dockerfile）?<br>

镜像： 类似于虚拟机的快照。<br>

容器： 把镜像启动起来，运行中的虚拟机。<br>

dockerfile: 代表了这个镜像的构建过程，或者说一个linux系统需要哪些系统依赖，哪些软件依赖。<br>

每个镜像一般都是一个微型linux系统，你可以用docker构建，管理，传播这些镜像。<br>

有什么好处：<br>

* 用户都通过docker和dockerfile构建了完全相同的环境，不会存在跨系统平台和系统配置不同导致的运行问题。<br>
* 传播过程被简化，只需要传播一个dockerfile,而不是网盘里的整合包。<br>
* 少掉很多由于时间推进带来的版本差异的问题，时效性更长。<br>

## 安装docker：

[https://www.docker.com/get-started/](https://www.docker.com/get-started/)

## 如何使用&&常用命令&&dockerfile怎么写：

这些你都可以让gpt帮你写，这里不提及了。<br>


## 碰到的问题和解决：

### docker每次构建时间长，而且因为系统不一样缺少依赖需要反复构建，效率很低:

构建python环境的时候，每次requirements.txt都会重新下载，这很浪费时间，而且最后都下完了，运行容器才发现python存在运行时依赖问题。<br>

解决方法：<br>

起一个容器，在容器里把每一步都成功后 写 Dockerfile 打包。<br>

比如我要构建python软件，那么我就可以使用python:3.10-slim这样的作为基础镜像。<br>

```cmd
 docker run -it python:3.10-slim /bin/bash
```

然后新建一个终端，进入你的项目根目录，把根目录下的所有文件拷贝到容器内。<br>

```cmd
docker cp . 462d8a083bb4:/app
```

那一串数字是你的容器id，可以通过docker ps查看。<br>

都构建好，并且把程序调试好了，你可以开始写Dockerfile了，如果不会写，可以把你之前运行过的命令都给gpt。<br>

### docker打包类似pyqt这样GUI程序的时候，运行时报错：
最好不要试图写一个GUI程序然后在docker里面运行，因为：<br>

docker本身是没有GUI的，所以你需要在docker里面运行一个vncserver，然后在windows里面用vncviewer连接。<br>

上面是Copilot帮我写的解法。<br>

但是我觉得太麻烦了，因为vnc的连接一般还需要额外的配置，比如我记得我连接树莓派的时候还需要windows安装一个vncviewer,然后每次连接用ip地址，还要输入密码。<br>

我们用docker的初衷是为了简化，而不是增加复杂度。<br>

所以这里我选择更改软件架构，比如我把pyqt的程序改成了flask的web程序，然后用docker部署，那么就只需要映射到一个端口，GUI由浏览器渲染。<br>

### 持久化本地容器和先制作确定容器步骤再打包镜像:

镜像本身是不会变的，容器销毁后数据是会消失的。<br>

但如果总是把容器启动时映射到某个特定路径，那么该路径上的数据不被手动删除，容器销毁后也不会删掉该部分数据，再次启动时，也同样不会变。<br>

一般是以/bin/bash进去，比如python的镜像一般自带pip,持久化后的容器，pip装的库下次依然可以使用。<br>

可以在这种情况下先调试环境和代码，保证容器内可以正常运行后，再把自己的修改容器的过程打包成镜像，或者我去问问能不能直接把某个容器打包成镜像。<br>

> 我先用镜像生成了一个容器，之后我持久化本地容器，修改了一些环境，调试好程序，能不能直接把该容器打包成镜像而不用写dockerfile<br>

```cmd
是的，你可以直接将已经调试好的容器打包成一个新的镜像，而不需要再编写 Dockerfile。这可以通过 Docker 提供的 docker commit 命令来实现，下面是具体步骤：

查看正在运行的容器ID或名称： 使用以下命令列出所有容器，找到你需要持久化的容器ID或名称：

docker ps -a
将容器提交为镜像： 使用 docker commit 命令将容器打包成镜像。假设你的容器ID是 your_container_id，你可以执行：

docker commit your_container_id new_image_name:tag

new_image_name:tag 是你希望新镜像的名称和标签，例如 my_app:v1。
your_container_id 可以替换成容器名称，方便记忆和管理。
验证新镜像： 镜像创建成功后，可以通过以下命令查看镜像是否已生成：

docker images
使用新镜像： 之后，你就可以像使用其他镜像一样，基于这个新生成的镜像启动新的容器：

docker run -it new_image_name:tag
```

但是，注意，持久化映射(`docker run --name local_paddle -it -v $PWD:/paddle`)的数据不会被打包，只会打包系统环境:<br>

如果需要映射文件也被打包，则还需要手动复制挂在目录到容器内部，退出容器，然后再docker commit.(具体问一下gpt,等我下次碰到了再给出一个示例。)<br>