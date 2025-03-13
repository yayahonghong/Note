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





# 动态代理







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
