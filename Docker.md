# Docker概述

Docker 是一个用于 [开发](https://docker.github.net.cn/get-started/overview/#)、发布和运行应用程序的开放平台。 Docker 使您能够将应用程序与基础设施分离，以便您可以快速交付软件。借助 Docker，您可以像管理应用程序一样管理基础设施。通过利用 Docker 的方法来传送、测试和部署代码，您可以显着减少编写代码和在生产中运行代码之间的延迟。



# 安装Docker

> Centos7

首先如果系统中已经存在旧的Docker，则先卸载

```bash
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine \
    docker-selinux 
```



首先要安装一个yum工具

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```




安装成功后，执行命令，配置Docker的yum源（已更新为阿里云源）：

```bash
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.reposudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```




更新yum，建立缓存

```bash
sudo yum makecache fast
```


执行命令，安装Docker

```bash
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```



启动和校验

```bash
# 启动Docker
systemctl start docker

# 停止Docker
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 执行docker ps命令，如果不报错，说明安装启动成功
docker ps
```



配置镜像加速

```bash
# 创建目录
mkdir -p /etc/docker

# 复制内容
tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "http://hub-mirror.c.163.com",
        "https://mirrors.tuna.tsinghua.edu.cn",
        "http://mirrors.sohu.com",
        "https://ustc-edu-cn.mirror.aliyuncs.com",
        "https://ccr.ccs.tencentyun.com",
        "https://docker.m.daocloud.io",
        "https://docker.awsl9527.cn"
    ]
}
EOF

# 重新加载配置
systemctl daemon-reload

# 重启Docker
systemctl restart docker
```



# 部署应用

> 以Mysql为例

```shell
docker run    #创建并运行一个容器
```



Docker运行mysql:

```shell
docker run -d \
    --name mysql \
    -p 3306:3306 \
    -e TZ=Asia/SHanghai \
    -e MYSQL_ROOT_PASSWORD=123456 \
mysql

# -d 使容器在后台运行
# --name [name] 给容器设置名称，需唯一
# -p [宿主机端口]:[容器端口] 端口映射
# -e KEY=VALUE 设置容器环境变量
# mysql 指定运行的镜像名 --> [名称]:[版本号] 不指定版本号默认使用最新版本
```







# 镜像&容器

```shell
docker pull [name]    #拉取镜像到本地

docker images    #查看本地镜像

docker rmi [name]:[version]    #删除镜像

docker ps [-a]    #查看容器

docker rm [name]    #删除容器

docker stop [name]    #停止容器

docker start [name]    #启动容器

docker exec -it [name] bash    #进入容器内部
```



# 数据卷

**数据卷**是一个虚拟目录，是**容器内目录**与**宿主机目录**之间映射的桥梁

```shell
docker volume create    #创建数据卷

docker volume ls    #查看所有数据卷

docker volume rm    #删除数据卷

docker volume inspect    #查看某个数据卷的详情

docker volume prune    #清除数据卷
```



在执行`docker run`时利用 -v [数据卷名]:[容器内目录] 挂载数据卷，若数据卷不存在会自动创建

```shell
docker run -d \
    --name nginx \
    -p 80:80 \
    -v html:/usr/share/nginx/html \
nginx
```



直接挂载本地目录

使用 -v 本地目录:容器内目录

> 本地目录必须以 "/" 或 "./" 开头，否则会被识别为数据卷





# 自定义镜像

镜像结构

**基础镜像**：应用依赖的函数库、环境、配置、文件等

**层**：添加安装包、依赖、配置等，每次操作都形成新的一层

**入口**：镜像运行入口，一般是程序启动的脚本和参数



**Dockerfile**是用来说明如何构建镜像的

```dockerfile
FROM        #指定基础镜像 FROM centos:6
ENV         #设置环境变量 ENV key value
COPY        #拷贝本地文件到镜像的指定目录
RUN         #执行Linux的shell命令
EXPOSE      #暴露端口
ENTRYPOINT  #镜像中应用启动命令，容器运行时调用 ENTRYPOINT java -jar ****.jar
```



构建镜像

```shell
docker build -t [img_name:version] .
# -t 指定镜像名
# . 指定Dockerfile所在目录，当前目录为 "."
```



# 网络

默认情况下，所有容器都是以bridge的方式连接到Docker的一个虚拟网桥上

> 问题：ip会改变，不便于容器间的访问



加入自定义网络的容器可以通过**容器名互相访问**

```shell
docker network ls

docker network create

docker network rm

docker network connect [网络名] [容器名]

docker network disconnect

docker network inspect
```



# DockerCompose快速部署

定义一组相关联的应用容器，实现快速部署

([Docker Compose | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/docker-compose.html))








































































