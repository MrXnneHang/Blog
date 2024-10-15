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

pyqt我现在是不太感冒的，因为它打着跨平台的旗号，实际上在wsl上稍微部署了一个简单的pyqt的程序，就发现谜一样的额外需要很多pyqt打头的系统依赖，以及和python依赖版本还是谜一样的一一对应否则冲突。以及在linux上opencv和pyqt也是存在冲突的。需要安装不支持直接显示GUI的opencv-headless。你这平台跨的挺艰难的，难怪大厂即使各个平台自研也不用这些通用性的库。臃肿是一方面，性能也是一方面，到时候Bug还不能自己解决更是一方面。pyqt有个专门记录bug的社区，里面几万个bug有多少是至今无法解决的。<br>

