---
title: 部署Hexo到云服务器
tags:
  - Hexo
  - 云服务器
categories:
  - 网站搭建
headimg: https://pic.zeng.cyou/wide/202409042025522.webp
abbrlink: 58145
date: 2024-09-07 11:47:57
---

## 购买域名

这里的推荐两个购买平台：

+ [NameSilo](https://www.namesilo.com/?rid=f441249go)：国外的，界面都是英文，使用起来可能有一定的上手难度
+ [雨云](https://www.rainyun.com/cyou_?s=xxx)：国内的，界面简单友好，推荐新手使用

这里不会讲解购买域名的流程。

如果使用`NameSilo`来购买域名，推荐把域名解析到`CloudFlare`，然后用`Cloudflare`来管理域名，因为`NameSilo`的管理界面过于简陋了。

解析的教程这里也不会展开说明，不过可以给一个大概的流程：

1. 注册Cloudflare账号，然后托管要绑定的域名，选择免费的计划
2. 来到NameSilo的域名管理界面，点击进入NameServer Manager（Options栏下最右边的图标）
3. 删除默认的两个NameServer，把Cloudflare分配的NameServer复制上去，之后点击提交按钮

## 购买云服务器

这里以[雨云](https://www.rainyun.com/cyou_?s=xxx)平台作为使用教程：

1. 我们注册好账号后，点击首页的`云服务器`，然后选择一个`区域`，如果不想备案的可以选择`香港`和`美国`

![202409071440358](https://pic.zeng.cyou/post/58145/202409071440358.webp)

2. 然后根据自己实际情况选择一个配置（注意看套餐，有两种：`流量叠加型`、`流量不限型`）

![202409071440359](https://pic.zeng.cyou/post/58145/202409071440359.webp)

3. 选择相应的配置后，点击购买即可

![202409071440360](https://pic.zeng.cyou/post/58145/202409071440360.webp)

## 域名解析

我们需要把购买的域名和服务器进行一个绑定，这样之后就可以通过域名来直接访问部署的网站。

登录`Cloudflare`（其他的平台操作步骤都一样），然后点击之前托管好的域名，找到`DNS`，添加两条记录。

![202409071440361](https://pic.zeng.cyou/post/58145/202409071440361.webp)

![202409071440362](https://pic.zeng.cyou/post/58145/202409071440362.webp)

## 部署网站

### 远程连接

推荐几个SSH远程连接工具：

+ [Termius](https://termius.com)

+ [Xshell](https://www.netsarang.com)

+ [MobaXterm](https://mobaxterm.mobatek.net)

+ [FinalShell](https://www.hostbuf.com)

打开SSH工具后，输入服务器的`IP`地址和`root`账号密码，就可以远程连接服务器了。

### 环境搭建

这里的所有环境都安装在`Docker`容器里，所以需要提前把`Docker`安装好：

+ [Docker官方文档](https://docs.docker.com)
+ [Docker快速入门](https://blog.zeng.cyou/posts/2763)

然后我们就可以安装`Nginx`来部署我们的网站：

1. 创建挂载的目录

```shell
mkdir -p /data/nginx/{conf,html,logs}
```

2. 先启动一次容器

```shell
docker run -d --name nginx nginx
```

3. 把容器里面的内容复制出来

```shell
docker cp nginx:/etc/nginx/nginx.conf /data/nginx/conf/nginx.conf
docker cp nginx:/etc/nginx/conf.d /data/nginx/conf/conf.d
docker cp nginx:/usr/share/nginx/html /data/nginx
```

4. 删除容器

```shell
docker rm -f nginx
```

5. 创建docker-compose.yml

```she
vim /data/nginx/docker-compose.yml
```

```yaml
services:
    nginx:
        image: nginx
        restart: unless-stopped
        volumes:
            - /data/nginx/html:/usr/share/nginx/html
            - /data/nginx/logs:/var/log/nginx
            - /data/nginx/conf/cert:/etc/nginx/cert
            - /data/nginx/conf/conf.d:/etc/nginx/conf.d
            - /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
        ports:
            - 80:80
            - 443:443
        hostname: nginx
        container_name: nginx
```

> 如果你需要开启`HTTPS`就映射`443`端口，不开启`HTTPS`就不用映射`443`端口

6. 启动容器

```shell
docker-compose up -d
```

### Nginx

如果要开启`HTTPS`，雨云提供了免费的`SSL证书`，可以直接去申请（不展开说明，自己研究...傲娇.ing）。

把申请到的`SSL证书`放到`/data/nginx/conf/cert`这个目录下面。

再去把服务器的`80`和`443`端口打开（一般都是在防火墙规则里面）。

![202409071440363](https://pic.zeng.cyou/post/58145/202409071440363.webp)

开启后，我们就可以去编写`nginx.conf`文件：

```conf
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log notice;
pid /var/run/nginx.pid;

events {
    worker_connections 2048;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main
    '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;

    keepalive_timeout 65;

    gzip on;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen 443 ssl;

        #填写证书绑定的域名
        server_name zeng.cyou;

        #填写证书文件名称
        ssl_certificate cert/blog/full_chain.pem;

        #填写证书私钥文件名称
        ssl_certificate_key cert/blog/private.key;

        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout 5m;

        #默认加密套件
        ssl_ciphers HIGH:!aNULL:!MD5;

        #表示优先使用服务端加密套件,默认开启
        ssl_prefer_server_ciphers on;

        location / {
            root /usr/share/nginx/html/blog/;
            index index.html index.htm;
        }
    }

    server {
        listen 80;
        #填写证书绑定的域名
        server_name zeng.cyou;
        #将所有HTTP请求通过rewrite指令重定向到HTTPS
        rewrite ^(.*)$ https://$host$1;
    }
}
```

> 这里你可以通过`Git`的方式去管理，把`Hexo`生成的静态文件托管到`Github`，然后在通过`git pull`的方式把文件拉取到服务器

把上面的域名进行修改，然后注意存放证书的路径不要错，然后把`Hexo`生成的静态文件丢到`/data/nginx/html`这个路径下面，最后重启`Nginx`容器。

```shell
docker restart nginx
```
