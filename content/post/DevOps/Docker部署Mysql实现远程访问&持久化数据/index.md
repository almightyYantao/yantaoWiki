---
title: Docker部署Mysql实现远程访问&持久化数据
date: 2024-03-26
tags: ['DevOps','Mysql'] 
---

## 部署方式

### Docker安装

```shell
# 安装必要的软件包，允许 yum 使用 HTTPS 仓库
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 设置 Docker CE 仓库
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker CE
sudo yum install -y docker-ce

# 启动 Docker 服务
sudo systemctl start docker

# 设置 Docker 服务开机自启动
sudo systemctl enable docker
```

### 拉取镜像

```shell
docker pull mysql:5.7.38
```

### 启动容器

```shell
docker run -d --name mysql_server -e MYSQL_ROOT_HOST='%' -e MYSQL_ROOT_PASSWORD=xxxx -e MYSQL_USER=xxx -e MYSQL_PASSWORD=xxx -e MYSQL_DATABASE=xxx -p 3306:3306 -v /home/mysql/db:/var/lib/mysql -v /home/mysql/conf:/etc/mysql/my.cnf mysql:5.7.38
```

参数说明：

- --name：容器名称
- -e MYSQL_ROOT_HOST='%'：允许root账户任何IP连接
- -e MYSQL_ROOT_PASSWORD=xxxx：root账户密码
- -e MYSQL_USER=xxx：新增用户名称
- -e MYSQL_PASSWORD=xxx：新增用户密码
- -e MYSQL_DATABSE=xxx：新增数据库名
- -p 3306:3306：端口映射
- -v /home/mysql/db:/var/lib/mysql：数据库持久化映射
- -v /home/mysql/conf:/etc/mysql/my.cnf：数据库配置文件映射
- mysql:5.7.38：版本

## 问题

如果出现无法连接的情况，大概率是tcp转发没有开启

### 启用 IP 转发

编辑 /etc/sysctl.conf 文件，新增一下内容：

```shell
net.ipv4.ip_forward=1
```

然后保存并退出文件，执行以下命令使修改生效：

```shell
sudo sysctl -p
```
