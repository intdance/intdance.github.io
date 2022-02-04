---
layout: post
title: "Nginx 根据子域名请求不同的目录资源"
date: 2022-01-25
categories: [ "Nginx" ]
tags: [ "Nginx" ]
---

* content
{:toc}

## 需求场景
在日常开发过程中，经常需要将前端资源部署到服务器上，并通过 Nginx 做代理，返回静态资源。团队合作时多个人会部署多套测试环境，通过子域名的方式，每个人可以部署属于自己的测试环境。
例如：有一个统一的域名 `xxx.com` 作为测试域名，每个人部署的测试环境可以为：`a.xxx.com` 、`b.xxx.com`。

下面介绍如何将子域名的请求映射至不同的文件夹。

## Nginx 配置

配置如下：

```nginx
server {
	listen 8500;
	server_name ~^(?<subdomain>.+)\.xxx\.com;

	location /frontend/ {
		alias /data/web_static/$subdomain/;
	}
}
```
- Nginx 配置 `server_name`时，支持使用正则表达式，即以 `~` 作为前缀即可使用正则表达式表示域名匹配规则。
- 正则表达式中的命名捕获组可以作为变量在 `location` 中使用。

上述正则表达式中，`(?<subdomain>.+)` 表示一个命名捕获组，名字为 `subdomain`，因此可以在 `location` 中以 `$domain` 使用。

我们事先将前端资源上传至服务器的某个目录下，此时保证目录名与子域名相同即可，例如想通过 `zhangsan.xxx.com` 访问前端资源，把构建好的前端资源上传至 `/data/web_static/zhangsan` 目录下即可。

然后在 `location /frontend/` 中配置前端资源的请求路径：
```nginx
location /frontend/ {
    alias /data/web_static/$subdomain/;
}
```
上述配置表示 `/frontend/` 开头的资源都根据请求的子域名去请求 `/data/web_static/$subdomain/` 下的资源。例如请求 `http://zhangsan.xxx.com/frontend/index.html` 时，会返回 `/data/web_static/zhangsan/index.html`.

这样即可实现按照子域名的不同访问不同的目录资源，每个人也就有了自己独立的测试环境。



以上就是本文的全部内容了，感谢各位阅读，如果有任何疑问，欢迎电子邮件留言。

