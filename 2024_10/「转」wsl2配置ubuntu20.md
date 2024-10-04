# 转载的部分:

转载自:[https://nagi.fun/wsl2](https://nagi.fun/wsl2)<br>

感觉随时可能会用到，免得到时候又自己折腾一遍。<br>

---


买了新电脑（3090ti）用来炼丹，犹豫了许久后在双系统和 WSL 中选择了后者。<br>

原因如下：<br>

不想折腾双系统的硬盘、网络配置，而 WSL 可以同步主系统的 host
尝鲜 WSL2，知道这东西好久了，但之前的用的拯救者笔记本跑起来很卡，没当生产力用过
存在感低，2-3 秒就能开启的子系统，可以跑着任务挂后台，同时满足我 “不想在 windows 里配环境” 的奇怪要求
网上说现在的 WSL2 对 NVIDIA GPU 的支持很好，很)多帖子中要避的雷都已经不再重要了。<br>

## 配置历程#
回忆一下配置路程：<br>

## 下载 WSL2#
Win11 下，只需要在 WIndows Powershell 中运行下面指令即可完成全部系统配置，不需要再去勾选什么 “虚拟机平台” 和 “开启 HyperV”：<br>

```cmd
wsl --install # 默认是 LTS 版本，要下载指定版本可以先输入wsl --list --online 查看
```

## 迁移硬盘#
个人习惯把东西放其它盘里，但 WSL 默认是装在系统盘，先确定版本：<br>

```cmd
wsl --list --verbose
```

### 导出现有的系统：<br>

```cmd
wsl --export Ubuntu-20.04 D:\Ubuntu.tar # 我这里的版本和命名是这样的
```


### 取消挂载：<br>

```cmd
wsl --unregister Ubuntu-20.04
```
### 重新挂载：

```cmd
wsl --import Ubuntu-20.04 D:\wsl2\Ubuntu D:\Ubuntu.tar
```

## wslconfig#
Win + r 输入 % UserProfile%，然后在该目录下创建一个名为 .wslconfig的文件（记得去开启文件后缀显示），具体配置还是得去网上找专业内容，WSL2 默认只能使用一半内存，我这里希望全开而已，前两条是网络 host 省心设置。<br>

```cmd
[wsl2]
networkingMode=mirrored
autoProxy=True
memory=32GB
```

## 避雷#

这里要注意的是，迁移硬盘重新挂载后默认用户将是 root 而非一开始登陆的 Unix User，为了美好的未来，最好不要用 root 用户来完成全部操作。<br>

```cmd
vim /etc/wsl.conf
```

然后复制下面内容，systemd 应该默认就有了现在：<br>

```cmd
[user]
default=你最开始注册的用户名

[boot]
systemd=true
此外，不要在 WSL 里尝试删库大法，可能会把整个盘都删了，想感受效果去虚拟机里整。
```
## 关机#
不用的时候想释放资源，可以在 Windows PowerShell 里输入：<br>
```cmd
wsl --shutdown
```
## ZSH#
习惯了 zsh 作为默认终端，sudo apt install zsh 就行，再找个 oh-my-zsh 的配置贴跟着走完就能得到一个够用的终端。<br>

如：[https://dev.to/equiman/zsh-on-windows-with-wsl-1jck](https://dev.to/equiman/zsh-on-windows-with-wsl-1jck)<br>
不过这篇推荐的用来适配终端的 font 不太好看，个人还是喜欢 JetBrains Nerd Font Mono 字体。<br>

## 其它配置#
我的需求是配置深度学习环境，没什么可讲的，跟你在纯正 Linux 服务器上操作一样，miniconda+uv pip install 即可。<br>

如果这里 pip 下载时提示你当前在用 root 账户操作的话，注意本文前面提到的<br>

## 尾声#
感觉 WSL2 确实节省了不少精力，也能便调配方、炼丹边摸鱼，这样的 Windows 才是我心中理想的操作系统，Mb Air 都放了两天了，除非出门根本不会用。<br>

至于炼丹速度，能跑起来就行了，网上说的 20% 左右的损耗只要不去想就没事，越爱比较烦恼越多～<br>

# 我的补充:让wsl2共享主机的代理设置

## 让wsl2的ubuntu访问外网：

参考:[https://blog.csdn.net/qq_43411964/article/details/127643198](https://blog.csdn.net/qq_43411964/article/details/127643198)

github,miniconda,每个都需要代理，但以前我有GUI的时候我linux代理都翻不利索，现在只有终端更不太行。所以第一思路是，如何让它直接走主机代理。<br>

这里我使用的是V2rayN的http代理，所以我需要让wsl2的ubuntu走主机的代理。<br>

### V2ray设置:

设置->参数设置->core:基础设置:<br>

~->允许来自局域网的连接<br>
~->本地socks.监听端口：你需要记下该端口<br>

`win+r:`<br>
```cmd
%UserProfile%
```

vim ~/.wslconfig<br>
```cmd
[wsl2]
export https_proxy=http://xxx.xxx.xxx.xxx:10808
```

这个似乎10808和10809都可以，但是10808是socks的端口，10809是http的端口。<br>

另外如果你碰到了类似端口拒绝连接的错误。<br>

检查v2ray的配置，看看是不是允许了局域网连接。有时候会被重置。<br>

另外一个是，需要设置你的网络连接为专用网络，我在用热点的时候，网络似乎是公用网络，导致我排查很久也不知道为什么一直拒绝连接。<br>

可以在设置->网络和internet->属性->专用网络  修改。<br>

xxx那个是你的本机IPV4地址。<br>

ipconfig查看,一般是WLAN的，如果有接网线就看以太网的。<br>

或者设置->网络和internet->属性->IPv4地址<br>

之后重启wsl2:<br>
```cmd
wsl --shutdown
```

再进入ubuntu:<br>
```cmd
curl www.google.com
```

如果可以打印出来一堆奇奇怪怪的字符，那就是成功了。然后你也随便git clone一下试试。<br>

## wsl2似乎是支持GUI的，而且可以直接共享到windows。

我在linux上面测试我的一个pyqt程序，启动的window竟然会直接到我的windows里。这可比ssh连接，远程桌面，双系统方便多了。<br>
