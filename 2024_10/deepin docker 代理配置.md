# Deepin 的 curl 代理有效而docker search pull 的代理无效。

[https://linux.do/t/topic/234372/2](https://linux.do/t/topic/234372/2)<br>

问题和具体解决都在上面。<br>

所以这里记录一下原因：<br>

linux里，代理等级分成系统代理，终端代理，以及systemed。<br>

Qv2ray，它的默认代理是系统代理。系统代理负责所有GUI的代理，比如浏览器，telegram等。但是如果只用v2ray，会发现和windows不同，终端里任何的http请求都不会走系统代理，除非用tun模式，我不太懂tun原理，所以这里不解释了。<br>

而在终端里，用curl,conda install,pip install等，都是走终端代理。终端代理是我自己取的名字，它实际上是环境变量，http_proxy,https_proxy。可以在v2ray里面打开入站设置，然后记住端口，配置环境变量=127.0.0.1:端口。<br>

systemd代理，一些systemd启动的应用，docker,mysql这类应用，即使配置了环境变量，它也不会走终端代理，而是走systemd代理。对于docker是这么配置的:<br>

```shell
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim  /etc/systemd/system/docker.service.d/http-proxy.conf
```

然后写入：<br>

```cmd
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:port"
Environment="HTTPS_PROXY=http://127.0.0.1:port"
```

然后重载:<br>

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

重载似乎只需要重载一次，之后可以不用再次重载。<br>

之后是用户组权限问题。<br>

可能碰到当前用户没有权限访问Docker守护进程的socket文件的情况:<br>

```shell
(base) xnne@xnne-PC:~/software$ docker ps
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
```

**解决** :<br>
```shell
sudo usermod -aG docker $USER
```
`$USER`写用户名，比如我的是xnne.<br>

然后:<br>
```shell
newgrp docker
```

最后测试一下:<br>

```shell
(base) xnne@xnne-PC:~/software$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
这样就正常工作了。<br>

你同样可以用`docker search mysql`来验证你的http_proxy。<br>