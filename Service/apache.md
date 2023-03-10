# apache

[TOC]

## 安装
```sh
sudo apt install -y apache2
```

## 配置
配置文件: /etc/apache2/apache2.conf

```
Alias /wiki "/home/xxx/WS/repos/wiki/_book"
<Directory /home/xxx/WS/repos/wiki/_book>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

## 重启服务
```sh
sudo systemctl restart apache2.service
```


