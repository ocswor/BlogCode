---
title: Nginx 负载均衡不宕机重启
meta: 'ios,python'
date: 2018-12-10 15:17:01
tags: Nginx 
categories: 服务器技术
keywords: Nginx
---

### nginx 配置
```
upstream adminProxy{
    server 127.0.0.1:8500 weight=2 max_fails=3 fail_timeout=10s;
    server 127.0.0.1:8501 weight=2  max_fails=3 fail_timeout=10s;
}
server {
    listen       8502;

    location /static {      
        alias /data/www/xy_manager_server/static;                     
    }
    location / {
            proxy_pass http://adminProxy;
            proxy_pass_header       Authorization;
            proxy_pass_header       WWW-Authenticate;

            proxy_http_version 1.1;             
            proxy_set_header Upgrade $http_upgrade;    
            proxy_set_header Connection "upgrade";              
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_next_upstream http_502 http_504 error timeout invalid_header;
            proxy_next_upstream on;
    }

    access_log /var/log/nginx/xy_manager_server_access.log;
    error_log /var/log/nginx/xy_manager_server_error.log;
}
```

重点是 

```
proxy_next_upstream http_502 http_504 error timeout invalid_header;
```

在服务器返回502,504，错误，超时 的时候；允许转发到其他服务器；

另外还需注意

```
server 127.0.0.1:8500 weight=2 max_fails=3 fail_timeout=10s;
```
这里max_fails 表示Nginx尝试连接后端主机失败的次数
fail_timeout 表示在max_fails定义的失败次数后，距离下次检查的间隔时间，默认是10s；

所以这里当重启一个后端服务时，需要间隔10s等待重启的服务，重新上线，再重启另一个服务。

[参考链接](https://blog.csdn.net/libinemail/article/details/51074866)
### supervisor 管理后端服务

这样重启，会等当前会话完毕，再关闭服务。
```
[program:manager_server]
command=/data/www/xy_manager_server/env/bin/gunicorn -k gevent --conf gunicorn.conf xy_manager_server.wsgi
environment=PYTHONPATH=/data/www/xy_manager_server/env
directory=/data/www/xy_manager_server
user=root
autostart=true
autorestart=true
```

也可以使用 启动服务
```
gunicorn -b 0.0.0.0:8501 -w 2 xy_manager_server.wsgi
```

### 关于标准日志输出

Django 将日志输出到标准输出
```
# 日志配置

LOGGING_FILE = os.path.join(BASE_DIR, 'xy_manager_server.log')
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '%(asctime)s [%(levelname)s] [%(process)d:%(threadName)s:%(thread)d] [%(filename)s:%(lineno)d] [%(funcName)s]- %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse'
        },
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'null': {
            'level': 'DEBUG',
            'class': 'logging.NullHandler',
        },
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
        'mail_admins': {
            'level': 'ERROR',
            'filters': ['require_debug_false'],
            'class': 'django.utils.log.AdminEmailHandler'
        },
        'file_handler': {
            'level': 'DEBUG',
            'class': 'logging.handlers.TimedRotatingFileHandler',
            'filename': LOGGING_FILE,
            'when': 'W0',
            'backupCount': 500,
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'manager_server': {
            'handlers': ['console'],
            'propagate': True,
            'level': 'DEBUG',
        },
        'django.request': {
            'handlers': ['mail_admins'],
            'level': 'ERROR',
            'propagate': True,
        },
    }
}

import logging

LOGGER = logging.getLogger('manager_server')
```

gunicorn 不修改日志配置。
supervisor 可以获取到Django 的标准日志输出。

supervisor 默认安装在 /etc/supervisor/

新增配置，需要supervisorctl update 更新配置。注意是否自动重启
```
autostart=true
autorestart=true
```