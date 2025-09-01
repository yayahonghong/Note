# Docker概述

Docker 是一个用于开发、发布和运行应用程序的开放平台。Docker 使您能够将应用程序与基础设施分离，以便您可以快速交付软件。借助 Docker，您可以像管理应用程序一样管理基础设施。通过利用 Docker 的方法来传送、测试和部署代码，您可以显著减少编写代码和在生产中运行代码之间的延迟。

### Docker 的核心概念：
- **镜像（Image）**：Docker 镜像是一个只读模板，包含了运行应用程序所需的所有文件、依赖和配置。镜像是容器的基础。
- **容器（Container）**：容器是镜像的运行实例。容器是轻量级的、可移植的，并且与主机环境隔离。
- **Docker Hub**：Docker Hub 是一个公共的镜像仓库，用户可以从中拉取和推送镜像。

### Docker 的优势：
- **轻量级**：容器共享主机的操作系统内核，因此比虚拟机更轻量。
- **可移植性**：容器可以在任何支持 Docker 的环境中运行，确保开发、测试和生产环境的一致性。
- **隔离性**：每个容器都运行在独立的环境中，互不干扰。

---

# 安装Docker

### CentOS 7 安装 Docker

1. **卸载旧版本 Docker**（如果存在）：
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

2. **安装必要的工具**：
   ```bash
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

3. **配置 Docker 的 yum 源**（使用阿里云源）：
   ```bash
   sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
   ```

4. **更新 yum 缓存**：
   ```bash
   sudo yum makecache fast
   ```

5. **安装 Docker**：
   ```bash
   yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

6. **启动 Docker 并设置开机自启**：
   ```bash
   systemctl start docker
   systemctl enable docker
   ```

7. **验证 Docker 是否安装成功**：
   ```bash
   docker --version
   docker ps  # 如果不报错，说明 Docker 安装并启动成功
   ```

8. **配置镜像加速**：
   ```bash
   mkdir -p /etc/docker
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
   systemctl daemon-reload
   systemctl restart docker
   ```

---

# 部署应用

### 以 MySQL 为例

1. **运行 MySQL 容器**：
   
   ```bash
   docker run -d \
       --name mysql \
       -p 3306:3306 \
       -v $PWD/conf:/etc/mysql/conf.d \
       -v $PWD/logs:/logs \
       -v $PWD/data:/var/lib/mysql \
       -e TZ=Asia/Shanghai \
       -e MYSQL_ROOT_PASSWORD=123456 \
       mysql
   ```
   
   - `-d`：使容器在后台运行。
   - `--name`：为容器设置名称。
   - `-p`：端口映射，格式为 `宿主机端口:容器端口`。
   - `-e`：设置环境变量。
   - `mysql`：指定运行的镜像名称。
   
2. **查看容器日志**：
   ```bash
   docker logs mysql
   ```

3. **进入容器内部**：
   ```bash
   docker exec -it mysql bash
   ```

---

# 镜像&容器

### 常用命令

1. **拉取镜像**：
   ```bash
   docker pull [image_name]:[version]
   ```

2. **查看本地镜像**：
   ```bash
   docker images
   ```

3. **删除镜像**：
   ```bash
   docker rmi [image_name]:[version]
   ```

4. **查看容器**：
   ```bash
   docker ps -a  # 查看所有容器
   ```

5. **删除容器**：
   ```bash
   docker rm [container_name]
   ```

6. **停止容器**：
   ```bash
   docker stop [container_name]
   ```

7. **启动容器**：
   ```bash
   docker start [container_name]
   ```

8. **进入容器内部**：
   ```bash
   docker exec -it [container_name] bash
   ```

---

# 数据卷

### 数据卷操作

1. **创建数据卷**：
   ```bash
   docker volume create [volume_name]
   ```

2. **查看所有数据卷**：
   ```bash
   docker volume ls
   ```

3. **删除数据卷**：
   ```bash
   docker volume rm [volume_name]
   ```

4. **查看数据卷详情**：
   ```bash
   docker volume inspect [volume_name]
   ```

5. **挂载数据卷**：
   ```bash
   docker run -d \
       --name nginx \
       -p 80:80 \
       -v html:/usr/share/nginx/html \
       nginx
   ```

6. **挂载本地目录**：
   ```bash
   docker run -d \
       --name nginx \
       -p 80:80 \
       -v /path/to/local/dir:/usr/share/nginx/html \
       nginx
   ```

---

# 自定义镜像

### Dockerfile 示例

```dockerfile
FROM centos:7  # 指定基础镜像
ENV MY_ENV="value"  # 设置环境变量
COPY ./app /app  # 拷贝本地文件到镜像
RUN yum install -y nginx  # 执行命令
EXPOSE 80  # 暴露端口
ENTRYPOINT ["nginx", "-g", "daemon off;"]  # 容器启动命令
```

### 构建镜像

```bash
docker build -t myapp:1.0 .
```

---

# 网络

### 常用命令

1. **查看网络列表**：
   ```bash
   docker network ls
   ```

2. **创建自定义网络**：
   ```bash
   docker network create my_network
   ```

3. **将容器连接到网络**：
   ```bash
   docker network connect my_network [container_name]
   ```

4. **查看网络详情**：
   ```bash
   docker network inspect my_network
   ```

---

# Docker Compose 快速部署

### 示例 `docker-compose.yml`

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
```

### 常用命令

1. **启动服务**：
   ```bash
   docker-compose up -d
   ```

2. **停止服务**：
   ```bash
   docker-compose down
   ```

3. **查看服务状态**：
   ```bash
   docker-compose ps
   ```

---

# 常见问题

1. **容器无法启动**：
   ```bash
   docker logs [container_name]
   ```

2. **端口冲突**：
   ```bash
   netstat -tuln | grep [port]
   ```



# 常用部署脚本

## MySQl

```bash
docker run -d \
    --name mysql \
    -p 3306:3306 \
    -v /docker/mysql/conf:/etc/mysql/conf.d \
    -v /docker/mysql/logs:/logs \
    -v /docker/mysql/data:/var/lib/mysql \
    -e TZ=Asia/Shanghai \
    -e MYSQL_ROOT_PASSWORD=Qwer1234! \
    mysql \
    --character-set-server=utf8mb4 \
    --collation-server=utf8mb4_unicode_ci
```

配置文件

```properties
[mysqld]
log_error = /logs/mysql-error.log
general_log = 0          # 默认关闭，需要时再开启（会产生大量日志）

slow_query_log = 1       # 建议开启慢查询
slow_query_log_file = /logs/mysql-slow.log
long_query_time = 2      # 超过 2 秒的查询视为慢查询
```





## Redis

```bash
docker run -d \
    --name redis \
    -p 6379:6379 \
    --log-opt max-size=100m \
    --log-opt max-file=2 \
    -v /docker/redis/conf/redis.conf:/etc/redis/redis.conf \
    -v /docker/redis/data:/data \
    -v /docker/redis/logs:/logs \
    redis \
    redis-server /etc/redis/redis.conf --requirepass Qwer1234!
```

> [!Tip]
>
> **密码设置**：
>
> - 建议在配置文件中设置密码，而不是命令行参数
> - 在`redis.conf`中添加：`requirepass 强密码`



配置文件示例(`/docker/redis/conf/redis.conf`)

```properties
bind 0.0.0.0
port 6379
protected-mode no
daemonize no
requirepass 你的强密码
appendonly yes
dir /data
logfile "/logs/redis.log"
```



## RabbitMQ

```bash
docker run -d \
  --name rabbitmq \
  --hostname my-rabbit \
  -p 5672:5672 \
  -p 15672:15672 \
  -p 25672:25672 \
  -v /docker/rabbitmq/data:/var/lib/rabbitmq \
  -v /docker/rabbitmq/conf:/etc/rabbitmq \
  -v /docker/rabbitmq/logs:/var/log/rabbitmq \
  -v /docker/rabbitmq/plugins:/plugins \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=123456 \
  --restart unless-stopped \
  rabbitmq
```

> [!Tip]
>
> 5672                  # AMQP 协议端口
>
> 15672                # 管理界面端口
>
> 25672                # 集群通信端口
>
> `-e RABBITMQ_ERLANG_COOKIE='secret_cookie' \`  # 集群需要



配置文件

```properties
# 基础配置
loopback_users.guest = false
listeners.tcp.default = 5672

# 内存和磁盘设置
vm_memory_high_watermark.relative = 0.6
disk_free_limit.absolute = 2GB

# 集群配置
# cluster_formation.peer_discovery_backend = rabbit_peer_discovery_classic_config
# cluster_formation.classic_config.nodes.1 = rabbit@rabbit1
# cluster_formation.classic_config.nodes.2 = rabbit@rabbit2

# 日志设置
log.file.level = info
log.file.rotation.date = $D0
log.file.rotation.size = 50MB
```



## Nginx

```bash
docker run -d \
  --name nginx \
  -p 80:80 \
  -p 443:443 \
  -v /docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /docker/nginx/conf.d:/etc/nginx/conf.d:ro \
  -v /docker/nginx/html:/usr/share/nginx/html \
  -v /docker/nginx/logs:/var/log/nginx \
  -v /docker/nginx/certs:/etc/nginx/certs:ro \
  --restart unless-stopped \
  nginx:latest
```



主配置文件 `/docker/nginx/conf/nginx.conf`

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    keepalive_timeout  65;

    include /etc/nginx/conf.d/*.conf;
}
```

站点配置 `/docker/nginx/conf.d/default.conf`

```nginx
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```



## Nacos

```bash
mkdir -p /docker/nacos/standalone/{logs,conf,data}

docker run -d \
  --name nacos-standalone \
  -p 8848:8848 \
  -e MODE=standalone \
  -e SPRING_DATASOURCE_PLATFORM=mysql \
  -e MYSQL_SERVICE_HOST=localhost \
  -e MYSQL_SERVICE_PORT=3306 \
  -e MYSQL_SERVICE_USER=root \
  -e MYSQL_SERVICE_PASSWORD=123 \
  -e MYSQL_SERVICE_DB_NAME=nacos_config \
  -v /docker/nacos/standalone/logs:/home/nacos/logs \
  -v /docker/nacos/standalone/conf:/home/nacos/conf \
  --restart unless-stopped \
  nacos/nacos-server:latest
```

> [!Tip]
>
> MySQL数据库脚本
>
> [nacos/distribution/conf/mysql-schema.sql at master · alibaba/nacos](https://github.com/alibaba/nacos/blob/master/distribution/conf/mysql-schema.sql?spm=5238cd80.2ef5001f.0.0.3f613b7chnlORI&file=mysql-schema.sql)



## ElasticSearch

```bash
mkdir -p /docker/elasticsearch/{data,config,logs}
chmod -R 1000:1000 /docker/elasticsearch  # Elasticsearch使用uid 1000

docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms1g -Xmx1g" \
  -v /docker/elasticsearch/data:/usr/share/elasticsearch/data \
  -v /docker/elasticsearch/config:/usr/share/elasticsearch/config \
  -v /docker/elasticsearch/logs:/usr/share/elasticsearch/logs \
  --ulimit nofile=65536:65536 \
  --ulimit memlock=-1:-1 \
  --restart unless-stopped \
  elasticsearch
```

> [!Tip]
>
>  ` -e "xpack.security.enabled=false" \`  # 开发环境可关闭安全认证



## PostgreSQL

```bash
docker run -d \
  --name postgres \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=mydb \
  -v /docker/postgres/data:/var/lib/postgresql/data \
  -v /docker/postgres/config:/etc/postgresql \
  --restart unless-stopped \
  postgres
```



## MongoDB

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=123456 \
  -v /docker/mongodb/data:/data/db \
  -v /docker/mongodb/config:/data/configdb \
  mongo
```



## Zookeeper

```bash
docker run -d \
--name zookeeper \
--privileged=true \
-p 2181:2181 \
-v /docker/zookeeper/data:/data \
-v /docker/zookeeper/conf:/conf \
-v /docker/zookeeper/logs:/datalog \
zookeeper
```

