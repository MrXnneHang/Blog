# Blog
存放博客的markdown，markdown中的图片引用SMMS  
博客中的github图片在国内没有代理的情况下几乎加载不出来。  
而偶然间看到的微博图床可以加以利用。  
我们的思路是：  
写markdown的时候自动上传到SMMS，然后我们push到github上来。  
然后我自己再写一个读取markdown中的图片链接并且request到本地然后再upload到微博图床再转义成url替换。  
也就是说github存放我们的博客源文件，然后转义成微博图床上的url再放到我的博客上面。这样即使微博图床挂了我也可以从github的markdown里面重新构建。  
