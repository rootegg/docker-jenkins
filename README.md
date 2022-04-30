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

> 设置网络，最小化Centos7安装后一般没有开启网络
```code
1)查看网卡
[root@localhost ~]# ip a
2)修改网卡参数
ONBOOT=no改成yes
[root@localhost ~]# sed -i 's|ONBOOT=no|ONBOOT=yes|g' /etc/sysconfig/network-scripts/ifcfg-enp0s3
3)重启网卡服务
[root@localhost ~]# systemctl restart network

2丶开启自动获取动态IP地址
1)修改网卡参数
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static #改成静态模式
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=dcbf623d-ea0d-41e3-8062-f147336c0f04
DEVICE=enp0s3
ONBOOT=yes #开启网卡
IPADDR=192.168.3.8 #静态IP
GATEWAY=192.168.3.1 #网关IP
NETMASK=255.255.255.0 #子网掩码
DNS1=114.114.114.114 #首先DNS地址
```

> Centos7软件的镜像设置清华源
```code
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
```
最后，更新软件包缓存
```code
yum makecache
```

> 设置DNS
```code
DNS配置增加，要重启reboot
# vi /etc/resolv.conf
nameserver 114.114.114.114
nameserver 114.114.114.115
```
## 第二步：docker

### 一、docker 安装
```code
参考地址 https://docs.docker.com/engine/install/centos/#install-using-the-repository
```
> 1、Set up the repository
```code
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
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

```code
记一次执行yum命令报错：Could not retrieve mirrorlist http://mirrorlist.centos.org/

DNS配置增加，要重启reboot
# vi /etc/resolv.conf
nameserver 114.114.114.114
nameserver 114.114.114.115
```
> 2、Install Docker Engine
```code
yum install docker-ce docker-ce-cli containerd.io
```
> 遇到问题
```code
安装过程中，会询问是否继续安装某些工具，输入y回车就行
```
> 3、设置开机启动
```code
systemctl enable docker
```
> 4、Start Docker
```code
systemctl start docker
```
> 5、Test
```code
docker run hello-world
```
>6、harbor 不能用http解决，192.168.1.100:5000是Harbor服务器地址
```code
在客户机”/etc/docker/“目录下，创建”daemon.json“文件。在文件中写入：

{ "insecure-registries":["192.168.1.100:5000"] }
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
docker container update --restart=always [containerID] 更新容器增加restart
docker container exec -it [containerID] /bin/bash  进入容器且启动shell
docker exec -it --user root [containerID] bash  用root进入容器shell
docker container cp [containID]:[/path/to/file]  /path/to/file 从正在运行的 Docker 容器里面，将文件拷贝到本机，两路径可交换
docker run -d --restart always -p 10240:8080 -p 10241:50000 -v /var/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime --name myjenkins jenkins/jenkins 运行jenkins容器，如果没有回下载镜像再安装容器，-d 后台运行  -p 端口映射 -v 目录挂载 --name 设置容器名称
```


## 第三步：gitlab
```code
参考地址 https://docs.gitlab.com/ee/install/docker.html#install-gitlab-using-docker-engine
```
> 1、Install GitLab using Docker Engine 社区版
```code
sudo docker run --detach \
  --publish 44301:443 --publish 36001:80 --publish 2201:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
  ```
  
> 2、等等gitlab启动成功，访问地址 192.168.1.10:36001
```code
这个启动非常耗cpu，冲到90%，可能服务器卡死
docker ps可以看到STATUS是  heathing: starting，等待变成 heathy才行，
如果长时间没启动成功，或者报错，可能是这三个volume 目录映射没有权限，可去掉这三个volume ，学会查看日志  docker logs gitlab
cpu过高问题 https://www.cnblogs.com/51core/articles/15305816.html

gitlab非常耗内存，如果少于700M可用内存，看是否要清楚缓存内存占用
free -m 命令查询当前内存使用情况，单位M
echo 1 > /proc/sys/vm/drop_caches :表示清除pagecache。
echo 2 > /proc/sys/vm/drop_caches :表示清除回收slab分配器中的对象（包括目录项缓存和inode缓存）。slab分配器是内核中管理内存的一种机制，其中很多缓存数据实现都是用的pagecache。
echo 3 > /proc/sys/vm/drop_caches :表示清除pagecache和slab分配器中的缓存对象。
```

> 3、获取root初始密码，这里获取的密码如果不能登录gitlab，直接用第三步重置root密码

注意：The password file will be automatically deleted in the first reconfigure run after 24 hours.
```code
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

> web界面首次修改root密码
```code
右上角 -> Edit profile -> password
```

> 4、reset password
```code
sudo docker exec -it gitlab bash
sudo gitlab-rake "gitlab:password:reset"
```

> 5、修改gitllab显示的clone地址，不然是一串数字乱码
```code
docker exec -it -u root gitlab bash
vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
修改地址 host: 192.168.1.10
gitlab-ctl restart
```

## 第四步：nginx
```code
参考地址 https://www.runoob.com/docker/docker-install-nginx.html
```
> 安装
```code
sudo mkdir -p /home/nginx/www
sudo docker run --restart always --name nginx -p 80:80 -v /home/nginx/www:/usr/share/nginx/html -d nginx
```
挂载到/home/nginx/www目录的目的是，后面jenkins发布dist文件用ssh直接推到这个目录就行了
访问 http://192.168.162.129/ 可以看到 Welcome to nginx!

## 可选：安装ngrok做内网穿透

> 拉取 wernight/ngrok
```code
docker pull wernight/ngrok
```

> 后台运行ngrok指向ngxin 80端口，authtoken 要先去 https://dashboard.ngrok.com/get-started/setup 官网注册获取
```code
docker run -d -p 4040 --name www_ngrok --link nginx wernight/ngrok ngrok http nginx:80 --authtoken 24GP7iKlsqGYDwh0QjjqcoviMws_6SQujd8xWkhB2oSVQd2Yk
```

> 显示穿透域名，外网便可直接访问
```code
curl $(docker port www_ngrok 4040)/api/tunnels
```
![image](https://user-images.githubusercontent.com/82021554/151295865-91447928-fb5f-4746-874d-7aee8f7944c6.png)

> 第三步报错  Error: No public port '4040/tcp' published for www_ngrok，则换可用authtoken
```code
24GP7iKlsqGYDwh0QjjqcoviMws_6SQujd8xWkhB2oSVQd2Yk
24HJkjwss1uvgmvSvXRMSFwsofF_55AaiQuiYWdeTnCFgzYji
```

> 第三步报错 curl: (3) Bad URL, colon is first character
```code
docker port www_ngrok 4040       
// 用上句显示的端口49167
curl http://127.0.0.1:49167/api/tunnels
```
![image](https://user-images.githubusercontent.com/82021554/151350801-a0a9d55b-130f-485a-b420-6297d7b5792a.png)


## 第五步：sonar
```code
参考地址 https://www.jianshu.com/p/e0883f347901
```
> 1、安装
```code
sudo docker run -d --restart always --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:8.9.2-community
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
sudo docker run -d --restart always -p 10240:8080 -p 10241:50000 -v /var/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins jenkins/jenkins
```
> 2、安装镜像加速
```code
修改文件 vi /var/jenkins_mount/hudson.model.UpdateCenter.xml
将url改为 https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```
> 3、获取默认admin初始密码
访问网站 http://192.168.162.129:10240/，云平台注意打开防火墙端口10240
```code
cat /var/jenkins_mount/secrets/initialAdminPassword
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

3、虚拟机重启后 Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?服务没开启
su root # 先切换到root用户, 再执行以下命令
systemctl enable docker # 开机自动启动docker
systemctl start docker # 启动docker

4、虚拟机重启后 没有ip了
执行 ifconfig 发现多了docker0，虚拟机保持NAT链接
执行
ifconfig docker0 down
service network restart
关闭虚拟机，重新启动，或许就好了


```
# 追加内容
## 第八步：搭建Harbor

### 安装docker-compose
下载docker-compose
curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

查看下载好的包
ls /usr/local/bin/

修改执行权限
chmod +x /usr/local/bin/docker-compose

软连接映射到/usr/bin/
ln -sf  /usr/local/bin/docker-compose /usr/bin/docker-compose

验证
which docker-compose

### 安装harbor
yum install -y wget
wget https://github.com/goharbor/harbor/releases/download/v2.5.0/harbor-online-installer-v2.5.0.tgz

解压harbor安装包
tar xf harbor-online-installer-v2.5.0.tgz -C /data/app/
编辑harbor.yml文件
cd /data/app/harbor
cp harbor.yml.tmpl harbor.yml

vim harbor.yml
hostname:   harbor01.k8s.com   #主机IP/或者域名
harbor_admin_password: harbor123456   #harbor UI界面登陆密码
data_volume: /data/app/harbor-data  #harbor 持久化数据

#关闭https（把以下的行都注释掉12-18行）
# https related config
#https:
# # https port for harbor, default is 443
# port: 443
# # The path of cert and key files for nginx
# certificate: /your/certificate/path
# private_key: /your/private/key/p

###　配置harbor开机自启动
1)编写启动脚本
vim /data/app/harbor/startall.sh
#!/bin/bash

cd /data/app/harbor
docker-compose stop && docker-compose start

2)赋予执行权限
chmod +x  /data/app/harbor/startall.sh
3)把启动脚本加到系统启动之后最后一个执行的文件
echo "/bin/bash /data/app/harbor/startall.sh" >>/etc/rc?
