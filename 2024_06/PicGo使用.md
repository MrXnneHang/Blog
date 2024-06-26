## PicGO使用中的Bug汇总:

我主要是配合Typora使用。

软件配置参考[使用 GitHub + jsDelivr + PicGo + Typora 搭建图床](https://naccl.top/blog/11)

这里主要提及一下我使用过程中碰到的几个bug：

* **上传失败，网络错误，参数我忘了。**

  在那个配置教程里面，提到说如果碰到这种情况确认端口是不是一致(默认是一致的)，然后把PicGO的Server关了再开，但是我不行，当时用了smms和github图床，都不行，而且是偶尔可以。

  **代理问题：**

  我的clash verge开启了服务模式和Tun模式，这么开我可以正常访问网页，但这两个模式听我室友说是特定节点才支持。而确实，在开启这两个模式的时候，我通常情况下向github进行代码push会报port和ip什么什么错，就是无法推送，将代理改为只有一个系统代理即可。

* **uploading...**

  这不是文件太大。

  我是两个问题，一个是同时开v2ray和clash代理，就会碰到，不清楚为什么。

  另一个是复制粘贴的问题，我习惯以粘贴的形式插入，如果要这么做，你需要在PicGo的设置里面打开**"使用内置剪切板上传"**

  ![Snipaste_2024-06-25_22-32-10](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406252232878.jpeg)

* **PicGO卡死，响应慢。**

  当我uploading太久，我会把它剪切了，然后再粘贴一次。反复几次，PicGO就卡死了。

  这个解决方法同上一个uploading的问题，打开内置剪切板。你就会发现，它变得很乖，很听话。

  你就可以像这样自由地上传啦=-=

![Snipaste_2024-06-25_22-37-00](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406252237998.jpeg)