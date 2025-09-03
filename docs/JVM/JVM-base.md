# JVM 基础

## 什么是JVM

JVM 全称是 Java Virtual Machine，中文译名 Java虚拟机。本质上是一个运行在计算机上的程序，他的职责是运行Java字节码文件。

![6d498c75-f60a-4bd8-b5e4-c6ccdc55ccd0](./images/6d498c75-f60a-4bd8-b5e4-c6ccdc55ccd0.png)

---

## JVM的功能

1. 解释和运行
   
    - 对字节码文件中的指令，实时的解释成机器码，让计算机执行

2. 内存管理
   
    - 自动为对象、方法等分配内存，自动的垃圾回收机制，回收不再使用的对象

3. 即时编译

    - 对热点代码进行优化，提升执行效率



## 常见的JVM

![常见JVM](./images/5711fc9f-5511-43b1-9217-fd5a1b526f15.png)


## 字节码文件
!!!tip
    [使用 jclasslib工具查看字节码文件]( https://github.com/ingokegel/jclasslib)

![字节码组成](./images/731c5b64-cca1-4316-b3b8-7ed6740b054e.png)


### Magic魔数

![5136e68a-2424-4c32-8f65-9f96ce8aab46](./images/5136e68a-2424-4c32-8f65-9f96ce8aab46.png)



### 主副版本号

![35862c18-43c3-44a2-9b31-c8f669fd35aa](./images/35862c18-43c3-44a2-9b31-c8f669fd35aa.png)



### 主版本号不兼容问题

![47a87490-3493-4fac-a056-3fdb9efbc2f8](./images/47a87490-3493-4fac-a056-3fdb9efbc2f8.png)



---

## 类的生命周期

### 加载

1. **类加载器**根据类的全限定名通过不同渠道以二进制流的方式获取字节码信息
    ![字节码加载](./images/7fb54692-4924-4cfa-852c-fc5ffb9d156c.png)

2. 类加载器加载完成后，Java虚拟机会将字节码中的信息保存到方法区中

3. 生成InstanceKlass对象（C++语言对象），保存类的所有信息，还包含实现特定功能比如多态的信息

4. 在堆区生成一份与方法区中数据类似的 java.lang.Class 对象，作用是在Java代码中去获取类的信息以及存储静态字段数据（JDK8之后）
!!!note
    对于开发者来说，只需要访问堆区的Class对象而不需要访问方法区中的数据，这样Java虚拟机可以很好的控制开发者访问数据的范围

![5f9358f0-ee7d-484d-8fa0-43e8334253db](./images/5f9358f0-ee7d-484d-8fa0-43e8334253db.png)

### 连接

1. 验证阶段，检测Java字节码文件是否遵守Java虚拟机规范（文件格式、元信息等）

2. 准备阶段，为静态变量分配内存并设置**初始值**(注意是赋初值不是赋值)
   
    > final修饰的基本数据类型的静态变量，准备阶段会直接赋值

    ![a8164689-4c6a-4e31-8606-e39d020705f0](./images/a8164689-4c6a-4e31-8606-e39d020705f0.png)

3. 解析阶段，将常量池中的符号引用替换为直接引用（内存地址）

### 初始化

执行静态代码块，并为静态变量赋值，执行字节码文件中clinit部分的字节码指令

![36d307ca-cb3b-44cb-8040-929facf67b64](./images/36d307ca-cb3b-44cb-8040-929facf67b64.png)

类的初始化时机：

- 访问一个类的静态变量或者静态方法，注意final修饰的变量并且等号右边是常量不会触发初始化

- 调用Class.forName(String className)方法

- new一个该类的对象时

- 执行main方法的当前类
  
  

clinit指令在特定情况下不会出现：

- 无静态代码块且无静态变量赋值语句

- 有静态变量的声明，但是没有赋值语句

- 静态变量的定义使用final关键字并且赋常量值，准备阶段直接赋值

注：**数组的创建不会导致数组中元素的类进行初始化**，**final修饰的变量如果赋值内容需要执行指令才能得出结果，会执行clinit方法进行初始化**


!!!tip "存在继承关系"

    直接访问父类的静态变量，不会触发子类的初始化

    子类的初始化clinit调用之前，会先调用父类的clinit初始化方法

![03ddc93c-7f8c-423c-a430-b0539788ffc4](./images/03ddc93c-7f8c-423c-a430-b0539788ffc4.png)

![727c2ac8-9d08-4628-bb8c-4d8e1b257967](./images/727c2ac8-9d08-4628-bb8c-4d8e1b257967.png)

### 使用

### 卸载

---

## 类加载器分类

![29a4a59c-cc42-41bb-a552-f123aa136873](./images/29a4a59c-cc42-41bb-a552-f123aa136873.png)



#### 启动类加载器

![17478a8d-ac0a-422e-ac4b-8d08788462e9](./images/17478a8d-ac0a-422e-ac4b-8d08788462e9.png)

#### 扩展类加载器

![2cc2747d-55d9-482a-98b8-76ee5f584c97](./images/2cc2747d-55d9-482a-98b8-76ee5f584c97.png)

![75db8ab0-72f2-4c09-bda9-766ed6f275e3](./images/75db8ab0-72f2-4c09-bda9-766ed6f275e3.png)

#### 应用程序类加载器



### JDK9之后的类加载器

![5b4efcf0-0db5-43f6-9fdf-1863bdc9d12a](./images/5b4efcf0-0db5-43f6-9fdf-1863bdc9d12a.png)



### 双亲委派机制

![fcb51ebf-f093-4649-aa66-5cb977cbc257](./images/fcb51ebf-f093-4649-aa66-5cb977cbc257.png)

    ![b3d8f388-bce0-4e91-b59c-611856650b65](./images/b3d8f388-bce0-4e91-b59c-611856650b65.png)

### 打破双亲委派机制

![e0076dee-e35f-4dac-afd5-8618406eabde](./images/e0076dee-e35f-4dac-afd5-8618406eabde.png)



---

### Java内存区域

![226cbaa9-73d4-418a-9bbf-079666860340](./images/226cbaa9-73d4-418a-9bbf-079666860340.png)

![ec1efced-6b0c-43fd-96b1-924a51723828](./images/ec1efced-6b0c-43fd-96b1-924a51723828.png)

#### 程序计数器

- 程序计数器（Program Counter Register）也叫PC寄存器，每个线程会通过程序计数器记录当前要执行的的字节码指令的地址。

- 在加载阶段，虚拟机将字节码文件中的指令读取到内存之后，会将原文件中的偏移量转换成内存地址。每一条字节码指令都会拥有一个内存地址。

- 在代码执行过程中，程序计数器会记录下一行字节码指令的地址。执行完当前指令之后，虚拟机的执行引擎根据程序计数器执行下一行指令。

![edf0ce65-ebb0-4589-aa54-87c336efc006](./images/edf0ce65-ebb0-4589-aa54-87c336efc006.png)

![744ba4a6-1b1a-428f-abcd-3d3114aab79e](./images/744ba4a6-1b1a-428f-abcd-3d3114aab79e.png)



#### 栈

##### Java虚拟机栈

![1ed3bbcc-c2f4-4c4b-b44e-40a08873172f](./images/1ed3bbcc-c2f4-4c4b-b44e-40a08873172f.png)

![d158976b-86ab-494d-930b-c8e2215b80a8](./images/d158976b-86ab-494d-930b-c8e2215b80a8.png)

- 局部变量表的作用是在方法执行过程中存放所有的局部变量。编译成字节码文件时就可以确定局部变量表的内容。
  
  > 起始PC和长度确定变量作用域
  
  > 局部变量表保存的内容有：实例方法的this对象，方法的参数，方法体中声明的局部变量。

![2020e9a4-89d9-4d47-a95c-b04aebb14dde](./images/2020e9a4-89d9-4d47-a95c-b04aebb14dde.png)

- 操作数栈是栈帧中虚拟机在执行指令过程中用来存放中间数据的一块区域。他是一种栈式的数据结构，如果一条指令将一个值压入操作数栈，则后面的指令可以弹出并使用该值。
  
  > 在**编译期**就可以确定操作数栈的最大深度，从而在执行时正确的分配内存大小。

- 帧数据
  
  > 当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用（编号）转换成对应的运行时常量池中的内存地址。**动态链接**就保存了编号到运行时常量池的内存地址的映射关系
  
  

> **方法出口**指的是方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，需要存储此方法出口的地址。



> **异常表**存放的是代码中异常的处理信息，包含了异常捕获的生效范围以及异常发生后跳转到的字节码指令位置。

![c0ef3996-eeb8-4b74-9a2f-63b86bb1f900](./images/c0ef3996-eeb8-4b74-9a2f-63b86bb1f900.png)

![40d970a2-9c93-4a99-8afa-7d9cc77468d9](./images/40d970a2-9c93-4a99-8afa-7d9cc77468d9.png)

##### 本地方法栈

![19e78468-487b-4300-bca8-107583214edf](./images/19e78468-487b-4300-bca8-107583214edf.png)

##### 虚拟机设置栈大小

![ecd9856a-9737-4fb0-a175-fd769b2e2bbc](./images/ecd9856a-9737-4fb0-a175-fd769b2e2bbc.png)

![bd62b31d-b441-4005-8644-8996d341d692](./images/bd62b31d-b441-4005-8644-8996d341d692.png)



#### 堆

![564a007a-3768-4b9b-a36f-52012e1530c7](./images/564a007a-3768-4b9b-a36f-52012e1530c7.png)

![21a40c04-4b90-4de7-a464-c8cbaba93716](./images/21a40c04-4b90-4de7-a464-c8cbaba93716.png)

![79c2d671-b65a-4f09-b286-6e5f323d3a54](./images/79c2d671-b65a-4f09-b286-6e5f323d3a54.png)

![55ed99c4-aac1-4731-b1b6-f2799a9fd76c](./images/55ed99c4-aac1-4731-b1b6-f2799a9fd76c.png)



#### 方法区

![ecdc350d-bbbd-4d55-9f58-f7f1dac577a8](./images/ecdc350d-bbbd-4d55-9f58-f7f1dac577a8.png)

![ ](./images/3f492ade-8565-4bce-ab58-1b16bb64b7f4.png)

![9261cba9-fa0f-4a2b-980f-e7728c50885b](./images/9261cba9-fa0f-4a2b-980f-e7728c50885b.png)

![d216ee86-ca9c-4a24-8bb4-e9693e178167](./images/d216ee86-ca9c-4a24-8bb4-e9693e178167.png)

![53330e05-2fef-44ff-bbcb-19755ce057be](./images/53330e05-2fef-44ff-bbcb-19755ce057be.png)

![0d545d61-c0c9-4be2-973d-647b78120462](./images/0d545d61-c0c9-4be2-973d-647b78120462.png)

![a21c9300-0035-40c1-a7bb-5e1b9a1416e3](./images/a21c9300-0035-40c1-a7bb-5e1b9a1416e3.png)



#### 直接内存（不属于Java运行时的内存区域）

![6515bd3f-c4d5-4c53-a68a-2f6afedd76cf](./images/6515bd3f-c4d5-4c53-a68a-2f6afedd76cf.png)

![5891cb2c-4f14-496c-b7ce-51a1b602eabc](./images/5891cb2c-4f14-496c-b7ce-51a1b602eabc.png)



### 自动垃圾回收

<img src="./images/8c25f4b6-6318-47d1-8332-8f391f6cbe87.png" title="" alt="8c25f4b6-6318-47d1-8332-8f391f6cbe87" data-align="left">

![29deccf6-6283-4c1d-93e0-8cdc288bcea4](./images/29deccf6-6283-4c1d-93e0-8cdc288bcea4.png)



#### 方法区回收

![5aa47486-8418-4395-9244-b8b7b996a2ad](./images/5aa47486-8418-4395-9244-b8b7b996a2ad.png)

![7ecaa733-d3fe-4865-b384-04a7d739bda0](./images/7ecaa733-d3fe-4865-b384-04a7d739bda0.png)

![8a0c2de6-4b1c-4f77-878a-f84f2aa79df3](./images/8a0c2de6-4b1c-4f77-878a-f84f2aa79df3.png)



#### 堆回收

![2cd56103-6b81-4d6e-8d44-ee9920c2e6f3](./images/2cd56103-6b81-4d6e-8d44-ee9920c2e6f3.png)

![81aa8041-0b62-4713-976f-e27999c0b025](./images/81aa8041-0b62-4713-976f-e27999c0b025.png)



##### 引用计数法

![a4bd3fe9-db84-48e1-8c37-06fab0b7e976](./images/a4bd3fe9-db84-48e1-8c37-06fab0b7e976.png)



##### 可达性分析

![c83e7b0d-2617-4201-8783-02456086f987](./images/c83e7b0d-2617-4201-8783-02456086f987.png)

![b07dce07-95f2-414b-80af-946c3b46b7c4](./images/b07dce07-95f2-414b-80af-946c3b46b7c4.png)

#### 引用类型

![6fec2445-67b4-4301-8ed9-6a227ccdb7dc](./images/6fec2445-67b4-4301-8ed9-6a227ccdb7dc.png)

##### 软引用

![ab087fbc-bf89-4930-a72f-fa0031ea4be0](./images/ab087fbc-bf89-4930-a72f-fa0031ea4be0.png)

> [!Warning]
>
> 注意：软引用对象也需要被强引用，否则也会被回收



![f36fa4c2-82b9-4656-88a6-c9def9c7eabc](./images/f36fa4c2-82b9-4656-88a6-c9def9c7eabc.png)

> 注：工具-->Caffeine缓存库



![d0fd342e-aa80-4425-9eea-4b99839254ba](./images/d0fd342e-aa80-4425-9eea-4b99839254ba.png)

```java
public class Demo{
    public static void main(String[] args){
        ArrayList<SoftReference> softReferences = new ArrayList<>();
        ReferenceQueue<byte[]> queues = new ReferenceQueue<byte[]>();
        for(int i=0;i<10;i++){
            byte[] bytes = new byte[1024*1024*100];
            SoftReference stuRef = new SoftReference<byte[]>(bytes,queues);//构造函数传递数据及引用队列
            softReferences.add(stuRef);
        }

        softReference<byte[]> ref = null;
        int count = 0;
        while((ref = (SoftReference<byte[]>) queues.poll()) != null){
            count++;
        }
        System.out.println(count);
    }
}
```

> 设置启动参数 -Xmx200m
> 
> 输出 ：
> 
> 9



##### 弱引用

![c9dc7ed9-5f25-4430-9c77-faa270e8cdf3](./images/c9dc7ed9-5f25-4430-9c77-faa270e8cdf3.png)

```java
import java.lang.ref.WeakReference;

public class WeakReferenceDemo {
    public static void main(String[] args) {
        byte[] bytes = new byte[1024 * 1024 * 100];
        WeakReference<byte[]> weakReference = new WeakReference<>(bytes);
        bytes = null;
        System.out.println(weakReference.get());
        System.gc();
        System.out.println(weakReference.get());
    }
}

```

> 输出：
> 
> [B@776ec8df
> null



##### 虚引用和终结器引用

![3edd66aa-08ac-4184-b4e1-dc4cb80b3488](./images/3edd66aa-08ac-4184-b4e1-dc4cb80b3488.png)



#### 垃圾回收算法

![2375c61e-164b-4f36-8228-82b7496567d2](./images/2375c61e-164b-4f36-8228-82b7496567d2.png)

![8b6e96a2-42f6-4f87-904c-b033db08760a](./images/8b6e96a2-42f6-4f87-904c-b033db08760a.png)

![e92ea9e5-453f-4844-abd8-55319796e6b4](./images/e92ea9e5-453f-4844-abd8-55319796e6b4.png)

![426fab77-8775-4325-8757-97006184dd41](./images/426fab77-8775-4325-8757-97006184dd41.png)

![9f5d3bd0-8b06-49e5-a75a-89e33e71122c](./images/9f5d3bd0-8b06-49e5-a75a-89e33e71122c.png)



##### 标记清除法

![70692ba1-33ae-4880-846b-996e3f732d81](./images/70692ba1-33ae-4880-846b-996e3f732d81.png)

> 优点：实现简单，只需要在第一阶段给每个对象维护标志位，第二阶段删除对象即可
> 
> 缺点：
> 
> - 碎片化问题，对象删除后内存中会出现很多细小的可用内存单元，如果需要较大的空间，则这些碎片无法被分配
> 
> - 分配速度慢，由于内存碎片存在，需要进行遍历寻找可用空间



##### 复制算法

![5aae0d82-92f5-4737-bb54-d0daee4e677e](./images/5aae0d82-92f5-4737-bb54-d0daee4e677e.png)

> 优点：
> 
> - 吞吐量高
> 
> - 不会发生碎片化
> 
> 缺点：内存使用效率低



##### 标记整理算法（标记压缩算法）

![673846d3-1d70-45ef-8429-aa27122f9f39](./images/673846d3-1d70-45ef-8429-aa27122f9f39.png)

> 优点：
> 
> - 内存使用率高
> 
> - 不会发生碎片化
> 
> 缺点：整理阶段效率不高



##### 分代垃圾回收算法

![8f51380a-2653-4b77-ad13-263d968111ed](./images/8f51380a-2653-4b77-ad13-263d968111ed.png)

![46cca94b-06e7-4818-8b74-8f32d597aee4](./images/46cca94b-06e7-4818-8b74-8f32d597aee4.png)

![a63baab8-5d57-4b2c-bc0f-a850240bb543](./images/a63baab8-5d57-4b2c-bc0f-a850240bb543.png)

![3e66eb26-77b3-489e-9a50-202e58906ffb](./images/3e66eb26-77b3-489e-9a50-202e58906ffb.png)

![856e1c23-0d86-4d48-a724-3e0da80e7dc4](./images/856e1c23-0d86-4d48-a724-3e0da80e7dc4.png)

> [!Caution]
>
> “Hotspot 遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了 survivor 区的 50% 时（默认值是 50%，可以通过 `-XX:TargetSurvivorRatio=percent` 来设置，参见 [issue1199](https://github.com/Snailclimb/JavaGuide/issues/1199) ），取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值”。
>
> ------
>
> 著作权归JavaGuide(javaguide.cn)所有 基于MIT协议 原文链接：https://javaguide.cn/java/jvm/jvm-garbage-collection.html



![52e4c13c-cb84-42f7-94ac-a98ef8a0d28b](./images/52e4c13c-cb84-42f7-94ac-a98ef8a0d28b.png)

![b4c90638-e405-4bc6-98b7-0fc03ca2df39](./images/b4c90638-e405-4bc6-98b7-0fc03ca2df39.png)

#### 垃圾回收器

![d57acf5f-754e-44aa-a5de-6e5c8e38c048](./images/d57acf5f-754e-44aa-a5de-6e5c8e38c048.png)



1. 组合一

![01895fe5-43a4-409b-9446-16bbe5248f9e](./images/01895fe5-43a4-409b-9446-16bbe5248f9e.png)

![8163c13f-f40c-44e5-8e64-272c503e111b](./images/8163c13f-f40c-44e5-8e64-272c503e111b.png)



2. 组合二

![05a57a17-c4b0-4bdc-819a-6fea52f6152d](./images/05a57a17-c4b0-4bdc-819a-6fea52f6152d.png)

![390698e7-18cb-40f5-8333-85c899828dea](./images/390698e7-18cb-40f5-8333-85c899828dea.png)

![120613c9-05c6-4667-8807-8dc8fe700abb](./images/120613c9-05c6-4667-8807-8dc8fe700abb.png)

![05811edb-5c10-420d-ba0f-5dbdfe186d93](./images/05811edb-5c10-420d-ba0f-5dbdfe186d93.png)



3. 组合三

![c30d14e0-2291-47ab-94b9-da34333805e8](./images/c30d14e0-2291-47ab-94b9-da34333805e8.png)

![4f095d3c-d7c6-41c5-a28e-4d955a5c96c4](./images/4f095d3c-d7c6-41c5-a28e-4d955a5c96c4.png)

![c9d32e5a-f0e8-4166-b560-704d1d318db8](./images/c9d32e5a-f0e8-4166-b560-704d1d318db8.png)



4.G1垃圾回收器

![fc247461-199e-4dbf-9f45-d850427ec295](./images/fc247461-199e-4dbf-9f45-d850427ec295.png)

![9f41f0a1-9b82-415b-b50f-2a8050858814](./images/9f41f0a1-9b82-415b-b50f-2a8050858814.png)

![b4604f0c-1d14-4549-b45a-9557a0aaad93](./images/b4604f0c-1d14-4549-b45a-9557a0aaad93.png)

![81d56fed-1a64-4f56-90c9-ea2db2922035](./images/81d56fed-1a64-4f56-90c9-ea2db2922035.png)

1. 新创建的对象会存放在Eden区。当G1判断年轻代区不足（max默认60%），无法分配对象时需要回收时会执行Young GC。

2. 标记出Eden和Survivor区域中的存活对象，

3. 根据配置的最大暂停时间选择某些区域将存活对象复制到一个新的Survivor区中（年龄+1），清空这些区域。

4. 后续Young GC时与之前相同，只不过Survivor区中存活对象会被搬运到另一个Survivor区。

5. 当某个存活对象的年龄到达阈值（默认15），将被放入老年代。

6. 部分对象如果大小超过Region的一半，会直接放入老年代，这类老年代被称为Humongous区。比如堆内存是4G，每个Region是2M，只要一个大对象超过了1M就被放入Humongous区，如果对象过大会横跨多个Region。

7. 、多次回收之后，会出现很多Old老年代区，此时总堆占有率达到阈值时（-XX:InitiatingHeapOccupancyPercent默认45%）会触发混合回收MixedGC。回收所有年轻代和部分老年代的对象以及大对象区。采用复制算法来完成。
   
   

> G1在进行Young GC的过程中会去记录每次垃圾回收时每个Eden区和Survivor区的平均耗时，以作为下次回收时的参考依据。这样就可以根据配置的最大暂停时间计算出本次回收时最多能回收多少个Region区域了。比如 -XX:MaxGCPauseMillis=n（默认200），每个Region回收耗时40ms，那么这次回收最多只能回收4个Region。



![bac91028-93cc-4f35-9866-b75dee96b94e](./images/bac91028-93cc-4f35-9866-b75dee96b94e.png)

G1对老年代的清理会选择存活度最低的区域来进行回收，这样可以保证回收效率最高，这也是G1（Garbage first）名称的由来。最后清理阶段使用复制算法，不会产生内存碎片。

> 注意：如果清理过程中发现没有足够的空Region存放转移的对象，会出现Full GC。单线程执行标记-整理算法，此时会导致用户线程的暂停。所以尽量保证应该用的堆内存有一定多余的空间。



![441391f1-96ba-4c6d-bb7e-c2f58befe751](./images/441391f1-96ba-4c6d-bb7e-c2f58befe751.png)



#### 垃圾回收器选择

![a0762648-fd06-4997-abbc-cec1677cd6cd](./images/a0762648-fd06-4997-abbc-cec1677cd6cd.png)

---





