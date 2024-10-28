# jupyter hub取消对用户名的需求

今天在运行这个的时候:<br>

```shell
docker run -p 80:80 --rm --env USER_PASSWD="hang" -v $PWD:/home/paddle registry.baidubce.com/paddlepaddle/paddle:3.0.0b1-jupyter
```

发现登陆的时候需要user_name和password,而我并未设置user_name。<br>

但问了一下gpt,这个password确实是为Jupyter配置的，那就让我有点摸不着头脑了。<br>

我们需要修改jupyter配置文件，让它只require password.<br>

以可交互式终端启动容器:<br>

```shell
docker run -p 80:80 --rm -it -v $PWD:/home/paddle paddlepaddle/paddle:3.0.0b1-jupyter /bin/bash
```

生成config文件:<br>

```shell
jupyter notebook --generate-config
```

```shell
(base) root@a0b8b2dfeae9:/srv# jupyter notebook --generate-config
Overwrite /home/jovyan/.jupyter/jupyter_notebook_config.py with default config? [y/N]y
Writing default config to: /home/jovyan/.jupyter/jupyter_notebook_config.py
```

记录一下config的路径.<br>

因为容器内可能没有文本编辑器:<br>

```shell
apt update
apt install vim
```

当然如果你喜欢用其他的也可以装.<br>

随后修改config文件:`vim /home/jovyan/.jupyter/jupyter_notebook_config.py`<br>

里面整个文件都是注释的，你可以直接在文件开头加入:<br>

```python
from notebook.auth import passwd
c.NotebookApp.password = passwd("hang")
c.NotebookApp.allow_password_change = False
c.NotebookApp.ip = "0.0.0.0"
c.NotebookApp.port = 80
```

退出来后运行`jupyter notebook`<br>

容器内可能需要`--allow-root`，因为不允许直接root用户运行,则运行`jupyter notebook --allow-root`<br>

这样进去后，你会发现就只需要一个password了.<br>

最后，因为我们最初在启动容器的时候，有映射到本地的文件系统，那么之后我们再次启动容器，如果映射的是相同的容器，则无需再次执行这些操作，只需要在/bin/bash情况下运行`jupyter notebook --allow-root`即可。<br>

一点关于docker的笔记:<br>

* Docker Image(镜像)在拉取后不会被改变，是静态的。
* Docker Conatiner(容器)是用户根据Image创建的，在容器销毁后所有数据都会消失，可以改变容器内的内容。<br>
* Container 可以映射到本地的特定路径，或者特定端口。映射到特定路径后，可以实现数据本地持久化，容器销毁也不会消失。<br>