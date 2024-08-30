cd /www/wwwroot/xnnehangblog.com/blog-view

nvm use 14

rm -rf dist

npm run build





source /root/.nvm/nvm.sh
nvm use 14

fuser -k 4070/tcp

cd /www/wwwroot/xnnehangblog.com/blog-view/
nohup node server.js &

fuser -k 4080/tcp

cd ../blog-cms
nohup node server.js &

fuser -k 4090/tcp

cd ../blog-api
nohup java -jar target/blog-api-0.0.1.jar > /www/wwwroot/xnnehangblog.com/blog-api/output.log 2>&1 &

cd .



---





vim  /www/wwwroot/xnnehangblog.com/blog-view/src/assets/css/base.css

vim  /www/wwwroot/xnnehangblog.com/blog-view/src/components/sidebar/Introduction.vue





![Snipaste_2024-06-28_10-23-13](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406281023389.jpeg)

![Snipaste_2024-06-28_10-24-23](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/06/202406281024596.jpeg)





1.输入未知人脸，提取特征，比较数据库

2.录入人脸，提取特征，存入数据库。



---

activate faces_gpu

D:

cd program\faces-detecter-master

python demo.py

