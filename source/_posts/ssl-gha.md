---
title: 通过GHA部署SSL至EO
date: 2026-08-09 21:00:00
updated: 2026-08-09 23:00:00
---

## Cloudflare

- 获取token
  1. 进入 https://dash.cloudflare.com/profile/api-tokens
  2. 创建令牌 > 使用“编辑区域 DNS”模板 > 选择域名 > 继续并创建
  3. 复制token保存

- 获取id
  - 域名概述界面右侧 > 账户ID

## acme

1. 安装acme

```sh
curl https://get.acme.sh | sh -s email=my@example.com
```

2. 切换至Let's Encrypt服务

```sh
acme.sh  --set-default-ca  --server letsencrypt
```

3. 在 `~/.acme.sh/account.conf` 中存入

```conf
SAVED_CF_Token='' # 刚复制的token
SAVED_CF_Account_ID='' # 域名界面右侧的账户ID
```

4. 申请证书（必须运行一次，没完成没关系，国内不一定能成功）

```sh
acme.sh --issue  -d example.com  --dns dns_cf
```

5. 打包账户信息

```sh
cd ~/.acme.sh
tar cz ca account.conf | base64 -w0
```

## 腾讯云

- 访问管理 > 用户 > 用户列表 > 新建用户
- 自定义创建 > 可访问资源并接收消息 > 输入用户名、勾选编程访问
- 搜索QcloudSSLFullAccess、QcloudTEOFullAccess并添加
- 完成并保存ID和KEY信息

## GHA

在GitHub仓库中 Settings > Secrets and variables > Actions > New repository secrets

1. 设置 `ACME_SH_ACCOUNT_TAR` 为打包的账户信息base64
2. 设置 `QCLOUD_SECRET_ID` 为腾讯云子用户的ID
3. 设置 `QCLOUD_SECRET_KEY` 为腾讯云子用户的KEY

.github/workflows/ssl.yml配置如下

```yaml
name: Issue & Deploy SSL Certificates

on:
  workflow_dispatch:
  schedule:
    # 每两个月运行一次
    - cron: "0 0 1 */2 *"

jobs:
  ssl-certificate:
    runs-on: ubuntu-latest
    steps:
      - name: Check Out
        uses: actions/checkout@v7

      - name: Issue SSL Certificate
        uses: Menci/acme@v2
        with:
          # 指定 acme.sh 的版本
          version: 3.1.4
          # 以 Base64 编码存储的凭据
          account-tar: ${{ secrets.ACME_SH_ACCOUNT_TAR }}
          # 域名列表，以空格分隔
          domains: lrd0.top *.lrd0.top
          # 是否申请通配符
          append-wildcard: false
          # 传递给 acme.sh 的额外参数
          arguments: --dns dns_cf
          # 导出的证书路径
          output-fullchain: fullchain.pem
          output-key: privatekey.key

      - name: Deploy Certificate to TEO
        uses: linrongda/deploy-certificate-to-tencent-cloud@v2
        with:
          # Use Access Key
          secret-id: ${{ secrets.QCLOUD_SECRET_ID }}
          secret-key: ${{ secrets.QCLOUD_SECRET_KEY }}

          # Specify PEM fullchain file
          fullchain-file: fullchain.pem
          # Specify PEM private key file
          key-file: privatekey.key

          # Deploy to TEO
          domains: |
            zone-XXXXXX lrd0.top
```

## 参考资料

- [使用 GitHub Actions 自动申请与部署 SSL 证书 - 宝硕博客](https://blog.baoshuo.ren/post/actions-ssl-cert/)

- [使用 GitHub Actions 自动申请与部署 ACME SSL 证书 - Menci's Blog](https://blog.men.ci/ssl-with-github-actions/)
