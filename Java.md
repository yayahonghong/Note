# 反射机制

Java 反射机制（Reflection）是 Java 提供的一种在**运行时动态获取类信息并操作类或对象的机制**。通过反射，程序可以在运行时获取类的构造方法、成员变量、方法等信息，并能够调用这些方法或访问/修改成员变量。反射机制使得 Java 程序具有更高的灵活性和动态性。

### 反射的核心类

Java 反射机制主要通过以下几个类来实现：

1. **Class 类**：表示一个类或接口，是反射的核心类。通过 `Class` 类可以获取类的构造方法、方法、字段等信息。
2. **Constructor 类**：表示类的构造方法，用于创建类的实例。
3. **Method 类**：表示类的方法，可以通过它调用类的方法。
4. **Field 类**：表示类的成员变量，可以通过它获取或修改对象的字段值。

### 反射的基本使用

#### 1. 获取 Class 对象

要使用反射，首先需要获取类的 `Class` 对象。获取 `Class` 对象的方式有以下几种：

- **通过 `Class.forName()` 方法**：

  ```java
  Class<?> clazz = Class.forName("java.lang.String");
  ```

- **通过类的 `.class` 属性**：

  ```java
  Class<?> clazz = String.class;
  ```

- **通过对象的 `getClass()` 方法**：

  ```java
  String str = "Hello";
  Class<?> clazz = str.getClass();
  ```

#### 2. 获取构造方法并创建对象

通过 `Class` 对象可以获取类的构造方法，并创建类的实例。

```java
Class<?> clazz = Class.forName("java.lang.String");
// 获取无参构造方法
Constructor<?> constructor = clazz.getConstructor();
// 创建对象
Object obj = constructor.newInstance();
```

#### 3. 获取方法并调用

通过 `Class` 对象可以获取类的方法，并调用这些方法。

java

复制

```java
Class<?> clazz = Class.forName("java.lang.String");
// 获取方法
Method method = clazz.getMethod("length");
// 创建对象
String str = "Hello";
// 调用方法
int length = (int) method.invoke(str);
System.out.println("Length: " + length);  // 输出: Length: 5
```

#### 4. 获取字段并访问/修改

通过 `Class` 对象可以获取类的字段，并访问或修改这些字段的值。

java

复制

```java
class Person {
    private String name;
    public Person(String name) {
        this.name = name;
    }
}

Class<?> clazz = Person.class;
// 获取字段
Field field = clazz.getDeclaredField("name");
// 设置字段可访问（即使是私有字段）
field.setAccessible(true);
// 创建对象
Person person = new Person("Alice");
// 获取字段值
String name = (String) field.get(person);
System.out.println("Name: " + name);  // 输出: Name: Alice
// 修改字段值
field.set(person, "Bob");
System.out.println("Updated Name: " + field.get(person));  // 输出: Updated Name: Bob
```

### 反射的优缺点

#### 优点：

1. **灵活性**：反射允许程序在运行时动态地获取类信息并操作对象，提高了程序的灵活性。
2. **通用性**：通过反射可以编写通用的代码，适用于不同类型的对象。

#### 缺点：

1. **性能开销**：反射操作比直接调用方法或访问字段要慢，因为它需要在运行时进行类型检查和访问控制。
2. **安全性问题**：反射可以访问和修改私有字段和方法，破坏了封装性，可能导致安全问题。
3. **代码可读性差**：反射代码通常比直接调用代码更难理解和维护。





# 代理

> [!Note]
>
> 代理（Proxy）是 Java 中一种重要的设计模式，用于在不修改原始类代码的情况下增强或控制对象的行为。Java 提供了多种代理实现方式，主要分为 **静态代理** 和 **动态代理**（JDK 动态代理、CGLIB 动态代理）。



## 静态代理

**特点**

- **手动编写代理类**，在编译时确定代理关系
- **代理类和目标类实现相同接口**
- **简单直接**，但每个目标类需要一个代理类

**示例**

```java
// 1. 定义接口
interface UserService {
    void save();
}

// 2. 目标类
class UserServiceImpl implements UserService {
    public void save() {
        System.out.println("保存用户");
    }
}

// 3. 静态代理类
class UserServiceProxy implements UserService {
    private UserService target;

    public UserServiceProxy(UserService target) {
        this.target = target;
    }

    public void save() {
        System.out.println("前置增强");
        target.save(); // 调用目标方法
        System.out.println("后置增强");
    }
}

// 4. 使用
public class Main {
    public static void main(String[] args) {
        UserService target = new UserServiceImpl();
        UserService proxy = new UserServiceProxy(target);
        proxy.save();
    }
}
```



|    优点    |            缺点            |
| :--------: | :------------------------: |
|  实现简单  |  每个目标类需要一个代理类  |
| 无额外依赖 | 接口修改时需同步修改代理类 |



## 动态代理

### JDK动态代理

**特点**

- **基于接口**（要求目标类必须实现接口）
- 通过 `java.lang.reflect.Proxy` 和 `InvocationHandler` 实现
- **运行时生成代理类**

**示例**

```java
// 1. 定义接口和目标类（同上）

// 2. 实现 InvocationHandler
class LogHandler implements InvocationHandler {
    private Object target;

    public LogHandler(Object target) {
        this.target = target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("方法调用前: " + method.getName());
        Object result = method.invoke(target, args); // 反射调用目标方法
        System.out.println("方法调用后");
        return result;
    }
}

// 3. 使用 Proxy 创建代理对象
public class Main {
    public static void main(String[] args) {
        UserService target = new UserServiceImpl();
        UserService proxy = (UserService) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new LogHandler(target)
        );
        proxy.save();
    }
}
```

**实现原理**

1. 运行时生成代理类（类名格式：`$Proxy0`）
2. 代理类继承 `Proxy` 并实现目标接口
3. 方法调用转发给 `InvocationHandler.invoke()`



### CGLIB动态代理

**特点**

- **基于继承**（可代理无接口的类）
- 通过 ASM 字节码框架生成子类
- 需要引入 `cglib` 依赖

**示例**

```java
// 1. 目标类（无需接口）
class UserService {
    public void save() {
        System.out.println("保存用户");
    }
}

// 2. 实现 MethodInterceptor
class LogInterceptor implements MethodInterceptor {
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("方法调用前: " + method.getName());
        Object result = proxy.invokeSuper(obj, args); // 调用父类方法
        System.out.println("方法调用后");
        return result;
    }
}

// 3. 使用 Enhancer 创建代理
public class Main {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserService.class); // 设置父类
        enhancer.setCallback(new LogInterceptor()); // 设置回调
        UserService proxy = (UserService) enhancer.create(); // 创建代理
        proxy.save();
    }
}
```

**实现原理**

1. 生成目标类的子类（如 `UserService$$EnhancerByCGLIB$$xxxx`）
2. 重写父类方法，加入拦截逻辑



## 对比

|       特性       |   静态代理   |   JDK 动态代理   |  CGLIB 动态代理  |
| :--------------: | :----------: | :--------------: | :--------------: |
| **是否需要接口** |     需要     |       需要       |      不需要      |
|   **生成方式**   |   手动编写   |  运行时动态生成  |  运行时动态生成  |
|     **性能**     |      高      | 中等（反射调用） | 高（ASM 字节码） |
|     **依赖**     |      无      |     JDK 自带     |  需引入 `cglib`  |
|   **适用场景**   | 简单少量代理 |  基于接口的代理  |  无接口的类代理  |



## **应用场景**

1. **AOP 编程**（如 Spring AOP）
2. **远程方法调用**（RPC 框架）
3. **事务管理**
4. **日志记录**
5. **权限控制**



# 不可变集合

不可变集合（Immutable Collections）是指一旦创建后就不能被修改的集合。不可变集合在**多线程环境**下非常有用，因为它们天生是线程安全的，同时也避免了意外的修改。



在 **Java 8** 及之前的版本中，可以通过 `Collections` 工具类的 `unmodifiableXXX()` 方法将可变集合转换为不可变集合。

```java
        // 创建一个可变集合
        List<String> mutableList = new ArrayList<>();
        mutableList.add("Apple");
        mutableList.add("Banana");
        mutableList.add("Cherry");

        // 转换为不可变集合
        List<String> immutableList = Collections.unmodifiableList(mutableList);
```

> 这种方法创建的不可变集合是对原始集合的视图，如果原始集合被修改，不可变集合的内容也会随之改变。





从 **Java 9** 开始，Java 提供了更简洁的工厂方法来创建不可变集合。(**推荐**)

```java
        // 创建不可变 List
        List<String> immutableList = List.of("Apple", "Banana", "Cherry");

        // 创建不可变 Set
        Set<String> immutableSet = Set.of("Apple", "Banana", "Cherry");

        // 创建不可变 Map
        Map<String, Integer> immutableMap = Map.of("Apple", 1, "Banana", 2, "Cherry", 3);
		// Map.of()具有长度限制10，可使用Map.ofEntries()，或者JDK10以上Map.copyOf()
```

> #### 注意事项：
>
> - `List.of()`、`Set.of()` 和 `Map.of()` 创建的集合是完全不可变的，且与原始集合无关。
> - 这些方法不接受 `null` 值，如果传入 `null`，会抛出 `NullPointerException`。





# 函数式编程

Java 8 引入了函数式编程的支持，主要通过 **Lambda 表达式**、**函数式接口**、**Stream API** 和 **方法引用** 等特性来实现。以下是 Java 函数式编程的核心知识点：

---

### 1. Lambda 表达式
Lambda 表达式是函数式编程的核心特性之一，它允许将函数作为方法参数传递，简化了匿名内部类的写法。

**语法：**

```java
(parameters) -> expression
(parameters) -> { statements; }
```

**示例：**

```java
// 传统匿名内部类
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};

// 使用 Lambda 表达式
Runnable r2 = () -> System.out.println("Hello World");
```

---

### 2. 函数式接口（Functional Interface）
函数式接口是只包含一个抽象方法的接口。Java 8 提供了 `@FunctionalInterface` 注解来标识函数式接口。

**常见的函数式接口：**

- `Runnable`：`void run()`
- `Supplier<T>`：`T get()`
- `Consumer<T>`：`void accept(T t)`
- `Function<T, R>`：`R apply(T t)`
- `Predicate<T>`：`boolean test(T t)`

**示例：**
```java
@FunctionalInterface
interface MyFunctionalInterface {
    void doSomething();
}

MyFunctionalInterface func = () -> System.out.println("Doing something");
func.doSomething();
```

---

### 3. Stream API
Stream API 是 Java 8 引入的用于处理集合数据的函数式编程工具。它支持链式操作，可以高效地处理数据。

**核心操作：**

- **中间操作**：`filter`, `map`, `sorted`, `distinct` 等。
- **终端操作**：`forEach`, `collect`, `reduce`, `count` 等。

**示例：**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// 过滤并打印长度大于 3 的名字
names.stream()
     .filter(name -> name.length() > 3)
     .forEach(System.out::println);

// 将名字转换为大写并收集到列表
List<String> upperCaseNames = names.stream()
                                   .map(String::toUpperCase)
                                   .collect(Collectors.toList());
```

---

### 4. 方法引用（Method Reference）
方法引用是 Lambda 表达式的一种简化写法，用于直接引用已有的方法。

**四种方法引用形式：**
1. 静态方法引用：`ClassName::staticMethod`
2. 实例方法引用：`instance::method`
3. 类的任意对象的实例方法引用：`ClassName::method`
4. 构造方法引用：`ClassName::new`

**示例：**

```java
// Lambda 表达式
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println(name));

// 方法引用
names.forEach(System.out::println);
```

---

### 5. Optional 类
`Optional` 是 Java 8 引入的用于解决 `null` 问题的容器类。它可以避免空指针异常，并提供函数式编程风格的操作。

**常用方法：**
- `of`, `ofNullable`, `empty`
- `isPresent`, `ifPresent`
- `orElse`, `orElseGet`, `orElseThrow`
- `map`, `flatMap`, `filter`

**示例：**
```java
Optional<String> name = Optional.ofNullable(getName());
name.ifPresent(System.out::println);

String result = name.orElse("Default Name");
```

---

### 6. 高阶函数
高阶函数是指接受函数作为参数或返回函数的函数。Java 中可以通过函数式接口实现高阶函数。

**示例：**

```java
public static <T> List<T> filterList(List<T> list, Predicate<T> predicate) {
    return list.stream()
               .filter(predicate)
               .collect(Collectors.toList());
}

List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
List<String> filteredNames = filterList(names, name -> name.startsWith("A"));
```

---

### 7. 并行流（Parallel Stream）
Java 8 的 Stream API 支持并行处理数据，可以充分利用多核 CPU 的优势。

**示例：**
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// 并行处理
long count = names.parallelStream()
                  .filter(name -> name.length() > 3)
                  .count();
```

---

### 8. 函数式编程的优势
- **简洁性**：代码更简洁易读。
- **不可变性**：强调不可变对象和纯函数，减少副作用。
- **并行处理**：Stream API 支持并行处理，提升性能。
- **组合性**：函数可以组合使用，提高代码复用性。

---

### 9. 注意事项
- **性能开销**：Lambda 表达式和 Stream API 在某些场景下可能带来性能开销。
- **调试困难**：Lambda 表达式的调试不如传统代码直观。
- **滥用问题**：过度使用函数式编程可能导致代码可读性下降。

---





# Stream流

Java Stream 流是 Java 8 引入的一个新的抽象层，用于处理序列（如集合）中的元素。Stream 流允许你以声明性的方式处理数据，支持诸如**过滤、映射、排序、聚合和查找**等操作，并且可以方便地进行并行处理



示例

```java
        ArrayList<String> list = new ArrayList<>();
        list.add("张三");
        list.add("张四四");
        list.add("张五");
        list.add("张六六");
        list.add("王五");

        list.stream()
                .filter(name -> name.startsWith("张"))
                .filter(name -> name.length() == 3)
                .forEach(name -> System.out.println("name = " + name));
```



流的使用步骤：

1. **创建流**：可以通过集合（如 List、Set）、数组、生成器或特定的方法来创建流。

   - 例如：`List<String> list = Arrays.asList("a", "b", "c"); Stream<String> stream = list.stream();`

   - **单列集合**如`ArrayList`、`LinkedList`可以直接使用`Collection`中的默认方法`stream()`获得流

   - **双列集合**如`HashMap`无法直接使用流，需要通过`keySet()`方法或`entrySet()`方法得到单列集合

   - **数组**可以使用`Arrays`工具类中的静态方法获得流

     

2. **中间操作**：这些操作会返回一个**新的流**，允许你进行一系列的操作。

   - 常见的中间操作有：`filter()`（过滤）、`map()`（映射）、`sorted()`（排序）、`distinct()`（去重）等。

     

3. **终端操作**：这些操作会结束流的流水线并返回一个结果（如一个值，一个集合，或者不返回任何值，如执行某些操作）。

   - 常见的终端操作有：`forEach()`（对每个元素执行操作）、`collect()`（收集结果）、`reduce()`（归约）、`count()`（计数）、`anyMatch()`（匹配任意元素）、`allMatch()`（匹配所有元素）、`noneMatch()`（不匹配任何元素）等。

     

> 请注意，Stream 流本身并不是集合类型，它并不存储元素，而是通过管道将一个集合连接到另一个集合，故不会修改原集合。
>
> 此外，<font color=red>流一旦被使用（即执行了终端操作），它就不能再被使用</font>。





# IO

## IO流

Java IO 流根据数据处理的方式可以分为字节流和字符流：

- **字节流**：以字节为单位，读写数据，适用于任何类型的文件，包括文本、图片、音频等。
- **字符流**：以字符为单位，读写数据，主要用于处理文本文件。



Java IO 流的核心由四个抽象类组成，它们是：

- *InputStream* / *OutputStream*：字节输入/输出流的基类。
- *Reader* / *Writer*：字符输入/输出流的基类。



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
