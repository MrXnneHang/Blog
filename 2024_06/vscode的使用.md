&#x20;                                                                         2024.4.2

# &#x20;   vscode的使用

## 一.vscode与github与git

### 1.本地库与远程库连接：

源代码管理中打开打开文件夹——终端输入git init 创建git仓库——上传——为远程仓库命名

### 2.打开远程库克隆库：

源代码管理点击克隆仓库——在github上复制需要的库连接

### 3.推送更改文件：

远程连接或克隆库后，修改项目中的文件，先提交更改，再推送

**问题：**

**（1）推送文件到远程库会报错git pull不了，报错为 :**

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200647496.jpeg)

解决：终端输入代码

    git config --global --unset http.proxy

如果用不了： [完美解决 git报错fatal: unable to access ‘https://github.com/.../.git‘:Recv failure Connection was reset\_git recv failure: connection was reset-CSDN博客](https://blog.csdn.net/m0_63230155/article/details/132070860)

**（2）推送过大文件报错：** 解决：将过大文件纳入gitignore文件中，不进行版本管理 注意：已经提交的大文件会纳入git中，所以无法推送

\*\*总结：\*\*修改多份文件时，应该一份一份提交，注释修改内容，然后统一推送不能一次性提交，不然推送时容易出问题

### 4.vscode在github上增删git分支

**新增**

​	1.克隆或打开git项目 ​&#x9;

&#x20;   2.左下角点击分支：

&#x20;	![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200647288.jpeg)

&#x20;	3.创建新分支并命名： 	![1712809498656](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200648556.png) 	4.创建后将git分支push到github上

**删除**

​	**1.删除本地分支：** ​	删除本地分支：

    git branch -d 分支名（remotes/origin/分支名）

​	强制删本地：

    git branch -D 分支名

​	**2.删除远程分支：** ​	删除远程分支：

    git push origin --delete 分支名（remotes/origin/分支名）

​	![image-20240411134417740](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200648938.png)

​	远程分支已删除，vscode还有记录，修剪远程分支：

    git remote prune origin

问题：

有时候删除不了整个项目 原因：修改后的文件没有提交或退回导致git持续运行 解决：提交或退回文件即可删除项目

### 5.添加.gitignore文件：

​	作用：声明不想要进行版本管理的文件 	用法： 	![1712988980509](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200648512.png)

### 6.git lfs:

作用：管理大型文件的版本控制 用法：下载gitlfs——安装在git/bin下——命令：git install lfs——git lfs track 文件夹/

## 二.vscode插件与环境解释器配置

**1.remote** 作用：远程隧道连接 使用：

### 2.Better Align

​	作用：优化代码缩进格式 ​	使用：alt+A

### 3.TODO tree

​	作用：高亮标识将要修改的代码块 ​	使用：#TODO

### 4.codeif

​	作用：给变量名取名字 ​	使用：双击变量——右键——选择codeif

**5.IntelliCode**

​	作用：java插件附带的ai自动整合代码

6.markdown image 作用：插入图片 使用

## 三.环境解释器的配置

### 1.java的JDK：

​	![1712835209682](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200648607.png)

为了编译您的Java项目，您需要下载并安装一个Java开发工具包（JDK）。JDK是用于开发Java应用程序的软件开发环境，它包括Java编译器、运行时环境和其他工具。

*   ​	**vscode输入命令：**

        java:Overview

    ![1712838336906](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200650011.png)

*   ​	  **点击show the java settings：**

    ![1712838349364](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200650522.png)

*   ​	**输入 home，点击settings.json:** 	![1712838375684](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200650588.png)

*   ​	**在"java.jdt.ls.java.home": "   " 中插入安装的jdk的完整路径：**

    ![1712838389505](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200650829.png)

### 2.jupyter notebook

问题：在下载低版本的tensorflow后，会出现jupyter内核重启失败的问题：

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200651507.jpeg)

原因：tensorflow版本过低，导致于jupyter的typing extension不兼容，jupyter无法启动

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200651641.jpeg)

解决：更换新版tensorflow

推广：当出现该类型的内核错误时，都是因为库的版本不适配导致的

## 四、ssh远程连接

[VS Code Remote-ssh 远程控制Windows主机 + 免密登录 + 内网穿透\_vscode 免密ssh 连接 windows-CSDN博客](https://blog.csdn.net/YYwxcsdn/article/details/134764938)

[vscode通过ssh连接服务器（吐血总结）\_vscode ssh-CSDN博客](https://blog.csdn.net/Oxford1151/article/details/137228119)

### 1.了解术语与概念：

*   **ssh**：安全的网络通信协议，是计算机互相交流的通道。

*   **服务端**：需要远程连接上的端口，所有处理都在服务端上进行

*   **客户端**：通过远程连接访问服务端的对象，用于显示在服务端处理的信息以及发送操控服务端的命令

*   **username**：服务端此时使用的用户名C：/user/username/，打开cmd输入whoami可查

*   **hostname**:服务端此时的主机名，打开cmd输入hostname可查，主机名在dns解析后变为ip地址

### 2.window服务器，vscode客户端访问服务器的步骤：

**启动服务器：**\
&#x20;   1.下载openssh服务端与客户端：

&#x20;       在windows系统中搜索可选功能即可找到相应的插件

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200651352.jpeg)

&#x20;   2.启动openssh-serve：

win搜索（服务），打开后设置启动与自启动：

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200651956.jpeg)

&#x20;   3.运行命令：

&#x20;   管理员运行powershell：

启动sshd服务

`Start-Service sshd`

将sshd服务设置为自动启动，若不设置需要在每次重启后重新开启sshd

`Set-Service -Name sshd -StartupType 'Automatic'`

确认防火墙规则，一般在安装时会配置好

`Get-NetFirewallRule -Name ssh`

若安装时未添加防火墙规则"OpenSSH-Server-In-TCP"，则通过以下命令添加

`New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22`\
&#x20;

**连接服务器：**\
&#x20;   1.在客户端生成公钥与私钥：

&#x20;   在存放密钥对的目录下打开powershell/cmd，输入以下命令：\
ssh-keygen -t rsa -b 2048 -f C:\Users\YourUsername.ssh\id\_rsa\_windows ​

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200651843.jpeg)

我设置的文件名叫id\_rsa，本客户端用户名叫33919

注意：要使用免密登陆时，在创建密钥对是不要再输入密码，直接空格跳过，此密码是对密钥的加密

&#x20;   2.上传公钥：

将公钥上传到服务端，作为认证。

将在客户端创建好的公钥文件复制到服务端的`C:\Users\username\.ssh\authorized_keys`文件中

问题：在`C:\Users\33919.ssh路径下找不到authorized_keys`

原因：在C:\ProgramData\ssh\ssh.config文件中，被设置了禁用

解决：打开ssh.config并修改以下内容

确保以下3条没有被注释 `PubkeyAuthentication yes # 使用公钥 AuthorizedKeysFile	.ssh/authorized_keys # 公钥位置 PasswordAuthentication no # 免密登录`

确保以下2条有注释掉 `#Match Group administrators`

`AuthorizedKeysFile PROGRAMDATA/ssh/administrators_authorized_keys`

&#x20;   3.在vscode中下载ssh远程连接插件

&#x20;   4.在vscode中添加主机：

&#x20;   (1)点击连接到再点击连接到主机

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200652002.jpeg)

&#x20;   （2）点击添加新主机：

注意：在服务端要打开ssh-serve，然后输入 `ssh username@hostname`

username：服务端此时使用的用户名C：/user/username/，打开cmd输入whoami可查

hostname:服务端此时的主机名，打开cmd输入hostname可查\
![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200652330.jpeg)

&#x20;   &#x20;

&#x20;   5.在vscode中添加私钥：

&#x20;   作用：连接时直接读取私钥，访问时无需输入密码访问，更便捷

添加主机完成后，点击配置ssh主机，进入后添加:

`IdentityFile "C:\Users\username.ssh\id_rsa"`(读取私钥）

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200652521.jpeg)

**穿透服务器：**

不同局域网中的计算机无法互相连接，通过使用内网穿透出一个隧道，让服务端的特定端口通过这个隧道暴露在公网中，客户端通过连接这个隧道，即可访问服务端

&#x20;   1.获取公网ip：

&#x20;   (1)首先去[樱花FRP](https://www.natfrp.com/)注册账号并实名认证

&#x20;   (2)下载并安装对应操作系统的frp客户端

&#x20;   (3)创建tcp隧道

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200652943.jpeg)

本地ip默认不更改，本地端口选择22（ssh）端口

&#x20;   （4）运行隧道，查看日志

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200652835.jpeg)

&#x20;   2.配置vscode远程连接：

&#x20;   具体详情见上面的在vscode中添加主机部分

&#x20;   不同点：

&#x20;   在输入`ssh username@hostname`连接时需要配置端口号，具体为：`ssh username@hostname -p <端口号>`

其中hostname为当前公网的ip，端口号为服务端监听公网的端口：\
![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406200653840.jpeg)

其余部分与上面的主机连接部分一模一样

总结：ssh是一个连接俩个计算机的通道，需要客户端需要知道服务端的位置才能连接，所以连接时需要服务端的ip地址，并且还要知道当前服务端的使用者是谁。客户端生成密钥对，私钥自己保管，公钥上传给服务端，当客户端访问服务端时，客户带想服务端发送公钥，服务端打开权限文件查看公钥是否匹配，匹配后，服务端将加密后的信息传送给客户端，客户端通过私钥进行解密从而获得信息。

当俩台计算机处于同一个内网时，客户端只要知道服务端的ip地址与用户名&#x20;

当俩台计算机处于不同的内网时，客户端要知道服务端映射到公网的ip地址与端口

原因：俩台计算机是不能互相连接的，因为不同内网下，俩台计算机的ip不能互相解读，因为不同内网可以看作公网的一个大ip，在内网中又分出多个小ip给计算机，大ip是可以互相连接通信的，但小ip不行。

这时需要用到内网穿透技术：不同的内网通过网关将信息传输到公网上，所以想要获取别的内网上的信息，必须通过公网获取，因此我们将服务端的内网的一个端口通过一根隧道穿出到某个公网上，即将内网的端口以及ip地址映射为公网的端口以及ip地址，由于内网只能访问公网，所以客户端所处的内网就可以通过连接到这根隧道（特定的公网ip与端口），来连接到服务端所处的内网
