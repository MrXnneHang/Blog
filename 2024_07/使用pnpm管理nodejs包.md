## 安装：

### 下载：

我是在windows下使用，nodejs14。

所以直接到github上面下了pnpm7。

![Snipaste_2024-07-08_06-35-52](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407080638002.jpeg)

兼容性如下，我也不懂为什么pnpm要高版本丢弃低版本，就好像miniconda不再支持python3.6一样，可能是为了代码不臃肿。

![Snipaste_2024-07-08_06-28-00](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407080639123.jpeg)

### 环境变量配置：

我是直接配置系统环境变量。

![Snipaste_2024-07-08_06-37-12](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407080639531.jpeg)

![Snipaste_2024-07-08_06-36-42](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407080639274.jpeg)

值得注意的是，github上面下载到的命名为pnpm-win-x64.exe。

这样子如果直接复制过去之后无法用pnpm -v来使用。

**我这里把pnpm-win-x64.exe重命名成了pnpm.exe，否则会报'pnpm' 不是内部或外部命令，也不是可运行的程序**

![Snipaste_2024-07-08_06-34-44](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407080639176.jpeg)

### 测试：

最后在cmd中测试:

![Snipaste_2024-07-08_06-40-00](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/07/202407080640101.jpeg)