搭建docker、gitlab、sonar、jenkins、nginx、mysql、redis、rabbitmq过程

## 1、centos
> VMware® Workstation

> Centos 7

## 2、docker

### 一、docker 安装
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

### 二、docker 语法介绍
```code
https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
```
> 常用命令
```code
docker pull [image] 拉取镜像
docker images 查看所有镜像
docker ps 查看正在运行的容器
docker ps -a 查看所有容器，包括停止的容器
docker containre rm  [containerID] 移除容器
docker container start [containerID] 启动容器
docker container stop [containerID] 停止容器
docker container restart [containerID] 重启容器
docker container exec -it [containerID] /bin/bash  进入容器且启动shell
docker container cp [containID]:[/path/to/file]  /path/to/file 从正在运行的 Docker 容器里面，将文件拷贝到本机，两路径可交换
docker run -d -p 10240:8080 -p 10241:50000 -v /var/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name myjenkins jenkins/jenkins 运行jenkins容器，如果没有回下载镜像再安装容器，-d 后台运行  -p 端口映射 -v 目录挂载 --name 设置容器名称
```

