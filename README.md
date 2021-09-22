搭建docker、gitlab、sonar、jenkins、nginx、mysql、redis、rabbitmq过程

## 第一步：centos7
> VMware® Workstation

> Centos 7
```code
https://mirrors.tuna.tsinghua.edu.cn/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso
```

## 第二步：docker

### 一、docker 安装
```code
参考地址 https://docs.docker.com/engine/install/centos/#install-using-the-repository
```
> 1、Set up the repository
```code
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
> 遇到问题
```code
执行yum安装命令式报如下错误，解决办法通过强制关掉yum进程。

Loaded plugins: fastestmirror, refresh-packagekit, security
Existing lock /var/run/yum.pid: another copy is running as pid 2922.
Another app is currently holding the yum lock; waiting for it to exit...
  The other application is: PackageKit
    Memory :  52 M RSS (908 MB VSZ)
    Started: Fri Sep 14 01:41:58 2018 - 01:58 ago
    State  : Sleeping, pid: 2922

实现方式如下，然后重新使用yum安装：

#rm -f /var/run/yum.pid
```
> 2、Install Docker Engine
```code
sudo yum install docker-ce docker-ce-cli containerd.io
```
> 遇到问题
```code
安装过程中，会询问是否继续安装某些工具，输入y回车就行
```
> 3、Start Docker
```code
sudo systemctl start docker
```
> 4、Test
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


## 第三步：gitlab
```code
参考地址 https://docs.gitlab.com/ee/install/docker.html#install-gitlab-using-docker-engine
```
> 1、Install GitLab using Docker Engine 社区版
```code
sudo docker run --detach \
  --publish 44301:443 --publish 8001:80 --publish 2201:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
  ```
> 2、Visit the GitLab URL, and log in with username root and the password from the following command:

注意：The password file will be automatically deleted in the first reconfigure run after 24 hours.
```code
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

> 3、reset password
```code
sudo gitlab-rake "gitlab:password:reset"
```

## 第四步：nginx
```code
参考地址 https://www.runoob.com/docker/docker-install-nginx.html
```
> 安装
```code
subo docker run --name nginx -p 80:80 -d nginx
```

## 第五步：sonar
```code
参考地址 https://www.jianshu.com/p/e0883f347901
```
> 安装
```code
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:8.9.2-community
```
注意 访问网址如192.168.162.129:9000 需要等待一段时间服务启动
