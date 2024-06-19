---
typora-copy-images-to: upload
---

![image-20240617200934680](https://s2.loli.net/2024/06/19/TFMZL62EUDh45bz.jpg)



*说在前面，我建议，你可以从一个新的镜像开始，并且是centos系列的，我的是centos7.如果这样，你会省去很多功夫，也可以跳过那些移除旧版本的内容。*

### Step1. 安装并且配置前置Redis 和 MySQL,JDK1.8,Maven,Nodejs(By nvm)

Redis: <br>
我的建议是用docker来管理和启动redis。 <br>
而且要pull特定redis也更容易。  <br>
**1.docker的安装和配置:**<br>
centos <br>
卸载历史版本: <br>

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```

然后，我建议使用宝塔面板，点击docker，安装即可。仓库什么的宝塔会配置好。<br>

---

**2.MYSQL安装** <br>
如果是新开的镜像，直接宝塔面板数据库里的MYSQL，不必配置。直接完成一半的工作。 <br>

```
sudo service mysql start # MySQL启动
```

---


**3.redis安装和配置** <br>

```
docker search redis
docker pull redis:5.0.5
```

不建议使用高版本的redis。因为涉及到配置redisconfig，不同版本的conf不同。修改这些内容。<br>

```
bind 127.0.0.1 #注释掉这部分，使redis可以外部访问
daemonize no#用守护线程的方式启动
requirepass 123456#给redis设置密码
appendonly yes#redis持久化　　默认是no
```

官方下载地址:<br>
https://redis.io/docs/latest/operate/oss_and_stack/management/config/ <br>
选择5.0的下载即可。<br>
配置完后。创建目录: <br>
/data <br>
/data/redis <br>

```
cp redis.conf /data/redis/
```

复制配置文件到指定目录。 <br>
运行redis:

```
sudo docker stop redis #如果有启动中的redis
sudo docker rm redis #就关闭该容器
sudo docker run -p 6379:6379 --name redis \
-v /data/redis/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data -d redis:5.0.5 \
redis-server /etc/redis/redis.conf --appendonly yes
```

---

**4.jdk1.8的安装:** <br>
我们使用作者搭建项目用的java版本。防止构建项目后因为版本问题无法运行代码。<br>
在这之前，查看自己是否安装过其他版本:<br>

```
rpm -qa |grep java
```

如果像这样:<br>

```
[root@localhost ~]$ rpm -qa |grep java

java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64
javapackages-tools-3.4.1-11.el7.noarch
java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
nuxwdog-client-java-1.0.3-2.el7.x86_64
java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
mysql-connector-java-5.1.25-3.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
tzdata-java-2015g-1.el7.noarch
javassist-3.16.1-10.el7.noarch
java-1.7.0-openjdk-devel-1.7.0.91-2.6.2.3.el7.x86_64
javamail-1.4.6-8.el7.noarch
```

那么你需要手动把这些都移除掉。<br>
**当然你要知道自己在干什么:** 如果你能判断每个名称是什么，那么只需要移除掉openjdk相关的就可以。javascript就大可不必。但我们讨论的是纯小白的情况，那么就都移除。当然如果是小白，我更建议从0开始，也就是从一开始新建一个镜像来做这些。<br>
你可以像这样移除: <br>

```
rpm -e --nodeps xxx #替换xxx
# 比如这样:
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
```

<br>
关于下载jdk:<br>
我建议到官网自己下载，寻找适合自己系统的版本，我下了别人百度网盘的因为一个arm和amd的问题卡了相当久，只是因为为了节省官网注册和填写信息的五分钟。<br>
官网:<br>
https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html<br>
下载后我是解压到/home/local/java/下。<br>
ps:如果你不会解压的话，你可以考虑使用宝塔面板。 <br>
然后配置~/.bashrc ，直接加入到最后一行之后:<br>

```
export JAVA_HOME=/home/local/java/jdk1.8.0_271
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

你需要改的是**jdk1.8.0_271**后面的数字，把它改成你解压后的文件夹的名字。 <br>
之后在终端运行:

```
source ~/.bashrc
java -version
```

---

**5.Maven的安装:**
我提到了我忘记了如何安装maven，也许是因为我没有在这里碰壁。我似乎是gpt直接给了我安装代码。<br>
你也可以参考这个: <br>
https://blog.csdn.net/qq_38738510/article/details/105567513 <br>

**6.Nodejs的版本管理和切换:**  
这个人讲得非常好，而且还很可爱，还附带文字教程。    
url:*https://www.bilibili.com/video/BV12h411z7Kq/?spm_id_from=333.880.my_history.page.click&vd_source=d7601f0fc447d708fff71aa75186ea10*   

如果到这里都没有报错，那么恭喜你。我在这些上面就已经花了一个晚上和一个下午的时间。 <br>


### Step2.启动前置MYSQL和redis服务器:

如果你已经完成前面的配置，那么可以直接运行:<br>

```
Step2: 启动前置. redis 和 MYSQL
为你整理可以配置开机自启动的几行。

sudo docker stop redis
sudo docker rm redis
sudo docker run -p 6379:6379 --name redis -v /data/redis/redis.conf:/etc/redis/redis.conf \
 -v /data/redis/data:/data -d redis:5.0.5 redis-server /etc/redis/redis.conf --appendonly yes
sudo service mysql start
```

这个可以设置成每次开机自动运行。 <br>


### Step3.创建和修改配置文件:

我们的目的是配置前台后台端口，后端服务端口。然后编写server.js启动前台后台。 <br>
我的配置是，8079前台页面端口，8080后台页面端口，8090后端的端口api。<br>
**首先是后端blog-api中的文件夹配置：** <br>
**1.blog-api/src/main/resources/application-dev.properties:** <br>

```
# 服务器端口号
server.port=8090
# 博客名称
blog.name=Naccl's Blog
# 生产环境需要修改为服务器ip或域名
# 后端服务URL
blog.api=http://xxx.xxx.xxx.xxx:${server.port}
# 后台管理URL https://admin.naccl.top
blog.cms=http://xxx.xxx.xxx.xxx:8080
# 前台界面URL https://naccl.top
blog.view=http://xxx.xxx.xxx.xxx:8079
# 更改xxx为你的服务器公网ip，我们目前配置公网ip访问。

# 数据库连接信息
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/nblog?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root  #保持root
spring.datasource.password=root  #M改成MYSQL的root密码.
# Redis连接信息
# 默认配置，因为我之前示例里面启动redis的时候设置密码为123456，如果你设置不同，用你的密码。
spring.redis.host=localhost
spring.redis.password=123456
spring.redis.port=6379
spring.redis.database=0
spring.redis.timeout=10000ms

```

暂时更改以上部分即可，保证除了我让你更改的内容，其余的你都和我一致。<br>
**关于数据库导入:**<br>
原仓库作者留下了一个**nblog.sql**在blog-api中。<br>
我是使用宝塔面板,创建一个叫nblog的数据库，密码无所谓。然后管理，把sql文件里面的内容复制到宝塔面板的sql执行页面。然后nblog的数据库就会被初始化成作者设定的样子。<br>
**关于服务端口的打开：**<br>
你可以使用比如腾讯云自带的安全组去新建规则添加8079,8080,8090,tcp.<br>
或者使用firewall指定打开然后reload。但是我不会写，你可以去问gpt.<br>

---

**2.blog-api/src/main/java/top/naccl/config/WebConfig.java:** <br>

```
package top.naccl.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import top.naccl.config.properties.UploadProperties;
import top.naccl.interceptor.AccessLimitInterceptor;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    AccessLimitInterceptor accessLimitInterceptor;
    @Autowired
    UploadProperties uploadProperties;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://xxx.xxx.xxx.xxx:8080", "http://xxx.xxx.xxx.xxx:8079")
                .allowedHeaders("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "HEAD", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600);
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(accessLimitInterceptor);
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler(uploadProperties.getAccessPath())
                .addResourceLocations(uploadProperties.getResourcesLocations());
    }
}

```

以上是我的配置，把xxx.xxx.xxx.xxx改成你的公网Ip.<br>

配置完这些后你可以尝试，回到blog-api目录下,然后运行:<br>

```
mvn clean install #这个可能要挺久，耐心，如果没有报错的话。
mvn package
```

**配置blog-view的前端内容.** <br>
**3.创建blog-view/server.js:** <br>

我的内容: <br>

```
const express = require('express');
const cors = require('cors');
const path = require('path');
const app = express();

const corsOptions = {
    origin: 'http://<你的服务器公网IP>:8079',
    methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
    credentials: true,
    optionsSuccessStatus: 204
};

app.use(cors(corsOptions));
app.use(express.static('./dist'));

app.get('*', function(req, res) {
    res.sendFile(path.resolve(__dirname, 'dist', 'index.html'));
});

const port = 8079;

app.listen(port, function (err) {
    if (err) {
        console.log(err);
        return;
    }
    console.log('Listening at http://localhost:' + port + '\n');
});

```

老规矩，改xxx. <br>

---

**4.修改blog-view/src/plugins/axios.js:** <br>
直接给出代码，其他部分保持不变即可： <br>

```
[root@VM-20-6-centos blog-view]# cat src/plugins/axios.js 
import axios from "axios";
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'

const request = axios.create({
        baseURL: 'http://xxx.xxx.xxx.xxx:8090/',
        timeout: 10000,
})
```

你需要修改的主要是baseURL的xxx和那个端口改成8090对应我们blog-api的地址。 <br>

---

**5.创建blog-cms/server.js:** <br>
server.js: <br>

```
const express = require('express');
const cors = require('cors');
const path = require('path');
const app = express();

const corsOptions = {
    origin: 'http://<你的服务器公网IP>:8079',
    methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
    credentials: true,
    optionsSuccessStatus: 204
};

app.use(cors(corsOptions));
app.use(express.static('./dist'));

app.get('*', function(req, res) {
    res.sendFile(path.resolve(__dirname, 'dist', 'index.html'));
});

const port = 8079;

app.listen(port, function (err) {
    if (err) {
        console.log(err);
        return;
    }
    console.log('Listening at http://localhost:' + port + '\n');
});

```

修改xxx. <br>

**6.修改blog-cms/src/util/request.js:** <br>

```
[root@VM-20-6-centos blog-cms]# cat src/util/request.js 
import axios from 'axios'
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'
import {Message} from 'element-ui'

const request = axios.create({
        baseURL: 'http://xxx.xxx.xxx.xxx:8090/admin/',
        timeout: 5000
})
```

这个就是文件开头,保持其他部分不变，修改xxx,修改成8090. <br>

---

## Step4.build项目.

**1.blog-view&blog-cms:**  

我演示一遍:   
我删除掉Build生成的dist和install生成的node_modules来让我和你的情况一致。   

```
[root@VM-20-6-centos blog-view]# ls
babel.config.js  node_modules  package-lock.json  README.md  src
dist             package.json  public             server.js  vue.config.js
[root@VM-20-6-centos blog-view]# rm dist/ node_modules/ -rf
[root@VM-20-6-centos blog-view]# ls
babel.config.js  package-lock.json  README.md  src
package.json     public             server.js  vue.config.js
```

你会看到说它说存在很多问题,但是我们可以不管。我试图去修复问题后反而无法Build项目，而我在这种情况下直接build，反而不会报错。   

```
[root@VM-20-6-centos blog-view]# nvm use 14
Now using node v14.21.3 (npm v6.14.18)
[root@VM-20-6-centos blog-view]# npm install
> core-js@2.6.11 postinstall /home/local/NBlog/blog-view/node_modules/babel-runtime/node_modules/core-js
****
added 1363 packages from 586 contributors and audited 1367 packages in 28.676s

87 packages are looking for funding
  run `npm fund` for details

found 198 vulnerabilities (2 low, 117 moderate, 59 high, 20 critical)
  run `npm audit fix` to fix them, or `npm audit` for details


   ╭────────────────────────────────────────────────────────────────╮
   │                                                                │
   │      New major version of npm available! 6.14.18 → 10.8.1      │
   │   Changelog: https://github.com/npm/cli/releases/tag/v10.8.1   │
   │               Run npm install -g npm to update!                │
   │                                                                │
   ╰────────────────────────────────────────────────────────────────╯
```

我们直接npm run build:    (在这之前npm install cors)记得。

```
[root@VM-20-6-centos blog-view]# npm run build

> blog-view@1.0.0 build /home/local/NBlog/blog-view
> vue-cli-service build
*****
  Images and other types of assets omitted.

 DONE  Build complete. The dist directory is ready to be deployed.
 INFO  Check out deployment instructions at https://cli.vuejs.org/guide/deployment.html
```

可以看到成功build.   
你可以用   

```
nohup node server.js &    
```

来启动我们的页面，因为我的服务器正在运行，会端口冲突我就不再启动一个了。这个指令有个问题，当你关闭当前终端，它就会退出，并不是持久化运行。   
**关于后台页面blog-cms:**   
你同样可以这么操作:   

```
npm install
npm install cors
npm install path
npm run build
nohup node server.js &    
```

如果你们碰到Exit报错，可以cat nohup.out看一下，然后问问gpt.   
对了如果npm build 没错而运行server.js报错可以考虑运行以下代码进行修复：   

```
npm install cors ##我后来加入了cors来防止一些跨节点无法访问的情况。
npm install path ##用于刷新页面后重定向到页面。
rm -rf dist/
npm run build ## 每次修改代码，增加东西最好都build一下。build前记得删除dist/
```

---


**2.blog-api的build和运行:**   

```
mvn clean install
mvn package
nohup java -jar target/blog-api-0.0.1.jar > output.log 2>&1 &
```

这个时候你去访问ip:8079 应该就可以看到自己的博客了，8080的用户名Admin,密码123456。如果你记得你有导入blog-api下的数据库文件sql。那么你就可以正常使用blog.

大部分的报错是数据库引起的，可能是MYSQL服务没启动，或者root密码配置错误。对了，我给你写了两个注释，会引起乱码。（root #Ã¤Â¿Â�Ã¦ÂŒÂ�root'@'localhost）

cat /www/wwwroot/xnnehang.top/blog-api/target/surefire-reports/*.txt
改成你的路径，看看是不是这个原因。

## 5.Step5.开机自启动:

我希望每次reboot的时候可以自动运行这个nblog，而运行的东西太复杂，我考虑将它加入到开机自启动。
这里是centos的演示:

```
sudo chmod +x /etc/rc.d/rc.local
vim /etc/rc.d/rc.local
```

把下方所有代码加入到rc.local末行以下：   

sudo docker stop redis <br>
sudo docker rm redis <br>
sudo docker run -p 6379:6379 --name redis -v /data/redis/redis.conf:/etc/redis/redis.conf -v /data/redis/data:/data -d redis:5.0.5 redis-server /etc/redis/redis.conf --appendonly yes<br>
sudo service mysql start<br>
source /root/.nvm/nvm.sh<br>
nvm use 14<br>
cd /www/wwwroot/xnnehang.com/blog-view/<br>
nohup node server.js &<br>
cd ../blog-cms<br>
nohup node server.js &<br>
cd ../blog-api<br>
nohup java -jar target/blog-api-0.0.1.jar > /www/wwwroot/xnnehang.com/blog-api/output.log 2>&1 &<br>
cd /www/wwwroot/xnnehang.com/<br>

---

## 6.Step6 关于域名和图床和写文章:

我目前已经够用了，但关于域名的购买，我暂时还没打算。图床我还没研究，因为暂时没用到。如果你后续有用到，欢迎留下你的做法。   
写文章，它有些烦的是每个框内的内容必须填满，以及我碰到过一次参数错误的报错，应该是文章首图URL的锅，我给出的解法是，当我把所有内容都调成test，那么我的文章就能发出去。我就发现文章首图url似乎没影响，我后来都是填我写文章的日期了。   

---
## 域名配置：
### 代码配置:
emmmm。  
其实基本上代码上没有改变。
以我为例：  
xnnehang.top 前台  
admin.xnnehang.top 后台  
xnnehang.top:8090 后端     

将xxx.xxx.xxx.xxx:8079改成前台  
将xxx.xxx.xxx.xxx:8080改成后台  
将xxx.xxx.xxx.xxx:8090改成后端。    
注意两个server.js的app.listen端口依然是localhost+port.  

### 宝塔配置:
**网站：**  
创建两个，一个xnnehang.top,一个admin.xnnehang.top。指向你的Nblog masetr目录 。  
**反向代理:**
设置->反向代理。对xnnehang.top:  
> 代理名称:front
> 目标url:http://localhost:8079  发送域名：xnnehang.top

对于admin.xnnehang.top:
>代理名称:admin
>目标url:http://localhost:8080   发送域名:admin.xnnehang.top

### 腾讯云（我的）配置:
**域名那边的配置：**
快速解析,添加一个www,一个@,然后子域名我写了一个admin.  
**查看服务器开放端口:**
80和443需要开通。  

---

到这一步其实配置已经完成了。但是我还无法通过域名访问。  
我纳闷了很久。  
后面，我通过购买了专业版DNS服务器解决。买完后更改DNS服务器，立刻解决。  



### 一些额外的东西:
**ICP备案悬挂到页脚:**
这个我通过后台的页面管理，把copyright的内容直接改成了ICP备案的内容。**不要更改源代码。注意，不要更改源代码**。最开始我在错误的方向越走越远。其实作者写在数据库里了，改起来很方便。  

**f5刷新后会出现not found:**
这里是需要更改cms和view的两个server.js,引入path刷新重定向。示例:  
```
const express = require('express');
const cors = require('cors');
const path = require('path');
const app = express();

const corsOptions = {
    origin: 'http://xnnehang.top',
    methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
    credentials: true,
    optionsSuccessStatus: 204
};

app.use(cors(corsOptions));
app.use(express.static('./dist'));

app.get('*', function(req, res) {
    res.sendFile(path.resolve(__dirname, 'dist', 'index.html'));
});

const port = 8079;

app.listen(port, function (err) {
    if (err) {
        console.log(err);
        return;
    }
    console.log('Listening at http://localhost:' + port + '\n');
});
```
并且鉴于这个是我们自己加的可能不在reqirement中，直接用npm insatll会缺少module，我们采取:
> npm install path

的形式独立安装。和python比较像。同样那个cors也是我加入的，你可以考虑一起安装。
>npm install cors 

之后你就可以安心地build。  