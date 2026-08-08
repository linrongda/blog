---
title: PCRE库缺失问题
date: 2026-08-08 15:00:00
updated: 2026-08-08 16:00:00
---

## 问题

在阿里云Ubuntu 26.04自编译安装Nginx遇到PCRE库缺失

报错如下：

```sh
./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```

## 解决方法

1. 下载最新PCRE包：https://github.com/PCRE2Project/pcre2/releases
2. 解压进入目录

```sh
tar zxvf pcre2-x.xx.tar.gz
cd pcre2-x.xx
```

3. 编译安装

```sh
./configure
make
make install
```

4. 更新动态链接器缓存

```sh
ldconfig
```

## Q&A

为什么不用 `apt-get install libpcre3-dev`?

因为会遇到报错：

```sh
Package libpcre3-dev is not available, but is referred to by another package. This may mean that the package is missing, has been obsoleted, or is only available from another source.
```
