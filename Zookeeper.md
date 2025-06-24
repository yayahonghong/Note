# *Zookeeper*

> [!Tip]
>
> 分布式系统的协调服务核心

*ZooKeeper*是一个**分布式的**、**开放源码**的分布式应用程序协调服务，它旨在为分布式应用提供一致性服务，包括配置维护、域名服务、分布式同步、组服务等核心功能



## 安装

docker安装

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



## 数据模型

ZooKeeper采用**层次化的命名空间**作为其数据模型，类似于标准的文件系统结构

1. **ZNode结构**：每个子目录项称为znode，被它所在的路径唯一标识。每个znode由3部分组成：stat(状态信息)、data(关联数据)和children(子节点列表)。
2. ZNode类型：
   - **持久节点(Persistent)**：一旦被创建，便不会意外丢失，即使服务器全部重启也依然存在
   - **临时节点(Ephemeral)**：在创建它的客户端与服务器间的Session结束时自动被删除
   - **顺序节点(Sequence)**：创建出的节点名在指定名称后带有10位十进制数的序号





## 常见命令

常用命令包括

- `create /path data`：创建节点
  - 默认创建持久节点
  - `-e` 临时节点
  - `-s` 顺序节点
- `get /path`：获取节点数据和状态
- `set /path data`：修改节点数据
- `ls /path`：列出子节点
  - `-s` 节点详细信息
- `delete /path`：删除节点



## Java API(*Curator*)

### 引入依赖

```xml
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.0.0</version>
        </dependency>
```



### 创建客户端（两种方式）

```java
    /**
     * 测试方法：演示两种方式创建 CuratorFramework 客户端并连接 Zookeeper
     */
    @Test
    public void testConnect() {
        // Zookeeper 服务地址
        String host = "localhost:2181";
        // 会话超时时间（毫秒）
        int sessionTimeoutMs = 5000;
        // 连接超时时间（毫秒）
        int connectionTimeoutMs = 5000;
        // 重试策略：初始等待3秒，最多重试10次
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(3000, 10);

        // 方式一：使用 newClient 创建客户端
        CuratorFramework client1 = CuratorFrameworkFactory.newClient(
                host,
                sessionTimeoutMs,
                connectionTimeoutMs,
                retryPolicy
        );
        client1.start(); // 启动客户端
        logger.info(client1.toString()); // 打印客户端信息

        // 方式二：使用 builder 创建客户端，并指定命名空间
        CuratorFramework client2 = CuratorFrameworkFactory.builder()
                .connectString(host)
                .sessionTimeoutMs(sessionTimeoutMs)
                .connectionTimeoutMs(connectionTimeoutMs)
                .retryPolicy(retryPolicy)
                .namespace("ysh") // 设置命名空间
                .build();
        client2.start(); // 启动客户端
        logger.info(client2.toString()); // 打印客户端信息
    }
```



### 创建节点

```java
    @Test
    public void testCreate() {
        try {
            String path = client
                    .create()
                    .withMode(CreateMode.EPHEMERAL) // 设置节点类型为临时节点
                    .forPath("/app", "hello".getBytes()); // 创建节点 /app 并设置数据为 "hello"
            logger.info(path);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
```

> [!Tip]
>
> 递归创建父节点
>
> ```java
> client.create().creatingParentsIfNeeded().forPath("/app/parent/child");
> ```



### 查询节点

```java
            // 获取节点 /app 的数据
            byte[] data = client.getData().forPath("/app");
            String s = new String(data, StandardCharsets.UTF_8);
            logger.info(s);

            // 获取节点 /app 的子节点列表
            List<String> children = client.getChildren().forPath("/app");
            logger.info(children.toString());

            // 获取节点 /app 的状态信息
            Stat stat = new Stat();
            client.getData().storingStatIn(stat).forPath("/app");
            logger.info(stat.toString());
```



### 修改节点

```java
    @Test
    public void testSet() throws Exception {
        // 修改节点 /app 的数据为 "world"
        // !!!使用CAS方法，比较版本号，确保数据一致性
        Stat stat = new Stat();
        client.getData().storingStatIn(stat).forPath("/app");
        int version = stat.getVersion();
        client.setData().withVersion(version).forPath("/app", "world".getBytes(StandardCharsets.UTF_8));
    }
```



### 删除节点

```java
    @Test
    public void testDelete() throws Exception {
        // 删除单个节点
        client.delete().forPath("/app/parent/child");

        // 删除节点及其所有子节点
        client.delete().deletingChildrenIfNeeded().forPath("/app");

        // 保证删除机制，它确保节点一定会被删除
        client.delete().guaranteed().forPath("/app");

        // 删除节点后回调
        client.delete().inBackground(new BackgroundCallback() {
            @Override
            public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) {
                logger.info(curatorEvent.toString());
            }
        }).forPath("/app");
    }
```



### Watch事件监听

> [!Tip]
>
> ZooKeeper的事件监听机制(Watches)是其核心特性之一，允许客户端监控ZNode的变化并及时获得通知。这种机制为分布式系统提供了高效的数据变更通知方式。
>
> 1. **一次性触发**：Watch是一次性的，触发后即失效，需要重新设置
> 2. **异步通知**：事件通知通过回调方式异步发送给客户端
> 3. **事件有序性**：客户端看到的事件顺序与ZooKeeper服务端发生的顺序一致
> 4. **先注册后触发**：必须先设置Watch，才能收到后续变更通知
> 5. **仅状态变化**：仅当ZNode状态发生改变时才会触发，数据内容不变但版本号变化也会触发



ZooKeeper定义了以下几种事件类型：

|      事件类型       |    触发条件    |               对应方法               |
| :-----------------: | :------------: | :----------------------------------: |
|     NodeCreated     |   节点被创建   |               exists()               |
|     NodeDeleted     |   节点被删除   | exists() / getData() / getChildren() |
|   NodeDataChanged   |  节点数据变更  |         exists() / getData()         |
| NodeChildrenChanged | 子节点列表变更 |            getChildren()             |



#### *NodeCache* - 监控单个节点

NodeCache用于监控单个节点的数据变化（创建、更新、删除）：

```java
    @Test
    public void testNodeCache() throws Exception {
        // 创建NodeCache对象
        NodeCache nodeCache = new NodeCache(client, "/app");
        // 注册监听
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                logger.info("节点状态改变");
                byte[] data = nodeCache.getCurrentData().getData();
                logger.info(new String(data, StandardCharsets.UTF_8));
            }
        });
        // 开始监听
        nodeCache.start();
    }
```



#### *PathChildrenCache* - 监控子节点

PathChildrenCache用于监控指定节点下子节点的变化（不包括该节点本身）：

```java
    @Test
    public void testPathChildrenCache() throws Exception {
        // 创建 PathChildrenCache 对象，监听 /app 路径下的子节点变化
        PathChildrenCache pathChildrenCache = new PathChildrenCache(client, "/app", true);
        // 注册监听器，处理子节点事件
        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent pathChildrenCacheEvent) throws Exception {
                logger.info("子节点事件: {}", pathChildrenCacheEvent.toString());
                switch (pathChildrenCacheEvent.getType()) {
                    case CHILD_ADDED:
                        logger.info("子节点添加: {}", pathChildrenCacheEvent.getData().getPath());
                        break;
                    case CHILD_UPDATED:
                        logger.info("子节点更新: {}", pathChildrenCacheEvent.getData().getPath());
                        byte[] data = pathChildrenCacheEvent.getData().getData();
                        logger.info("新数据: {}", new String(data, StandardCharsets.UTF_8));
                        break;
                    case CHILD_REMOVED:
                        logger.info("子节点删除: {}", pathChildrenCacheEvent.getData().getPath());
                        break;
                    default:
                        break;
                }
            }
        });
        // 启动 PathChildrenCache，开始监听子节点变化
        pathChildrenCache.start();
    }
```



#### *TreeCache* - 综合监控

TreeCache是NodeCache和PathChildrenCache的组合，可以监控整个子树的变化：

```java
    @Test
    public void testTreeCache() throws Exception {
        // 创建 TreeCache 对象，监听整个树形结构的变化
        TreeCache treeCache = new TreeCache(client, "/app");
        // 注册监听器，处理树形结构事件
        treeCache.getListenable().addListener(new TreeCacheListener() {
            @Override
            public void childEvent(CuratorFramework curatorFramework, TreeCacheEvent treeCacheEvent) throws Exception {
                logger.info("树形结构事件: {}", treeCacheEvent.toString());
            }
        });
        // 启动 TreeCache，开始监听树形结构变化
        treeCache.start();
    }
```



## 分布式锁实现

> [!Tip]
>
> 分布式锁是一种在分布式系统环境下，通过多个节点对共享资源进行访问控制的同步机制。它的主要目的是防止多个节点同时操作同一份数据，从而避免数据的不一致性。与单机环境下的锁不同，分布式锁需要跨越多个节点，确保在整个分布式系统范围内对共享资源的互斥访问。
>
> 在单机环境中，我们可以使用语言提供的同步机制（如Java的synchronized关键字或ReentrantLock类）来实现线程间的同步。但在分布式系统中，应用程序运行在多个物理或虚拟节点上，传统的本地锁机制已无法满足跨节点的同步需求

