# 搭建docker、gitlab、sonar、jenkins、nginx、mysql、redis、rabbitmq过程

## centos 基本环境准备
> VMware® Workstation

> Centos 7

## docker 环境安装

### docker 安装
```code
https://docs.docker.com/engine/install/centos/
```
> Set up the repository
```code
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
> Install Docker Engine
```code
sudo yum install docker-ce docker-ce-cli containerd.io
```
> Start Docker
```code
sudo systemctl start docker
```
> Test
```code
sudo docker run hello-world
```

> docker 语法介绍
```code
https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
```

