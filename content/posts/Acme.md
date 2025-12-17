+++
title = "使用 acme.sh 和 Vultr DNS API 申请 SSL 证书"
date = "2025-06-27"
draft = true
tags = ["SSL/TLS"]
categories = ["软件使用备忘"]
+++

安装 acme.sh

```sh
curl https://get.acme.sh | sh -s email=your-email@example.com
```

# 申请证书

```sh
export VULTR_API_KEY="your_vultr_api_key"
acme.sh --issue -d your-domain.com --dns dns_vultr --log
```

# 安装证书

服务器已经安装 nginx，通过 apt 安装，配置目录在 /etc/nginx/ssl

```sh
acme.sh --installcert -d your-domain.com --keypath /etc/nginx/ssl/your-domain.com/your-domain.com.key --fullchainpath /etc/nginx/ssl/your-domain.com/your-domain.com.cer --reloadcmd 'sudo systemctl reload nginx'
```

创建 hdparams.pem 文件

```sh
openssl dhparam -out /etc/nginx/ssl/your-domain.com/dhparams.pem 2048
```

然后配置 nginx:

{{<gist simileas 699e6ca59bbb1f778d20fd80662aa3ef>}}

重载 nginx:

```sh
sudo systemctl reload nginx
```
