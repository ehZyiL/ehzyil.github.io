---
title: Docker安装Nginx
author: ehzyil
tags:
  - docker
  - git
categories:
  - 技术

date: 2024-01-09 20:22:46
headimg:
---

###  创建挂载目录

```
mkdir -p /home/ehzyil/data/docker_data/nginx/conf
mkdir -p /home/ehzyil/data/docker_data/nginx/log
mkdir -p /home/ehzyil/data/docker_data/nginx/html
```

### 拉取docker镜像

```
docker pull nginx:latest
```

### 创建Nginx容器并运行

```
docker run -d --name nginx -p 80:80 -v /home/ehzyil/data/docker_data:/data nginx
```

### 容器中的nginx.conf文件、conf.d文件和html文件夹复制到宿主机

```

docker cp nginx:/etc/nginx/nginx.conf /home/ehzyil/data/docker_data/nginx/

docker cp nginx:/etc/nginx/conf.d /home/ehzyil/data/docker_data/nginx/

docker cp nginx:/usr/share/nginx/html /home/ehzyil/data/docker_data/nginx/
```

### 关闭并删除Nginx容器

```
# 关闭该容器
docker stop nginx
# 删除该容器
docker rm nginx
```

### 创建Nginx容器，指定挂载点和端口号并运行

```
docker run -p 81:80 --name nginx -v   /home/ehzyil/data/docker_data/nginx/nginx.conf:/etc/nginx/nginx.conf -v  /home/ehzyil/data/docker_data/nginx/conf.d:/etc/nginx/conf.d -v /home/nginx/log:/var/log/nginx -v  /home/ehzyil/data/docker_data/nginx/html:/usr/share/nginx/html -d nginx:latest
```



### Nginx默认配置(备份)

```

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;a

    include /etc/nginx/conf.d/*.conf;
}

```





