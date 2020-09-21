# Docker on centos7
[TOC]

参考 [这里](https://docs.docker.com/engine/installation/linux/centos/)

> Docker 需要 64位的系统 和 3.10或以上的kernel版本


## 配置 yum 源
```bash
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```

> 现在的7.3 不必配置了

## 安装 docker
```sh
sudo yum install docker
sudo systemctl enable docker.service
sudo systemctl start docker
```

## docker 代理
代理配置参考[这里](https://docs.docker.com/engine/admin/systemd/#/http-proxy)

```sh
sudo mkdir /etc/systemd/system/docker.service.d
```

新建一个 `/etc/systemd/system/docker.service.d/http-proxy.conf`
```
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
```

或者 在 /usr/lib/systemd/system/docker.service 添加
```
Environment=HTTP_PROXY=http://proxy.example.com:80
Environment=HTTPS_PROXY=http://proxy.example.com:80
```


## 修改为国内镜像 
/etc/docker/daemon.json
```js
{
  "registry-mirrors": ["https://www.daocloud.io"]
}
```

重启 docker 服务
```sh
sudo systemctl daemon-reload
sudo systemctl show --property=Environment docker
sudo systemctl restart docker
```

## 创建 docker group 并加入
```sh
sudo groupadd docker
sudo usermod -aG docker ${USER}
docker run --rm hello-world
```

> 重新登录tty之后， 用${USER}才开始生效，否则会有permission denied

## pull docker 镜像

```sh
docker pull nginx:1.10
```

## 查看镜像配置
```sh
docker inspect nginx:1.10
```

## 启动容器
```sh
docker run ubuntu /bin/echo 'Hello world'
docker run -t -i ubuntu /bin/bash
```

* `run` 运行一个容器
* `ubuntu` 选择一个镜像
* `-i` 以交互模式运行容器，通常与 -t 同时使用
* `-t` 为容器重新分配一个伪输入终端，通常与 -i 同时使用
* `/bin/bash` 在容器里启动一个Bash

```sh
docker run --privileged -it -P docker.io/nginx:1.10 /bin/bash
docker run --name ng -it -p 8888:80 -v /home/user/abc:/mnt docker.io/nginx:1.10 /bin/bash
docker run -d -P training/webapp python app.py
```

* `-d` 后台运行容器，并返回容器ID
* `-P` 为容器内的端口 映射到 主机的随机端口
* `-p 8888:80` 容器内的80端口 映射到 主机的 8888端口
* `--privileged` 容器内的root拥有真正的root权限
* `--name ng` 为容器取一个名字叫 ng
* `-v /home/user/abc:/mnt` 将host里的 /home/user/abc 挂载到容器里的 /mnt 目录下
* `python app.py` 在容器内运行这个命令


```sh
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
ef892632f55f        ubuntu              "/bin/sh -c 'while tr"   About a minute ago   Up About a minute                       loving_mahavira
```

```sh
docker ps -a
docker logs loving_mahavira
docker stop loving_mahavira
```

## 进去已经启动的docker容器
第一种：
```sh
docker exec -it centos:centos7.3.1611 /bin/bash
```

第二种：
```sh
docker attach CONTAINER_ID
```

导出 docker 镜像
```sh
docker save media/umd:centos7.4 > umd.tar
```

导入 docker 镜像
```sh
docker load -i umd.tar
```

上面的情况是没有压缩的tar包，可以做压缩处理
```sh
docker save ubuntu:v0.6 | gzip > ubuntu.tgz
zcat ubuntu.tgz | docker load
```

导出容器
```sh
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:14.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ docker export 7691a814370e > ubuntu.tar
```

导入容器
```sh
cat ubuntu.tar | docker import - test/ubuntu:v1.0
```

## 在container中执行命令的脚本
```sh
docker exec -i practical_stonebraker bash << EOF
ls
pwd
lsb_release -a
EOF
```

## 建立私有库
```sh
docker pull registry
docker run -d -p 5000:5000 --restart=always --privileged -v /home/media/registry:/tmp/registry docker.io/registry:latest
```

为了上传 取消安全监测
server: /etc/default/docker (Ubuntu16.04)
```
DOCKER_OPTS="--insecure-registry www.example.com:5000"
ADD_REGISTRY='--add-registry 10.67.116.92:5000'
```

client: /etc/sysconfig/docker (RHEL/CentOS)
```
INSECURE_REGISTRY='--insecure-registry www.example.com:5000'
```

或者 在 /etc/docker/daemon.json (Debian/Ubuntu)
```js
{
  "insecure-registries": ["www.example.com:5000"]
}
```

重启docker
```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

修改 tag
```sh
docker tag media/umd:centos7.4 www.example.com:5000/media/umd:centos7.4
```

上传到私有库
```sh
docker push www.example.com:5000/media/umd:centos7.4
```

pull image
```sh
bzhux@media-pxe3$[docker] >> curl www.example.com:5000/v2/_catalog
{"repositories":["media_driver_centos7.4"]}

bzhux@media-pxe3$[docker] >> curl www.example.com:5000/v2/media_driver_centos7.4/tags/list
{"name":"media_driver_centos7.4","tags":["v2.0"]}

bzhux@media-pxe3$[docker] >> docker pull www.example.com:5000/media_driver_centos7.4:v2.0
```

## Dockerfile
```
FROM centos:centos7.3.1611
MAINTAINER  zhubinqiang <zhubinqiang@gmail.com>

ENV TZ "Asia/Shanghai"
ENV TERM xterm

ADD CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
ADD epel.repo /etc/yum.repos.d/epel.repo

RUN yum install -y wget vim passwd yum-utils net-tools tmux git openssh-server
RUN yum groupinstall -y "Development Tools"

#WORKDIR /root/
#RUN ./run.sh

RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key -N '' && \
    ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N '' && \
    ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' && \
    ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N '' && \
    echo "123456" | passwd --stdin root && \
    echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config

EXPOSE 22
```

build Dockerfile
```sh
docker build -t my/centos:7.3 .
```

启动 container
```sh
docker run -it -P my/centos:7.3 /bin/bash
```

## docker 启动图像界面
```dockerfile
FROM centos:centos7.4.1708

RUN yum groupinstall -y "GNOME Desktop"
RUN yum groupinstall -y "Development Tools"
```

```sh
docker run -it -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix docker-gui firefox
```

