[Windows/macOS/Linux上安装Node.js，并使用NVM管理多版本Node.js](https://www.mintimate.cn/2021/07/26/nvmNode/)

这次是打算在windows上面修改我的前端，所以只专注于windows端的。

## 1.下载解压nvm-noinstall.zip并配置

这次选择的是手动配置而非安装的版本。

[github release地址](https://github.com/coreybutler/nvm-windows/releases)

![Snipaste_2024-06-28_08-47-30](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2909961873.webp)

下载完成后解压到一个空目录，比如我的是。

![Snipaste_2024-06-28_09-06-23](https://image.baidu.com/search/down?url=https://img2.doubanio.com/view/photo/l/public/p2909961871.webp)

然后创建打开**settings.txt，不要忘记s**。

写入以下内容，如果目录和我不一样请自行更改:

![Snipaste_2024-06-28_09-12-01](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2909961872.webp)

```
root: D:\env\node\nvm
path: D:\env\node\nodejs
arch: 64
proxy: none

node_mirror: https://npmmirror.com/mirrors/node/
npm_mirror: https://npmmirror.com/mirrors/npm/
```



## 2.添加到系统环境变量。

![Snipaste_2024-06-28_09-08-24](https://image.baidu.com/search/down?url=https://img2.doubanio.com/view/photo/l/public/p2909961881.webp)

先添加NVM_HOME和NVM_SYMLINK两项(这个即使没有指向已存在文件夹也可以。)

![Snipaste_2024-06-28_09-11-39](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2909961883.webp)

然后编辑Path。加入:

```
%NVM_HOME%
%NVM_SYMLINK%
```

**记得重启，注销也可以**



## 3.运行测试。

![Snipaste_2024-06-28_09-13-21](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2909961886.webp)

```
nvm version
nvm install 14
nvm use 14
```

如果下载时出现:

```
ERROR open D:\env\node\nvm\settings.txt: The system cannot find the file specified.
```

检查你的settings.txt文件，有没有少打一个t或者s。