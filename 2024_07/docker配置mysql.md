# 使用docker安装和使用MYSQL | Windows

比起官网MYSQL的安装和配置docker真的太方便了。

![](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2409194280.webp)

## Docker Desktop安装(windows)

[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

正常安装即可，部分电脑安装后会碰到报错不支持WSL2，虚拟化平台什么的。

则管理员运行cmd然后:

```cmd
bcdedit /set hypervisorlaunchtype auto
```



## MYSQL安装

````
docker search mysql
````

 

## 启动MYSQL容器

```cmd
docker stop my-mysql
docker rm my-mysql
docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=123456 -v D:\iso\mysql:/var/lib/mysql -p 3306:3306 -d mysql:latest
```



### 端口被占用的报错.

```cmd
docker: Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:3306 -> 0.0.0.0:0: listen tcp 0.0.0.0:3306: bind: An attempt was made to access a socket in a way forbidden by its access permissions.
```
**解法:重启winnat**   
先管理员运行cmd,然后:   
```windows
net stop winnat  // 停止WinNAT服务
net start winnat // 重新启动WinNAT服务
```

## 创建MYSQL DATABSE

如果2中设置了密码则第一步启动的时候也需要密码。

```
docker exec -it my-mysql mysql -u root -p
CREATE DATABASE faces;
```



## 用dbeaver来编辑或者浏览数据库(可选)

[https://dbeaver.io/](https://dbeaver.io/)

界面挺现代化的。

如何连接MYSQL数据库。

菜单栏->数据库->新建数据库连接->SQL->MYSQL->驱动属性->allowPublicKeyRetrieval=True(否则会出现输入密码无法连接的情况)->主要->数据库名称->root密码->完成。