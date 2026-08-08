---
title: Ubuntu 26.04自编译安装Nginx遇到的坑
date: 2026-08-08 16:00:00
updated: 2026-08-08 17:00:00
---

## PCRE库缺失

在进行 `config` 配置时会遇到，解决办法见→[PCRE库缺失问题](/PCRE)

## getpwnam("nginx") failed

这个主要是因为在 `config` 阶段配置了

```sh
--user=nginx --group=nginx
```

只要将nginx添加进用户组即可

```sh
useradd -s /sbin/nologin nginx
```

## mkdir() "/var/cache/nginx/client_temp" failed

其实是nginx没有足够权限创建这个目录，运行这个就好了

```sh
# -pv 意思是逐层创建文件夹
mkdir -pv /var/cache/nginx/client_temp
```

## 附上Nginx常用的配置

- config编译配置：

```sh
./configure --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --modules-path=/etc/nginx/modules \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --with-compat \
    --with-file-aio \
    --with-threads \
    --with-http_addition_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_realip_module \
    --with-http_slice_module \
    --with-http_sub_module \
    --with-http_v2_module \
    --with-stream \
    --with-stream_realip_module \
    --with-cc-opt='-O2 -g -pipe -Wall -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' \
    --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

- nginx.conf

```nginx
#user  nobody;
worker_processes auto;
worker_cpu_affinity auto;
worker_rlimit_nofile 1024;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
    use		epoll;
    accept_mutex on;
    multi_accept on;

}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;
    proxy_hide_header Server;
    proxy_hide_header X-Powered-By;
    server_tokens off;

    gzip  on;
	gzip_min_length 1k;
	gzip_buffers 32 4k;
	gzip_comp_level 1;
	gzip_types text/plain application/javascript application/x-javascript text/css application/xml application/json text/javascript application/x-httpd-php;
	gzip_vary on;

	include /etc/nginx/conf.d/*.conf;

}
```

- /usr/lib/systemd/system/nginx.service

```txt
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /var/run/nginx.pid)"
ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /var/run/nginx.pid)"

[Install]
WantedBy=multi-user.target
```
