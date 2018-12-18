#                           docker-quick-start

# 1.docker简单介绍
##  1.1 docker是什么:
   Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。与java的一次编译多出运行类似，docker的出现实现了应用从搬家到直接搬整栋房子的过程，镜像文件中包含了应用运行所需要的完整的配置，环境等，因而可以实现一处打包，多处部署。用户可已将自己的镜像放到仓库，其他用户通过拉取镜像，运行镜像实例，进而运行应用，极大简化了部署维护的工作量。**容器是完全使用沙箱机制，相互之间不会有任何接口。** 关于docker容器之间的交互见后文。
##  1.2 docker的部件
### 1.2.1. Docker 镜像：
Docker 镜像是 Docker 容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成，一层一层向上堆叠，最后暴露给我们的就是我们需要的，以tomcat镜像为例从下到上kernel,centos/ubuntu,jdk,tomcat，最后暴露给我们的是Tomcat应用。这种分层的文件系统有利于资源的共享。Docker 使用 UnionFS 来将这些层联合到单独的镜像中。UnionFS允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。正因为有了这些层的存在，Docker 是如此的轻量。当你改变了一个 Docker 镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新 的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。
### 1.2.2 Docker 仓库：
Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。公有的 Docker 仓库名字是 Docker Hub。Docker Hub 提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker 仓库是 Docker 的分发部分。
### 1.2.3 Docker 容器：
Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个 Docker 容器都是从 Docker 镜像创建的。Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是 Docker 的运行部分，每一个doker容器都相当于一个精简的linux系统，一个docker镜像可以运行多个实例。
## 1.3 docker 与虚拟机的区别:
docker的优势在于其相对于虚拟机的快速和轻量级，虽然二者都是虚拟技术，但是虚拟的层次不同。传统的虚拟机是在宿主机的体系之上虚拟出自己一个完整的体系包括内核，软硬件，环境等，而docker则是寄生或者说利用了宿主机的内核，自己则最精简最小化（不包含硬件的虚拟）。一个docker就是一个最精简的linux操作系统，这就很好解释，通过虚拟机安装的centos有4个多G，而通过docker容器构建的centos只有不到200MB。虚拟机重启的时候，需要重新启动操作系统的内核，然后加载文件系统，而docker依赖于宿主机的内核，重启时候只加载文件系统即可，这样就大大节省了启动的时间。




# 2.Docker基本操作指令(万能的docker --help)
## 2.1 安装docker

### 2.1.1 centos7 安装docker

    sudo yum remove docker \
                      docker-client \
                      docker-client-latest \
                      docker-common \
                      docker-latest \
                      docker-latest-logrotate \
                      docker-logrotate \
                      docker-selinux \
                      docker-engine-selinux \
                      docker-engine         //清除之前的安装记录
     sudo yum install -y yum-utils device-mapper-persistent-data lvm2  //安装必要的工具
     sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo    //添加阿里云的docker镜像库到yum源中
     sudo yum makecache fast   //更新yum缓存
     sudo yum -y install docker-ce   //安装docker
     sudo systemctl start docker //启动docker

更多详细的知识转到[帮助文档地址](http://www.runoob.com/docker/centos-docker-install.html) 介绍了通过源码和yum的方式安装以及测试是否安装成功。

### 2.1.2 Ubuntu 16

目前只有在centos7上安装过docker,ubuntu的[帮助文档](http://www.runoob.com/docker/ubuntu-docker-install.html)中 介绍了不同的版本的Ubuntu的docker安装的详细步骤。

## 2.2 运行docker（以centos7为例）

    启动docker：sudo systemctl start docker        //需要管理员权限

## 2.3 基本指令

### 2.3.1.docker build：
docker build 命令用于使用 Dockerfile 创建镜像，语法为 docker build [OPTIONS] PATH | URL |  常用的OPTIONS为-t 表示指定镜像的名字以及标签（版本相关的标记）PATH为本地的dockerfile路径，URL为远程的。实际在安装某个image之前需要去docker hub这个中心仓库查询相关镜像。指令为 docker search 镜像名，如docker search mysql(可能需要ROOT权限) ,会得到所有的mysql相关的镜像文件列表。例如通过dockerfile来构建mysql的镜像（[完整版本](https://github.com/docker-library/mysql/blob/master/5.7/Dockerfile)）

    FROM centos:7
    MAINTAINER chenyufeng "yufengcode@gmail.com"  
    # 使用yum的方式安装mysql；
    RUN yum install -y mysql-server mysql  
    # 安装完成以后，执行以下命令。配置用户名密码相关信息；
    RUN /etc/init.d/mysqld start &&\  
        mysql -e "grant all privileges on *.* to 'root'@'%' identified by 'test' WITH GRANT OPTION ;"&&\  
        mysql -e "grant all privileges on *.* to 'root'@'localhost' identified by 'test' WITH GRANT OPTION ;"&&\ 
        mysql -u root -p123456 -e "show databases;"  
    # 镜像暴露3306端口；
    EXPOSE 3306
    # 容器启动后执行以下命令，启动mysql；
    CMD ["/usr/bin/mysqld_safe"]
 以上是mysql5.7的dockerfile的最简化可以应用版本。完整的版本见上方连接。然后dockerfile创建完成以后，就可以通过docker build 指令来创建本地mysql镜像,指令如下：
​    
     
    docker  build  -f   dockerfile    -t   test/mysql:5.7 //创建镜像

    docker run -name mysql  -p 3306:3306  -e MYSQL_ROOT_PASSWORD=123456 -d test/mysql   //开启一个mysql的容器

然后进入容器内部就可以查看数据库进行相关操作


### 2.3.2 docker image ls:
查看本地的所有镜像。完整的语法为docker image ls  [OPTIONS]  常用的参数 –q 只显示镜像id，-f 筛选镜像显示。

### 2.3.3 docker rmi :
删除本地一个或多少镜像,完整语法docker rmi [OPTIONS] IMAGE [IMAGE...]    OPTION常用 –f 强力删除，可接受多个image一次删除。

### 2.3.4 docker  run:
创建一个新的容器并运行一个命令,完整语法docker run [OPTIONS] IMAGE COMMAND   常用的OPTION   –itd  -d: 后台运行容器(以守护进程的方式运行)，并返回容器ID；-i: 以交互模式运行容器，通常与 -t 同时使用；-t: 为容器重新分配一个伪输入终端.不同的应用在运行的时候需要指定不同的参数，具体情况需要具体去查询。（例如利用docker启动kafka）

    docker run -d --name kafka --publish 9082:9092 \ -d指定运行方式为后台运行 –publish是指定端口映射--     --link zookeeper:zookeeper \  --link 是指定将kafka链接到zookeeper容器

    　　--env KAFKA_BROKER_ID=100 \ --env=-e 指明运行环境

    　　--env HOST_IP=192.168.1.55 \

    　　--env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \

    　　--env KAFKA_ADVERTISED_HOST_NAME=192.168.1.55 \

    　　--env KAFKA_ADVERTISED_PORT=9082 \

    　　--restart=always \

    　　--volume /etc/localtime:/etc/localtime \

    　　wurstmeister/kafka

### 2.3.5 容器关闭，重启，启动指令:
docker start :启动一个或多个已经被停止的容器

docker stop :停止一个运行中的容器 

docker restart :重启容器 

三者语法结构类似docker start /stop/restart   [OPTIONS] CONTAINER [CONTAINER...]

### 2.3.6 docker image *

docker image pull [OPTIONS ] NAME[:TAG]：从仓库拉取镜像文件

docker image save[OPTION ] IMAGE [image]:将镜像保存，默认是到标准输出设备，-o 写入文件中去

docker image push  [OPTION]  IMAGE[:TAGE]:将一个镜像推送到仓库

docker image tag SOURCE_IMAGE[:TAGE]  TARGET_IMAGE[:TAG]:创建一个TARGET_IMAGE指向SOURCE_IMAGE

docker image ls[OPTION]:列出所有本地镜像

### 2.3.7进入容器内部：
docker attach  CONTAINER ：进入一个被退出但是未关闭的容器

docker exec  [OPTION]   CONTAINER COMMAND [ARGS] ,常用的参数 -i,-t(含义同上),进入容器并且生成一个线程，作用同上，但是更强，可以在不进入容器的环境（bash）的情况下，就将指令传入容器内部并且运行。如生成一个kafka的bash指令为

     docker exec -it kafka  /bin/bash       //进入一个容器的bash环境，可以进行查看和配置修改
### 2.3.8 退出容器
Ctrl+Q+P:退出容器而且不关闭，在进入docker attach container即可。

exit:退出且关闭容器

## 2.4 dockerfile

### 2.4.1 dockerfile介绍
dockerfile也是docker镜像创建的一种方式，类似于linux的shell脚本，能够识别linux指令。通过dockerfile文件用户可以自己创建镜像，并且推送到远程仓库，其功能远比docker pull强大，给用户自己添加或者更改镜像，或者自定义镜像提供了方便。
### 2.4.2 dockerfile的基本指令

#### 2.4.1  \# 用于注释
在dockerfile中#开头的行将不会被执行，通常作为补充说明信息

#### 2.4.2 escape 定义转义字符，\\用于连接
为了使得dockerfile的文件具有较好的可读性，当语句过长的时候，通常会分行写，当同一条命令要分行的时候可以用“\”进行连接，路径较长的时候使用escape定义转义字符，后者用法如下：

- \# escape=\ (backslash)  （默认的）

#### 2.4.3 FROM 声明继承的基础镜像
类似于java的继承，声明当前dockerfile生成的镜像的父镜像。表示开始构建，指定后续操作的基础镜像，dockerfile必须以FROM 开头（在FROM之前只可以有一个ARG指令，[二者的交互](https://docs.docker.com/engine/reference/builder/#from)），支持多个FROM指令，每次FROM都会清空之前的指令状态用法如下：
1. FROM <image>[AS name]
2. FROM <image>[:TAG] 默认latest

#### 2.4.4 LABLE 指定镜像的元数据信息
LABLE指令支持对父镜像的继承和重写，可以通过docker inspect container查看容器的信息，采用键值对的形式，支持多个标签，用法如下
1. LABEL <`key`>=<`value`> <`key`>=<`value`> <`key`>=<`value`> ...

#### 2.4.5 MAINTAINER 指定作者
格式为：
- MAINTAINER  USER <`email`>

慢慢被格式更加灵活的LABLE代替，格式如下
- LABEL "maintainer"="email"

#### 2.4.6 ENV 指定环境变量
同shell中的设置环境变量供后面使用（用法相同$KEY来获取值），常用的格式如下:
1. ENV <`key`> <`value`>
2. ENV "key"="value"

#### 2.4.7 COPY 复制文件

#### 2.4.8 EXPOSE “暴露端口”
告知容器监听某个端口，并未真的暴露，需要在run指令中通过-p或者-P来打开，支持制定协议类型，用法如下
- EXPOSE port/tcp(udp)

#### 2.4.9 VOLUME 挂载目录
用于在容器内部设置挂载点，用于资源的持久化和数据共享，用法如下：
- VOLUME ["containerdest1",]

支持同时制定多个挂载目录，默认会在主机的特定目录下随机生成对应的目录和文件，用于主机和容器之间进行数据共享。
#### 2.4.10 WORKDIR 指定后续指令的工作存储目录
如果指定的目录不存在，会自动创建，用法如下：
- WORKDIR  dest

#### 2.4.11 ADD "复制文件+"
#### 2.4.12 CMD 运行指令
#### 2.4.13 ENTRYPOINT 运行指令
#### 2.4.14 ONBUILD 钩子指令
#### 2.4.15 ARG 定制传参数
#### 2.4.16 SHELL 指定shell环境（多shell环境使用）




## 2.5 docker数据持久化以及文件共享（非dockfile方式）
1. 通过文件复制的方式：

    docker cp [OPTION] CONTAINER_SRC:DEST_PATH
    
    docker cp [OPTION] DEST_PATH:CONTAINER_SRC

这种复制需要频繁操作，比较麻烦。

2. 通过运行的时候指定：
    docker run -it -v <宿主目录>:<容器目录>   -P container 

默认的是可读可写，如需要限定需要在上面容器目录后添加相应的字符，对挂载点的读写权限可以通过doker inspect container 查看


　　　　 
