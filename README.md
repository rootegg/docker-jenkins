搭建docker、gitlab、sonar、jenkins、nginx、mysql、redis、rabbitmq过程
![image](https://user-images.githubusercontent.com/82021554/134346250-1c20b8ed-7e56-4756-b473-8d66f0b1fdc2.png)
![image](https://user-images.githubusercontent.com/82021554/134346362-152818fd-97bb-440d-9a73-946897cc6521.png)
![image](https://user-images.githubusercontent.com/82021554/134346445-a232019b-34b2-4a19-8fab-5b98812e3294.png)
![image](https://user-images.githubusercontent.com/82021554/134346512-02258f37-42cd-46cc-be0f-7212e48bdd87.png)


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
docker run -d -p 10240:8080 -p 10241:50000 -v /var/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime --name myjenkins jenkins/jenkins 运行jenkins容器，如果没有回下载镜像再安装容器，-d 后台运行  -p 端口映射 -v 目录挂载 --name 设置容器名称
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
> 2、获取root初始密码

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
sudo mkdir -p /home/nginx/www
sudo docker run --name nginx -p 80:80 -v /home/nginx/www:/usr/share/nginx/html -d nginx
```
挂载到/home/nginx/www目录的目的是，后面jenkins发布dist文件用ssh直接推到这个目录就行了
访问 http://192.168.162.129/ 可以看到 Welcome to nginx!

## 第五步：sonar
```code
参考地址 https://www.jianshu.com/p/e0883f347901
```
> 1、安装
```code
sudo docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:8.9.2-community
```
注意 初始账号admin密码admin  访问网址如192.168.162.129:9000 需要等待一段时间服务启动

2、建一个项目，后面jenkins用
```code
#sonarqube服务器地址
sonar.host.url=http://192.168.162.129:9000/
#sonarqube token
sonar.login=fc3486ae08da99f0735bfb241b888f68d736d55e
#项目唯一标识
sonar.projectKey=myvue1
#项目名称
sonar.projectName=myvue1
#源代码目录
sonar.sources=src
#语言
sonar.language=js
#源代码文件编码
sonar.sourceEncoding=UTF-8
#忽略目录（非自身项目代码）
# sonar.exclusions=src/components/**
```

## 第六步：jenkins
> 1、安装
```code
参考地址 https://www.cnblogs.com/fuzongle/p/12834080.html
```
```code
sudo mkdir -p /var/jenkins_mount
sudo chmod 777 /var/jenkins_mount
sudo docker run -d -p 10240:8080 -p 10241:50000 -v /var/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins jenkins/jenkins
```
> 2、安装镜像加速
```code
修改文件 vi /var/jenkins_mount/hudson.model.UpdateCenter.xml
将url改为 https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```
> 3、获取初始密码
访问网站 http://192.168.162.129:10240/
```code
cd /var/jenkins_mount/secrets/
cat initialAdminPassword
```
> 4、点击安装推荐的插件按钮即可

## 第七步：搭建自动化构建
> 1、安装插件（nodejs、sonarqube scanner、Publish Over SSH） [图片](https://user-images.githubusercontent.com/82021554/134290759-5efed515-6e68-4d39-973a-f0763be56a88.png)
```code
地址 http://192.168.162.129:10240/pluginManager/available
Manage Jenkins(管理jenkins) -> 插件管理 -> 可用插件 -> nodejs
```

> 2、全局配置node版本 [图片](https://user-images.githubusercontent.com/82021554/134291123-bb9da92a-2694-4fbe-bf90-4c243de342d0.png)
```code
地址 http://192.168.162.129:10240/configureTools/
Manage Jenkins(管理jenkins) -> （global tool confirguation）全局工具配置 -> nodejs
```


> 3、配置sonar服务器 [图片](https://user-images.githubusercontent.com/82021554/134292152-a8727dff-8696-407f-b427-2cb415f41119.png)
```code
地址 http://192.168.162.129:10240/configure
Manage Jenkins(管理jenkins) -> （confirguation system）配置系统 -> SonarQube servers
```


> 4、配置jenkins关联sonarqube scanner [图片](https://user-images.githubusercontent.com/82021554/134292300-7c301564-b594-4831-9e48-a2c854d9deba.png)
```code
地址 http://192.168.162.129:10240/configureTools/
Manage Jenkins(管理jenkins) -> （global tool confirguation）全局工具配置 -> SonarQube Scanner
```


> 5、配置SSH服务 [图片](https://user-images.githubusercontent.com/82021554/134295272-b5712e84-ed13-400e-a0ba-e1a7ef744f73.png)
```code
地址 http://192.168.162.129:10240/configure 
Manage Jenkins(管理jenkins) -> （confirguation system）配置系统 -> Publish over SSH -> SSH Servers
配置完点一下Test Configuration看通不通
```


> 6、发布项目
> 
 [图片](https://user-images.githubusercontent.com/82021554/134345116-393ce616-3858-451b-9e59-6a2b6bb2ef65.png)
 [图片](https://user-images.githubusercontent.com/82021554/134295775-ebe28393-8d23-4c6e-b939-a4757a37ecc5.png)
 [图片](https://user-images.githubusercontent.com/82021554/134303912-79d66c8a-bbc6-4168-ad25-9b4442fcbe94.png)
```code
1、新自由风格空项目
2、git源码配置
3、勾选构建环境 -> Add timestamps to the Console Output
4、勾选构建环境 -> Provide Node & npm bin/ folder to PATH ，勾选就好了，其他不用动
```


4、sonar信息配置 [图片](https://user-images.githubusercontent.com/82021554/134303569-7c1e0b9b-8228-498d-a5e4-1c6ec0ad8377.png)
构建 -> 新增Execute SonarQube Scanner -> Analysis properties 其他不用填
```code
#sonarqube服务器地址
sonar.host.url=http://192.168.162.129:9000/
#sonarqube token
sonar.login=fc3486ae08da99f0735bfb241b888f68d736d55e
#项目唯一标识
sonar.projectKey=myvue1
#项目名称
sonar.projectName=myvue1
#源代码目录
sonar.sources=src
#语言
sonar.language=js
#源代码文件编码
sonar.sourceEncoding=UTF-8
#忽略目录（非自身项目代码）
# sonar.exclusions=src/components/**
```


5、node信息配置 [图片](https://user-images.githubusercontent.com/82021554/134305192-a512ea0b-798c-403b-a90b-cd78bc1c8e00.png)
构建 -> 新增执行shell
```code
npm install --registry https://registry.npm.taobao.org/
npm run build
cd dist
tar -cvf dist.tar ./
```


6、SSH发送到nginx 

[图片1](https://user-images.githubusercontent.com/82021554/134343453-4346ab4f-c28f-4129-b475-796eb6f64417.png) 
[图片2](https://user-images.githubusercontent.com/82021554/134343538-975a6f22-7820-41ce-88ea-dd5f79acd996.png)
[图片3](https://user-images.githubusercontent.com/82021554/134343633-86215ac2-a814-42d1-b2f0-e3e88ece08ae.png)
[图片4](https://user-images.githubusercontent.com/82021554/134343712-05291995-0e8b-43b3-b54b-788b6ec366ae.png)
[图片5](https://user-images.githubusercontent.com/82021554/134343760-84b4ea81-8b4c-4a0a-865b-a0be4a9bd819.png)

```code
构建后操作 -> 新增Send build artifacts over SSH
Source files填 dist/dist.tar
Remove prefix填 dist/
Remote directory填 ./
Exec command填 
cd /home/nginx/www
tar -xvf dist.tar
rm -rf dist.tar
```

7、到项目myvue1点击立即构建
遇到问题

```code
1、解决Unpacking https://nodejs.org/dist/v16.9.1/node-v16.9.1-linux-x64.tar.gz to /var/jenkins_home/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/nodejs16.9.1 on Jenkins
[图片](https://user-images.githubusercontent.com/82021554/134309505-354257b1-cb6f-4a19-ad0e-81d1ae23d2c0.png)
参考地址 https://blog.csdn.net/u012075238/article/details/103052201
手动下载node-v16.9.1-linux-x64.tar.gz，上传/var/jenkins_home目录，执行 tar -xzvf node-v16.9.1-linux-x64.tar.gz，会生成/var/jenkins_home/node-v16.9.1-linux-x64目录


2、解决SSH: Transferred 0 file(s)
参考地址 https://www.jianshu.com/p/ef6a4022b7b5
Source files **/* 表示sskzmz这个job的工作目录下所有的文件和目录。
Remove prefix 该操作是针对上面的source files目录，会移除匹配的目录。通常留空。
Remote directory 该操作是基于设定的服务器目录进行。这里我的服务器配置是的/www. 因此这里应该写sites/sskzmz即可。
Exec command 远程服务器执行的命令。例如可以输出 service nginx restart 或者/www/xx. sh 均可。

重点一: source files 要基于任务的目录进行。不支持绝对路径。如果配置不对，则找不到文件。上例中/var/jenkins_home/workspace/sskzmz 是任务目录。最终jenkins会选择 /var/jenkins_home/workspace/sskzmz/**/* 查询所要传送的文件。

重点二: Remote directory 要基于你远程服务器的目录配置。你远程服务器配置的基准是/www 。则最终的文件目录是 /www+ Remote directory的配置参数。不支持绝对路径。
```

3、虚拟机重启后 Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?服务没开启
su root # 先切换到root用户, 再执行以下命令
systemctl enable docker # 开机自动启动docker
systemctl start docker # 启动docker


