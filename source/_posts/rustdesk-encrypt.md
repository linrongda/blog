---
title: RustDesk非安全连接问题
date: 2026-10-26 00:00:00
updated: 2026-10-26 01:00:00
excerpt: 分析 RustDesk 显示非安全连接的原因，介绍如何通过修复服务器端密钥解决问题。
tags:
  - RustDesk
  - Ubuntu
  - Docker
categories:
  - 运维
---

## 问题

在RustDesk客户端连接别的客户端时，总是弹窗 **非安全连接**

具体信息为：

```txt
无法验证端到端加密。
远程设备可能仍在准备中，请稍后重试。
如果此问题持续出现，服务器可能不受信任。
仍要继续吗？
```

## 问题分析

根据弹窗信息来分析：

### 1. 远程设备可能仍在准备中

这个应该是偶尔因为网络问题出现，总是出现可能性不大。

### 2. 服务器可能不受信任

这个是最有可能的，大概率是设置了KEY参数（公钥），但是服务器里没有对应的私钥导致的。我排查了很久才发现是这个原因。RustDesk需要一对完整的密钥对来验证加密。

{% callout type="info" %}
网上都搜不到这个问题，难道没人出现过？
{% endcallout %}

{% callout type="warning" %}
不要忘记备份密钥对！！！也不要只备份KEY！！！
{% endcallout %}

## 解决办法

设置KEY参数为`_`，这个参数会自动生成一对密钥对**或**自动识别服务器里存的密钥对。

{% callout type="warning" %}
只不过这样就得在所有客户端都更换KEY了，这也没办法。。。
{% endcallout %}

例如在docker-compose中：

```yaml
services:
  hbbs:
    container_name: hbbs
    image: rustdesk/rustdesk-server:latest
    command: hbbs -k _ # 就像这样
    environment:
      - "ENCRYPTED_ONLY=1" # 强制加密
    volumes:
      - /home/rustdesk/root:/root # 这样挂载就能保留和识别密钥对和数据了
    networks:
      - rust_host
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    image: rustdesk/rustdesk-server:latest
    command: hbbr -k _
    environment:
      - "ENCRYPTED_ONLY=1"
    volumes:
      - /home/rustdesk/root:/root
    networks:
      - rust_host
    restart: unless-stopped
```
