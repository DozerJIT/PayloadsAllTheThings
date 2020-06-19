# 使用docker搭建环境

## 摘要

* [install docker](#install-docker)
* [start docker](#start-docker)
* [install docker-compose](#install-docker-compose)
* [部署upload-labs/sqli-labs](#部署upload-labssqli-labs)
* [部署dvwa](#部署dvwa)
* [部署vulhub](#部署vulhub)

## install docker

#### 安装docker依赖包:ubuntu(apt-get)，centos(yum)

```
yum install -y yum-utils device-mapper-persistent-data lvm2 
```

#### 设置yum源

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

#### 安装docker

```
yum install docker-ce
```

#### 若报错，更换yum源

```
Error downloading packages:
  containerd.io-1.2.13-3.1.el7.x86_64: [Errno 256] No more mirrors to try.
  1:docker-ce-cli-19.03.7-3.el7.x86_64: [Errno 256] No more mirrors to try.
  3:docker-ce-19.03.7-3.el7.x86_64: [Errno 256] No more mirrors to try.
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## start docker

#### 启动并加入开机启动

```
systemctl start docker
systemctl enable docker
```

#### 存在client和server表示docker安装启动成功

```
docker version
```

#### 修改镜像源

```
vi /etc/docker/daemon.json
{

"registry-mirrors": ["http://hub-mirror.c.163.com"]

}
systemctl restart docker.service
```

#### 基本命令

```
下载镜像：                         docker pull 镜像名
查看镜像：                         docker images
查看正在运行镜像：                  docker ps
删除已下载镜像：                    docker rm 镜像名
停止/删除运行的容器：               docker stop/rmi 容器id
后台运行/映射端口：                 docker run -d -p 8080:80 镜像名
进入容器：                         docker exec -it 容器名 bash
保存/加载(类似vm虚拟机的vmdk文件）： docker save XX > 1.tar / docker load XX < 1.tar
```

## install docker-compose

Docker Compose 是一个用来定义和运行复杂应用的 Docker 工具，使用 Docker Compose 不再需要使用 shell 脚本来启动容器(通过 docker-compose.yml 配置)

### 1.通过pip安装

#### 首先安装python-pip并升级
```
yum install gcc libffi-devel python-devel openssl-devel -y
yum -y install epel-release
yum -y install python-pip
pip install --upgrade pip
```

#### 安装docker-compose
```
pip install docker-compose
```

#### 安装若报错
```
ERROR: Could not find a version that satisfies the requirement requests (from versions: none)
ERROR: No matching distribution found for requests

执行：pip install requests --ignore-installed chardet
```

#### 之后若报错，则更新python版本
```
ERROR: Exception:
Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/pip/_internal/cli/base_command.py", line 186, in _main
    status = self.run(options, args)
  File "/usr/lib/python2.7/site-packages/pip/_internal/commands/install.py", line 331, in run
```

### 2.通过源码安装

```
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

```

## 部署upload-labs/sqli-labs

#### 以upload-labs为例

### 1、先克隆项目到本地/opt并解压

```
cd ~/opt
wget https://github.com/c0ny1/upload-labs/archive/0.1.tar.gz
tar zxvf 0.1.tar.gz
```

#### 解压完之后会有一个upload-labs-0.1文件夹

### 2.建立docker image

```
[root@localhost opt]# cd upload-labs-0.1/
[root@localhost upload-labs-0.1]# cd docker/
[root@localhost docker]# ls
Dockerfile  docker-php.conf  php.ini
```

### 3.创建镜像

```
[root@localhost docker]# pwd
/opt/upload-labs-0.1/docker
[root@localhost docker]# docker build -t upload-labs .
```

### 4.运行镜像

```
docker run -d -p 8080:80 upload-labs
浏览器访问http://本机ip:8080
```

#### -d表示后台运行，-p表示端口映射，docker虚拟端口80映射本机8080端口

## 部署DVWA
### 1.搭建lamp容器

```
docker pull vuldocker/lamp
docker run -it -d --name dvwa -p 8008:80 vuldocker/lamp
docker ps
docker exec -it 容器id /bin/bash
```

### 2.安装dvwa

#### 安装git

```
yum install git
```

#### 下载、安装dvwa

```
git clone https://github.com/ethicalhack3r/DVWA.git
cd /var/www/html
mkdir dvwa
cd ../../../../
将下载的DVWA移动到/var/www/html/dvwa：mv /var/www/html/dvwa
cd /DVWA/config
mv config.inc.php.dist config.inc.php
浏览器访问http://本机ip:8008/dvwa/DVWA/setup.php
```

## 部署vulhub

```
git clone https://github.com/vulhub/vulhub.git
```

### vulhub集成了许多CVE漏洞，以安装CVE-2015-5254为例

```
cd vulhub/activemq/CVE-2015-5254
docker-compose build
docker-compose up -d
查看README.md
访问http://本机ip:8161（端口可在docker-compose.yml中修改)
```

## References

* [Get Docker Engine - Community for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
* [Centos7上安装docker](https://blog.csdn.net/hylaking/article/details/87978819)
* [记一次使用Docker部署漏洞靶场环境(以upload-labs为例)到ECS的经历](https://blog.csdn.net/cynthrial/article/details/88371274)
* [Docker镜像源修改](https://blog.csdn.net/jixuju/article/details/80158493)
* [CentOS7下安装Docker-Compose](https://www.cnblogs.com/YatHo/p/7815400.html)
* [更新python](https://www.cnblogs.com/fjping0606/p/9156344.html)
* [ubuntu中运用docker搭建dvwa漏洞靶场环境](https://blog.csdn.net/weixin_45426801/article/details/103275495)
* [docker下vulhub漏洞环境安装](https://blog.csdn.net/qq_36973952/article/details/80661023)




