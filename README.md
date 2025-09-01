# YSH 的技术知识体系库

> 非碎片化笔记，而是对后端核心技术的系统化梳理与深度总结。持续更新，深入原理。

---

## 📚 核心专题 | Core Topics

我将学习笔记组织成了以下核心专题，每个专题都试图从**使用、原理到实践**进行完整阐述：

### 1. 🚀  Java & JVM 生态
-   **[《JVM》](./JVM)** - JVM内存结构、垃圾回收算法、类加载机制、性能监控与调优工具（jstack, jmap, VisualVM）。
-   **[《并发编程》](./并发编程)** - Java内存模型（JMM）、synchronized、Lock、并发工具包（JUC）、原子类、线程池原理。
-   **[《Spring》](./Spring)** - IOC容器核心原理、AOP实现机制、事务管理。
-   **[《SpringBoot》](./SpringBoot)** - 自动配置原理、启动流程、内置Web容器。
-   **[《SpringCloud》](./SpringCloud)** - 微服务核心组件（Nacos, Loadbalance, Feign, Sentinal）原理与整合。

### 2. 🗃️  数据库与中间件
-   **[《MySQL》](./MySQL)** - InnoDB存储引擎、索引原理（B+Tree）、事务与锁机制（MVCC）、SQL优化与Explain实战。
-   **[《Redis》](./Redis)** - 核心数据结构与底层编码、持久化（RDB/AOF）、高可用架构（主从/哨兵/集群）、缓存问题（穿透/雪崩/击穿）解决方案。
-   **[《RabbitMQ》](./RabbitMQ)** - 消息模型、消息可靠性投递、延迟队列、集群化。
-   **[《Zookeeper》](./Zookeeper.md)** - ZAB协议、典型应用场景（分布式锁、选主）。
-   **[《Netty》](./Netty)** - NIO核心原理、Netty线程模型、编解码器、核心组件详解。

### 3. ⚙️  运维与工具
-   **[《Docker》](./Docker.md)** - 核心概念、镜像与容器、Dockerfile编写、网络与数据卷。
-   **[《Linux》](./Linux.md)** - 常用命令、Shell脚本、系统性能监控。
-   **[《Git》](./Git)** - 工作流、常用命令、原理浅析。

---

## 🎯 写作原则

-   **系统化 (Systematic)**： 构建完整的知识图谱，而非零散知识点。
-   **深度优先 (Depth-First)**： 优先探究技术背后的设计思想与源码原理。
-   **导向实践 (Practice-Oriented)**： 结合理论，给出落地实践的最佳方案和思考。

---

## 🤔 如何阅读

每个专题都是一个文件夹，内含Markdown文件，建议按顺序阅读。

---

> 此仓库是我技术成长之路的沉淀，希望对您也有帮助。如有谬误，恳请指正！
