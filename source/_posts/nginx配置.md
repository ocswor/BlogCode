---
title: nginx配置
meta: 'ios,python'
date: 2018-05-21 17:57:34
tags:
categories:
keywords:
---
# Nginx 配置请求端口转发

1.测试场景
    本地django 处理请求，处理过程会请求 其他10.11.255.124服务器，并且有图片保存在这台服务器上。

如果在正式部署到10.11.255.124上则不会出现请求图片，跨服务器问题
   现在本地测试需要正常显示图片。

   则需要nginx在本地监听一个端口做转发，正常的请求转发给 django 关于图片请求转发给后台的服务器。

  具体配置如下：
  
  ```code
   server {
	listen 8000;
	listen [::]:8000;



	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		        proxy_pass  http://localhost:8001;
            	proxy_set_header Host $host;
            	proxy_set_header X-Real-IP $remote_addr;
            	proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
	}

	location /upload/charts {

		proxy_pass http://10.11.255.124:8500;
            	proxy_set_header   Host    $host;
            	proxy_set_header   X-Real-IP   $remote_addr;
            	proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
	}
  ```
  8001就是本地 django 服务器的端口。
  8500就是远程图片存储的位置。