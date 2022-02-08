### 比较虚拟机技术和容器虚拟化技术

虚拟机，占用资源多，启动慢，实际上不需要那么多额外的硬件

Linux容器，LXC，可以将软件运行所需的资源打包到一个隔离容器中，不需要捆绑一整套操作系统，高效轻量，在任何环境中的软件都能始终如一地运行。

三要素：镜像容器仓库 





**镜像image** 就是模板

**容器container** 就是镜像的一个实例，是独立运行的一个或者一组应用

同一个镜像可创建多个容器

**仓库repository** 集中存放镜像文件的场所 



我们将程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的环境就是image镜像文件，只有通过这个镜像文件才能生成Docker容器。image文件可以看做容器的模板，Docker根据image文件生成容器的实例，同一个image文件，可以生成多个同时运行的容器实例。

加速地址

https://wsahe6x6.mirror.aliyuncs.com



运行一个容器，容器来自于镜像，首先查找本地有没有这个镜像，没有的话就会去找仓库。

运行docker时，主机是一守护进程方式运行，我们输入命令是通过客户端跟主机交互，主机运行容器。



#### 帮助命令

- docker version

- docker info

- docker --help

#### 镜像命令

- docker image    [-a] 列出所有镜像，包含中间层  [-q]镜像id   [--digests]摘要  [--no-trunc]详细id
- docker search 查找镜像，从docker仓库查，而不是阿里云 [--no-trunc]详细摘要  [-s N]罗列点赞数大于N的镜像  [--automated]
- docker pull 镜像名   拉取镜像，默认最新  
- docker rmi 镜像名
- docker rmi -f $(docker images -qa)  全部删除，使用了联合命令



#### 容器命令

- docker run 镜像名   根据镜像生成容器并运行，  [--name]给容器指定名字 [-d] [-i] 以交互式启动 [ -t]为容器重新分配一个伪终端   [-P]随机端口映射
- docker ps  正在运行的docker容器  [-a]  当前+历史   [-l]最近创建的容器  [-n] N个容器  [-q]静默模式，只显示编号  
- exit   关闭退出
- Ctrl + p + q 不关闭退出(相当于放后台)
- docker start id/名字  启动停止的容器
- docker restart id/名字 重启停止的容器
- docker stop id/名字 正常关闭的容器
- docker kill id/名字 强制关闭的容器
- docker rm [-f]   id/名字 删除已关闭的容器

**重要**

- docker run -d 启动守护式容器 ，ps查不出来（已经退出）重要的一点，Docker容器后台运行，必须要有一个前台进程，如果没有前台进程，他会觉得你没事可做就关掉了

- docker logs id 查看容器日志  [-t]时间戳  [-f]持续输出   [--tail]

-  docker top id  运行中的容器的进程

- docker inspect id 查看内部细节 （每一个容器都是一层套一层）

- 重新进入容器（从前台通过Ctrl + p + q放到后台的）

  - docker attach id 先进入
  - docker exec id  COMMAND 直接执行命令，不需要显式进入，在容器外操作。（新建一个进程，这时候exit退出的是新建的那个进程而不是原来的容器）

- 拷贝到容器外  docker cp 容器内部路径 外部路径




### 镜像

UnionFS 联合文件系统：分层轻量级并且高性能的文件系统。

底层是一个linux内核，因而每一个镜像都是一个精简版的linux系统，只有所需的东西，我们就是在跟镜像之上一层一层叠加我们的功能，就像一个花卷一样。

 我们所使用的不同的linux系统，内核（bootfs）都是一样的，不同的是rootfs，在我们拉取一个镜像的时候都是一个一个镜像叠加而来。



我们也可以基于容器创建一个镜像并commit发布到仓库中

docker commit -a="作者名"  -m="摘要" 容器id   镜像名称：版本



### 容器数据卷

就像Redis中的RDB和AOF

用于容器持久化和数据的共享，我们有时候希望容器能保存好数据，但又不希望打包成新的镜像 



- docker run -it **-v /宿主机绝对路径 ：/容器内目录**

两个文件夹数据互通



- DockerFile：  镜像的描述文件

你可以自己写一个DockerFile，过程类似于，首先你先拿一个linux的内核作为底子，然后在这个基础之上扩展，如果**想做容器数据卷**，可以使用VOLUME命令，注意只能指定容器内部路径，因为你这个镜像可以在不同的宿主机上运行，所以不能指定宿主机路径。但是会映射到宿主机的路径中，使用inspect查看。你编写的每一条命令都会成为一层花卷叠加上去。

使用**docker  build -f  dockerfile名  -t 建造者/镜像名   .**



要完成容器间的数据传递

docker run -it **--volume-from  正在运行的容器名**  镜像名（两个容器必须来自同一个镜像）

容器之间配置的信息传递，数据卷的生命周期一直持续到没有容器使用它为止。





### DockerFile

指令（按顺序执行）：

- FROM 必须在第一条，表示bootsf
- MAINTAINER   指定维护者信息
- RUN  指令，构建镜像时执行命令
- ADD src dest 复制本地文件到容器的目的地址中（会解压更强大）
- COPY src dest   
- CMD   用于指定容器**启动时**执行的命令，每个Dockerfile只能有一个CMD命令，多个CMD命令只执行最后一个。而且会被docker run 后面带的命令参数所覆盖
- EXPOSE  port1 port2  ...  暴露接口相当于run  -p
- ENV key value 环境变量
- ENTRYPOINT  用于配置容器启动后执行的命令，docker run 后面带的命令参数会追加
- VOLUME
- USER  username  指定启动时用户名
- WORKDIR  启动路径





# 重启的Docker（狂神说）

- Docker 概述
- Docker安装
- Docker命令
- Docker镜像
- 容器数据卷
- DockerFile
- Docker网络原理
- IDEA整合 Docker
- Docker compose
- Docker Swarm 























