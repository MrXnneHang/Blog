# 如果你的vscode找不到php.exe解释器:



## like this:

![image-20241012112900105](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410121129261.png)



* 你添加php.exe到系统环境变量，但依然无法运行。

* 你试图调整vscode的插件配置，但是没有找到配置路径的选项。



## Solve:

### 找到five sever ,安装。

![image-20241012113258439](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410121132659.png)

### 根目录创建fiveserver.config.js:

![image-20241012113402260](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410121134348.png)

写入：

```cmd
module.exports = {
    php: "C:\\xampp\\php\\php.exe"   // Windows
  };
```

更改php的路径为你的安装路径。



### 通过open with five server 运行：

![image-20241012113506551](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410121135595.png)



### 进入本地端口（Local）：

![image-20241012113650554](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410121136640.png)



### 如果你依然无法运行：

可能还需要一些php server的依赖。你可以把我装过的相关的依赖插件都装一遍。

![image-20241012113855927](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/10/202410121138008.png)
