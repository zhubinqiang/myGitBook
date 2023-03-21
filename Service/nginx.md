# nginx

[TOC]

## 安装
```sh
sudo apt install -y nginx
```

## 配置
配置文件在 /etc/nginx/nginx.conf


## 命令
```sh
sudo systemctl start nginx.service
sudo systemctl stop nginx.service
sudo systemctl restart nginx.service
sudo systemctl status nginx.service


sudo nginx -s stop

sudo nginx -s quit

sudo nginx -s reload

## test configuration
sudo nginx  -t
```

## 三大块
nginx配置文件分为三大块[^3block]，**全局块**，**events块**，**http块**

全局块:
主要会设置一些影响 nginx 服务器整体运行的配置指令，
主要包括配 置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，
进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

events块:
events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，
常用的设置包括是否开启对多 work process下的网络连接进行序列化，
是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，
每个 word process 可以同时支持的最大连接数等。

http块:


开启目录[^dir]
```
location / {   
    root /data/www/file                     //指定实际目录绝对路径；   
    autoindex on;                            //开启目录浏览功能；   
    autoindex_exact_size off;            //关闭详细文件大小统计，让文件大小显示MB，GB单位，默认为b；   
    autoindex_localtime on;              //开启以服务器本地时区显示文件修改日期！   
}
```


在 /etc/nginx/sites-available/default 配置虚拟路径
```
    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    location /wiki {
        alias  /home/user/WS/repos/wiki/_book/;
        index  index.html index.htm;
        # root  /home/user/WS/repos/wiki/_book/;
    }
```


[^3block]: 作者：韩数 https://juejin.cn/post/6844903983622914062
[^dir]: https://developer.aliyun.com/article/327244


