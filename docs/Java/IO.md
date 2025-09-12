# IO

IO 是 Input/Output 的缩写，指的是计算机系统中数据的输入和输出操作。

在 Java 中，IO 主要涉及文件操作、网络通信、数据流处理等方面。

Java 提供了丰富的 IO 类库，支持多种数据源和数据格式的读写操作。

## IO流

Java IO 流根据数据处理的方式可以分为字节流和字符流：

- **字节流**：以字节为单位，读写数据，适用于任何类型的文件，包括文本、图片、音频等。
- **字符流**：以字符为单位，读写数据，主要用于处理文本文件。

<br>

Java IO 流的核心由四个抽象类组成，它们是：

- *InputStream* / *OutputStream*：字节输入/输出流的基类。
- *Reader* / *Writer*：字符输入/输出流的基类。

<br>

**常用的 Java IO 流类**

- *FileInputStream* / *FileOutputStream*：用于读取和写入文件的字节流。
- *FileReader* / *FileWriter*：用于读取和写入文件的字符流。
- *BufferedReader* / *BufferedWriter*：带有缓冲区的字符流，提高读写效率。
- *DataInputStream* / *DataOutputStream*：用于读取和写入基本数据类型的数据流。
- *ObjectInputStream* / *ObjectOutputStream*：用于读取和写入序列化对象的流。



|       角色       |         说明         |                典型实现类                |
| :--------------: | :------------------: | :--------------------------------------: |
|    **节点流**    |  直接操作数据源的流  |      `FileInputStream`/`FileReader`      |
|    **处理流**    | 对现有流进行包装增强 |     `BufferedInputStream`/`Scanner`      |
|    **转换流**    | 字节流与字符流的桥梁 | `InputStreamReader`/`OutputStreamWriter` |
| **对象序列化流** |  处理Java对象持久化  | `ObjectInputStream`/`ObjectOutputStream` |



## IO模型

### BIO-阻塞式IO

- **同步阻塞模型**：线程在读写操作时完全**阻塞**
- **1:1线程连接**：每个连接需要独立线程处理
- **简单直观**：编程模型简单，适合低并发场景

```java
// 典型BIO服务器实现
ServerSocket server = new ServerSocket(8080);
while(true) {
    Socket client = server.accept(); // 阻塞等待连接
    new Thread(() -> {
        InputStream in = client.getInputStream();
        // 阻塞读取数据
        byte[] buffer = new byte[1024];
        int len = in.read(buffer); // 阻塞点
        // 处理请求...
    }).start();
}
```



---

|    优点    |        缺点        |
| :--------: | :----------------: |
|  编程简单  |   线程资源消耗大   |
|  调试方便  | 并发量受限于线程数 |
| 适合短连接 |   线程切换开销高   |

---



### NIO-非阻塞IO

- **同步非阻塞模型**：通过Selector实现多路复用
- **事件驱动机制**：通过Channel注册感兴趣的事件
- **单线程处理多连接**：基于就绪选择机制



**核心组件**

|     组件     |       作用       |
| :----------: | :--------------: |
| **Channel**  | 双向数据传输通道 |
|  **Buffer**  |    数据缓存区    |
| **Selector** |  多路事件监听器  |



```java
// NIO服务器核心代码
Selector selector = Selector.open();
ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.bind(new InetSocketAddress(8080));
ssc.configureBlocking(false);
ssc.register(selector, SelectionKey.OP_ACCEPT); // 注册accept事件

while(true) {
    selector.select(); // 阻塞直到有就绪事件
    Set<SelectionKey> keys = selector.selectedKeys();
    Iterator<SelectionKey> iter = keys.iterator();
    
    while(iter.hasNext()) {
        SelectionKey key = iter.next();
        if(key.isAcceptable()) {
            // 处理新连接
            SocketChannel sc = ((ServerSocketChannel)key.channel()).accept();
            sc.configureBlocking(false);
            sc.register(selector, SelectionKey.OP_READ);
        } else if(key.isReadable()) {
            // 处理读事件
            SocketChannel sc = (SocketChannel)key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            sc.read(buffer); // 非阻塞读取
            // 处理数据...
        }
        iter.remove();
    }
}
```



### AIO-异步IO

- **真正的异步非阻塞**：内核完成IO后回调通知
- **Proactor模式**：应用程序不参与IO过程
- **回调驱动**：通过CompletionHandler处理结果



```java
// AIO服务器示例
AsynchronousServerSocketChannel server = 
    AsynchronousServerSocketChannel.open().bind(new InetSocketAddress(8080));

// 异步接收连接
server.accept(null, new CompletionHandler<AsynchronousSocketChannel, Void>() {
    @Override
    public void completed(AsynchronousSocketChannel client, Void attachment) {
        server.accept(null, this); // 继续接收新连接
        
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        // 异步读操作
        client.read(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {
            @Override
            public void completed(Integer result, ByteBuffer buf) {
                // 处理读取到的数据
                buf.flip();
                // 异步写回
                client.write(buf);
            }
            
            @Override
            public void failed(Throwable exc, ByteBuffer buf) {
                exc.printStackTrace();
            }
        });
    }
    
    @Override
    public void failed(Throwable exc, Void attachment) {
        exc.printStackTrace();
    }
});
```





|  模型   |     英文全称     | JDK版本 |      核心特点       |          适用场景          |
| :-----: | :--------------: | :-----: | :-----------------: | :------------------------: |
| **BIO** |   Blocking I/O   |  1.0+   |      同步阻塞       | 连接数少(<1000)的固定架构  |
| **NIO** | Non-blocking I/O |  1.4+   | 同步非阻塞/多路复用 |  高并发连接(如聊天服务器)  |
| **AIO** | Asynchronous I/O |  1.7+   |     异步非阻塞      | 长连接大文件操作(如云存储) |





1. **BIO适用场景**：
    - 客户端应用开发
    - 内部系统通信
    - 连接数可预估的服务器
2. **NIO适用场景**：
    - 高并发Web服务器(Tomcat 8+默认NIO)
    - 实时通信系统(如WebSocket)
    - P2P网络应用
3. **AIO适用场景**：
    - 大型文件上传下载
    - 数据库备份系统
    - 需要极致性能的金融交易系统

---
**完**