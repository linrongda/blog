---
title: 新版office图标缩略图如何改回旧版
date: 2026-08-14 22:00:00
updated: 2026-08-14 23:00:00
---

## 背景

2025年10月，微软更新office图标。新版word缩略图与照片缩略图相近，难以辨别。

## 解决方案

1. 用powershell（终端）管理员运行

```powershell
cd "C:\Program Files\Common Files\Microsoft Shared\ClickToRun\"
./officec2rclient.exe /update user updatetoversion=16.0.19231.20194
```

2. 等office部署工具提示完成

3. 去word里把更新禁用

{% callout type="info" %}
目前已知修改图标版本号为19231.20216，19231.20194为未更改的最后一个版本
{% endcallout %}

{% callout type="warning" %}
以上版本号均为office的内部版本号，不影响你实际使用的大版本
{% endcallout %}

## 参考资料

- [Office 更新 - Office release notes | Microsoft Learn](https://learn.microsoft.com/zh-cn/officeupdates/)
- [Office 版本回退 - Aino_D - 博客园](https://www.cnblogs.com/aino-d/p/18991313)
