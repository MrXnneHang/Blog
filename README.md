server {
  listen 443;
  server_name xnnehang.top;
  ssl on;

  ssl_certificate     xnnehang.top.pem;
  ssl_certificate_key xnnehang.top.key;

  ssl_session_timeout 30m;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;

  location / {
    proxy_pass http://127.0.0.1:8079/;
  }
}





sudo fuser -k 8079/tcp
sudo fuser -k 8080/tcp
sudo fuser -k 8090/tcp

sudo systemctl restart nginx

sudo docker stop redis
sudo docker rm redis
sudo docker run -p 6379:6379 --name redis -v /data/redis/redis.conf:/etc/redis/redis.conf -v /data/redis/data:/data -d redis:5.0.5 redis-server /etc/redis/redis.conf --appendonly yes
sudo service mysql start
source /root/.nvm/nvm.sh
nvm use 14
cd /www/wwwroot/xnnehang.top/blog-view
nohup node server.js &

cd ../blog-cms
nohup node server.js &

cd ../blog-api
nohup java -jar target/blog-api-0.0.1.jar > /www/wwwroot/xnnehangblog.com/blog-api/output.log 2>&1 &

