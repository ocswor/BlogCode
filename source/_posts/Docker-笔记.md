---
title: Docker 笔记
meta: Docker
date: 2018-05-28 10:33:34
tags: Docker
categories: 服务器技术
keywords: Docker
---

# 安装
两种安装方式

[apt-get](https://docs.docker.com/install/linux/docker-ce/ubuntu/#prerequisites)

修改免sudo 调用权限

Run this command in your favourite shell and then completely log out of your account and log back in (**if in doubt, reboot!**):[参考](https://techoverflow.net/2017/03/01/solving-docker-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket/)
```code
sudo usermod -a -G docker $USER
```

[convenience-script](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-convenience-script)

# 小试牛刀
使用 别人的mysql docker 在本地部署一个mysql服务

```code
docker run -d -e MYSQL_ROOT_PASSWORD=admin --name mysql -v /data/mysql/data:/var/lib/mysql -p 3306:3306 mysql
```
-d 以守护进程运行
-e 配置环境变量 设置用户名密码 
--name 镜像的名字
-v  volume 独立于容器之外的持久存储挂载，本地的/data/mysql/data 到 /var/lib/mysql下面
-p 端口映射 本地端口：mysql暴露的端口
mysql 要部署的镜像名字（本地没有，会从register 拉取远程镜像）
```code
docker run -d -e MYSQL_ROOT_PASSWORD=admin --name mysql -v /data/mysql/my.cnf:/etc/mysql/my.cnf 
-v /data/mysql/data:/var/lib/mysql -p 3306:3306 mysql 
```
这样，即可修改配置文件，还能把数据存在本地目录，一举两得，**volume -v参数可以多次使用**，每次映射一个目录，通过这种方式，很容易进行配置。

# 容器和虚拟机 与 CMD命令
CMD 容器启动命令
Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。
```code
CMD service nginx start
```
然后发现容器执行后就立即退出了。甚至在容器内去使用 systemctl 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。
正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：
```code
CMD ["nginx", "-g", "daemon off;"]
```
[容器参考](https://love2.io/@ayamefing/doc/docker_practice/image/dockerfile/cmd.md)


# 如何进入容器？

```code
docker run -dit ubuntu
```
增加 -dit 保持容器进程运行状态，不然会立即退出
查看当前 容器状态 目前有两个正在运行的容器
```code
yijialiang@yijialiang:~/mydocker/mynginx$ docker ps
CONTAINER ID        IMAGE                            COMMAND             CREATED             STATUS              PORTS                NAMES
93f0d5043702        ubuntu                           "/bin/bash"         2 hours ago         Up 2 hours                               serene_goldstine
74e446990c2c        dockerfiles/django-uwsgi-nginx   "supervisord -n"    2 hours ago         Up 2 hours          0.0.0.0:81->80/tcp   dreamy_hawking

```
执行如下命令 可以进入容器，类似于远程登录
```code
docker exec -it 74e446990c2c bash
```
# 镜像的体积
docker image ls 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

你可以通过以下命令来便捷的查看镜像、容器、数据卷所占用的空间。
```code
 docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              3                   3                   1.121GB             0B (0%)
Containers          11                  2                   34.33kB             27.07kB (78%)
Local Volumes       0                   0                   0B                  0B
Build Cache                                                 0B                  0B
```

# 镜像的创建方式
两种创建方式
### commit 第一种 在已有的镜像基础上，
执行
```code
 docker exec -it webserver bash
```
进入容器内部进行修改，退出后，
```code
$ docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2
sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
```
修改 commit 形成新的镜像
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

可以通过 如下命令查看 镜像的改动过程
```code
docker history nginx:v2
```
#### 慎用 docker commit 
[参考链接](https://love2.io/@ayamefing/doc/docker_practice/image/commit.md)
使用 docker commit 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为黑箱镜像，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。而且，即使是这个制作镜像的人，过一段时间后也无法记清具体在操作的。虽然 docker diff 或许可以告诉得到一些线索，但是远远不到可以确保生成一致镜像的地步。这种黑箱镜像的维护工作是非常痛苦的。

### 使用 Dockerfile 定制镜像
   #### FROM 指定基础镜像
   >FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。
   
   FROM scratch
   如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。
   
   不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 swarm、coreos/etcd。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 FROM scratch 会让镜像体积更加小巧。使用 Go 语言 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。
   ####   RUN 执行命令
   之前说过，Dockerfile 中每一个指令都会建立一层，RUN 也不例外。每一个 RUN 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，commit 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。
这是很多初学 Docker 的人常犯的一个错误。
```code
FROM debian:jessie

RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

# 镜像构建的上下文
[参考链接](https://love2.io/@ayamefing/doc/docker_practice/image/build.md)
```code
$ docker build -t nginx:v3 .
```
`.` 表示的就是上下文路径，Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。
>docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

所以是`.`将当前目录下的所有文件打包包括 Dockerfile 以及相关配置文件以及代码
如果上下文路径是`.`
那么
```code
COPY ./package.json /app/
```
build的时候就会出错，因为 ./package.json已将超出了上下文范围。
同时：
      支持git 仓库创建
      
      $ docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14
    
      $ docker build http://server/context.tar.gz
      
      docker会自动git clone 或者 解包，将获得文件夹认为是一个docker上下文目录进行build
      