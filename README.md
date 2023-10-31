
# TechJava

**Java常见八股文，高频速记版**

- [Java基础](#JavaBasic)
- [Java并发和多线程](#JUC)
- [计算机网络](#Network)
- [操作系统](#OS)
- [关系型数据库](#DB)
- [非关系型数据库Redis](#NoSQL)
- [设计模式](#DesignPattern)
- [Spring](#Spring)
- [Mybatis](#Mybatis)
- [算法](#Algorithm)
- [智力题](#IQ)
- [进阶问题](#AdvancedpProblem)
- [面试突击问题](#tmpProblem)

# <span id='JavaBasic'> Java基础 </span>

### 1.Java中重载和重写有什么区别？


### 2.为什么重写 `equals()` 时必须重写 `hashCode()` 方法？


- `hashCode()` 作用：获取哈希码（`int` 整数），也称为散列码（用于确定该对象在哈希表中的索引位置）
- `equals()` 方法

  - **类没有重写 `equals()`方法** ：通过 `equals()`比较该类的两个对象时，等价于通过 `“==”` 比较这两个对象，使用的默认是 `Object`类 `equals()` 方法。
  - **类重写了 `equals()`方法** ：一般我们都重写 `equals()`方法来比较两个对象中的属性是否相等；若它们的属性相等，则返回 true (即，认为这两个对象相等)。

- 在 `HashSet` 中进行 `key` 比较时：

  - 如果两个对象 `hashCode()` 不相等，则直接认为这两个对象不相等，不再进行 `equals` 比较；（*逻辑短路*）

  - 如果发现有相同 `hashCode` 值的对象，这时会调用 `equals()` 方法来检查 `hashCode` 相等的对象是否真的相同。

    - 如果两者相同，`HashSet` 就不会让其加入操作成功。
    - 如果不同的话，就会重新散列到其他位置。

  > 当 `hashCode()` 不同时，才进行 `equals()` 比较，大大**减少了 `equals` 的次数，相应就大大提高了执行速度**。

- 总结 :fire:
  - 如果两个对象的 `hashCode` 值相等，那这两个对象不一定相等（哈希碰撞）。
  - 如果两个对象的 `hashCode` 值相等并且`equals()`方法也返回 `true`，这两个对象才相等。
  - 如果两个对象的 `hashCode` 值不相等，我们就可以直接认为这两个对象不相等。

---

- 为什么重写 `equals()` 时必须重写 `hashCode()` 方法？
  - 因为两个相等的对象的 `hashCode` 值必须是相等。也就是说如果 `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等。
  - 如果重写 `equals()` 时没有重写 `hashCode()` 方法的话就可能会导致 `equals` 方法判断是相等的两个对象，`hashCode` 值却不相等。

### 3.`Java` 中 `String` 为什么要设计为不可变类？


主要的原因主要有以下三点：

1. 字符串常量池的需要

   字符串常量池是 `Java` 堆内存中一个特殊的存储区域，当创建一个 `String` 对象时，**假如此字符串值已经存在于常量池中，则不会创建一个新的对象，而是引用已经存在的对象**；

2. 允许 `String` 对象缓存 `HashCode`

   `Java` 中 `String` 对象的哈希码被频繁地使用，比如在 `HashMap` 等容器中。

   **字符串不变性保证了 `hash码` 的唯一性，因此可以放心地进行缓存。**

   这也是一种性能优化手段，意味着不必每次都去计算新的哈希码

3. 保证安全性

   `String` 被许多的 `Java` 类(库)用来当做参数，例如：网络连接地址 `URL`、文件路径 `path`、还有反射机制所需要的 `String` 参数等，**假若 `String` 不是固定不变的，将会引起各种安全隐患。**

### 4.`Java` 中是如何保证 `String` 不可变的？

主要通过如下 3 个方面来保证：

1. `char[] value` 使用  `final` 修饰，保证 `value` 不会指向新的数组引用；

2. `String` 内部没有提供/暴露 修改 `value` 中元素的方法，保证 `value` 的状态不会改变；

3. `String` 类使用 `final` 修饰、`value` 数组使用 `private` 修饰，保证子类不会影响 `value` 的值。

> `JDK8` 中 `String` 类部分源码如下：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    // 使用final修饰，保证value数组不会指向新的数组引用
    private final char[] value;

    // 构造器
    public String() {
        this.value = "".value;
    }

    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }

    public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }

    public char[] toCharArray() {
        // Cannot use Arrays.copyOf because of class initialization order issues
        char result[] = new char[value.length];
        System.arraycopy(value, 0, result, 0, value.length);
        return result;
    }

    // 省略其他...
}
```

### 5.`Java` 中修改 `String` 的值但不让其指向新的引用，能做到吗？


- **能**。可以通过 “**反射**” 做到，具体操作如下

  ```java
  public static void main(String[] args)
      throws NoSuchFieldException, IllegalAccessException {
      String s = "Markdown";
      System.out.println("s.hashCode = " + s.hashCode());

      Field field = String.class.getDeclaredField("value" );
      // 暴力访问 private 字段（char[] value）
      field.setAccessible(true);
      System.out.println("\ts = " + s);
      field.set(s, new char[] {'V', 'i', 'm'});

      System.out.println("\ts = " + s);
      System.out.println("s.hashCode = " + s.hashCode());
  }
  ```

- 运行结果如下

  <div align=center><img src="https://s2.loli.net/2022/05/06/FKcshgVp7EOb4aQ.png" alt="反射-修改String" /></div>


### 6.List<?>、List<? extends T>、List<? super T> 有什么区别？

`?` 是泛型通配符，一般是使用**代替**具体的**类型实参**

> **注意**：通配符是**类型实参**，而**不是类型形参**

`<? extends T>`、`<? super T>` 是**限定通配符**，它们对类型进行了限制：

- `<? extends T>` 它通过确保类型必须是 T 的子类（或 T）来设定类型的**上界**
  - 允许读取一个泛型对象 `getter` 类型，但是不能调用 `setter` 类型方法

- `<? super T>` 它通过确保类型必须是 T 的父类（或 T）来设定类型的**下界**
  - 使用 `<? super T>` 时，**可以调用 setter 方法**，但要保证参数类型一定是 **`T` 或其子类型**；

- `<?>` 是**非限定通配符**，`?` 可以用任意类型来替代
  - `List<?>` 的意思是这个集合是一个可以持有任意类型的集合，它可以是 `List<A>`，也可以是`List<B>`

### 7.JDK1.7 和 JDK1.8 中 HashMap 有什么区别？


1. 底层数据结构

   在 JDK1.7 中 HashMap 采用的底层数据结构是 “**数组+链表**” 的形式；

   而在 JDK1.8 中 HashMap 采用的是 “**数组+链表+红黑树**” 的数据结构（当链表长度大于8且数组长度大于等于64时链表会转成红黑树，当长度低于6时红黑树又会转成链表）。

   > JDK1.8 中引入红黑树，解决 “冲突链” 过长时，查询效率低下的问题。

2. 扩容：头/尾插法、rehash 方式

   - JDK1.7 中采用 “头插法”，多线程可能导致循环链表的出现，造成多线程扩容下死循环；JDK 1.8 中采用 “尾插法”，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。
   - JDK1.7 需要重新计算每个 node 的 hash；JDK1.8 只需要看看原来的 hash 值新增的那个bit是1还是0就好了（是 0 的话索引没变，是 1 的话索引变成 "原索引+oldCap "），省去了重新计算hash值的时间。

### 8.static 关键字

**public static 方法可以被继承吗？public static 方法可以被重写吗？**

- **static 方法可以被继承**

  ```java
  class Parent {
      public static void test() {
          System.out.println("Parent.test()");
      }
  }

  class Son extends Parent {
      public static void main(String[] args) {
          Son.test(); // OK
      }
  }
  ```

- **static 方法不能被重写**

  - Java 中 static 方法不能被覆盖，因为方法覆盖是基于运行时动态绑定的；
  - 而 static 方法是编译时静态绑定的，static 方法跟类的任何实例都不相关，所以 static 方法不能被重写

  ![static方法-重写](https://s2.loli.net/2022/05/23/Z1bAjSDiUC68Bwr.png)

**执行顺序**

> 参考：[https://juejin.cn/post/6844903986475040781](https://juejin.cn/post/6844903986475040781)

- 代码块执行顺序：静态代码块 --> 构造代码块 --> 构造函数 --> 普通代码块
- 继承中代码块执行顺序：父类静态块 --> 将类静态块 --> 父类代码块 --> 父类构造器 --> 子类代码块 --> 子类构造器

### 9.BIO、NIO、AIO的区别?


- `BIO`：同步并阻塞，在服务器中实现的模式为**一个连接一个线程**。也就是说，客户端有连接请求的时候，服务器就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然这也可以通过线程池机制改善。**BIO一般适用于连接数目小且固定的架构**，这种方式对于服务器资源要求比较高，而且并发局限于应用中，是 JDK1.4 之前的唯一选择，但好在程序直观简单，易理解。
- `NIO`：同步并非阻塞，在服务器中实现的模式为**一个请求一个线程**，也就是说，客户端发送的连接请求都会注册到**多路复用器**上，多路复用轮询到有连接 IO 请求时才会启动一个线程进行处理。**NIO 一般适用于连接数目多且连接比较短（轻操作）的架构**，并发局限于应用中，编程比较复杂，从 JDK1.4 开始支持。
- `AIO`：异步并非阻塞，在服务器中实现的模式为**一个有效请求一个线程**，也就是说，客户端的 IO 请求都是通过操作系统先完成之后，再通知服务器应用去启动线程进行处理。**AIO 一般适用于连接数目多且连接比较长（重操作）的架构**，充分调用操作系统参与并发操作，编程比较复杂，从 JDK1.7 开始支持。

### 10.final、finally、finalize 的区别?

`final`

- final 变量必须初始化，通常称被修饰的变量为常量

  - final 修饰八大基本类型变量时候，变量值不可改变
  - final 修饰引用类型时，只是引用不可变，但是对象内部状态值仍然可以改变

- final 方法：被修饰的方法不允许任何子类重写，子类可以使用该方法

- final 类：被修饰的类不能被继承，所有方法不能被重写

> final 方法、final 类大多是为了“安全”考虑，避免子类中扩展修改功能。
>
> 比如，JDK 中 `String`、`Integer` 类都是 final 类，可以有效避免 API 使用者更改基础功能，某种程度上，这是保证平台安全的必要手段。

`finally`

- finally 作为异常处理的一部分，它只能在 `try/catch` 语句中，并且附带一个语句块表示这段语句最终一定被执行（无论是否抛出异常），经常被用在需要释放资源的情况下（比如，关闭 IO 流、保证 `unlock` 锁等操作）

- 更推荐使用 Java 7 中添加的 `try-with-resources` 语句，来进行关闭连接等操作

- 注意：`System.exit(0)` 可以阻断 finally 执行

  ```java
  try {
    System.exit(0);
  } finally{
    // 不会执行
    System.out.println("finally");
  }
  ```

`finalize `

- finalize 是基础类 java.lang.Object 的一个方法，一个对象的 finalize 方法只会被调用一次
- `finalize` 的设计**目的**是保证对象在被垃圾收集前完成特定资源的回收
- finalize 机制现在已经不推荐使用，并且在 JDK 9 开始被标记为 deprecated
  - finalize 被调用不一定会立即回收该对象，所以有可能调用 finalize 后，该对象又不需要被回收了；（如果 GC 之前执行 finalize 方法时，使得当前对象和 GC Root 关联，则当前对象会被“救活”）
  - 然后到了真正要被回收的时候，因为前面调用过一次，所以不会再次调用 finalize 了，进而产生问题，因此不推荐使用 finalize 方法。

### 11.说说什么是Java内存模型?
![JMM(Java内存模型） (1)](https://user-images.githubusercontent.com/50728712/182500528-adc0d85f-96c2-4602-bd1d-494d1ed81956.png)


# <span id='JUC'> Java并发和多线程 </span>

### 1.Java 中，创建线程有几种方式？

**概述**：一共有 3 种方式（线程池其实也是其中一种），如下

- 方式一：`extends Thread`
- 方式二：`implements Runnable`
- 方式三：`implements Callable`

**用法**：这三种方式的常见用法如下

- 方式一：`extends Thread` :chestnut:

  ```java
  /**
   * @author: posper
   * @version: v1.0
   * @date: 2022/05/02 20:50
   **/
  public class CreateThread1 {
      public static void main(String[] args) {
          new MyThread("thread1").start();
      }
  }

  class MyThread extends Thread {
      public MyThread(String name) {
          super(name);
      }

      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + " is running...");
      }
  }
  ```

- 方式二：`implements Runnable` :chestnut:

  ```java
  /**
   * @author: posper
   * @version: v1.0
   * @date: 2022/05/02 20:55
   **/
  public class CreateThread2 {
      public static void main(String[] args) {
          new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println(Thread.currentThread().getName() + " is running...");
              }
          }, "Thread2").start();
      }
  }
  ```

- 方式三：`implements Callable`  :chestnut:

  ```java
  import java.util.concurrent.Callable;
  import java.util.concurrent.ExecutionException;
  import java.util.concurrent.FutureTask;

  /**
   * 通过Callable接口创建线程
   *
   * @author: posper
   * @version: v1.0
   * @date: 2022/05/02 20:58
   **/
  public class CreateThread3 {
      public static void main(String[] args)
          throws ExecutionException, InterruptedException {
          // 1、创建FutureTask
          FutureTask task = new FutureTask(new Callable() {
              @Override
              public Object call() throws Exception {
                  System.out.println("call() ...");
                  return 100 + 200;
              }
          });
          // 2、创建thread对象
          Thread thread = new Thread(task);
          thread.start();
          // 3、获取线程执行结果（此时，当前main线程会被“阻塞”，直到获取get()方法的返回结果）
          Object res = task.get();
          System.out.println("res = " + res); // 300
      }
  }
  ```

**对比**

|      | 方式一：`extends Thread`     | 方式二：`implements Runnable`                                |                方式三：`implements Callable`                 |
| ---- | ---------------------------- | ------------------------------------------------------------ | :----------------------------------------------------------: |
| 优点 | 直接使用 `this` 访问当前线程 | 节省了“单继承”的位置，还可以继承其他类                       | 节省了“单继承”的位置；可以获取线程的执行返回结果；可以抛出异常 |
| 缺点 | 无法再继承其他类             | 必须通过 `Thread.currentThread()` 访问线程；不能获取线程返回结果 | **效率较低**（在获取线程的执行结果时，调用方线程会被阻塞，直至获取被调用线程的执行结果） |
| 用法 | 重写 `run()` 方法            | 重写 `run()` 方法                                            |                      重写 `call()` 方法                      |

### 2.`sleep()` VS `wait()`

**区别**

1. `sleep()` 方法是 **Thread 的静态方法**，而 `wait()` 是 **Object 类的实例方法**；

2. `wait()` 方法必须要在同步方法或者同步块中调用，也就是**必须已经获得对象锁**。而 `sleep()` 方法没有这个限制**可以在任何地方种使用**

3. `wait()`方法“会**释放**”占有的对象**锁**，使得该线程进入等待队列中，等待下一次获取资源。而 `sleep()` 方法只是**会让出 CPU 并“不会释放掉对象锁”**；

4. 调用 `sleep(long)` 方法后，线程会变成 **`TIMED_WAITING` 状态**。而调用 `wait()` 方法后线程变成 **`WAITING` 状态**

   <img src="https://s2.loli.net/2022/05/10/DusoIBf5UZYXvnx.png" alt="Java线程状态转换" style="zoom:80%;" />

5. `sleep()` 方法在休眠时间达到后，如果再次获得 CPU 时间片就会继续执行。而 `wait()` 方法**必须等待 `Object.notift()` 或 `Object.notifyAll()` 通知**后，才会**从“等待队列”中被移动到“同步队列”中**，并且再次获得 CPU 时间片才会继续执行。

   ![等待通知机制](https://s2.loli.net/2022/05/10/pUFjO5HKJ3ePA4V.png)

> 共同点：二者都可以暂停线程的执行

### 3.AQS 了解过吗？

概述

- AQS，`AbstractQueuedSynchronizer`，是用来**构建锁或者其他同步组件的基础框架**，使用 AQS 能简单且高效地构造出大量应用广泛的同步器。
  - 比如， `ReentrantLock`，`Semaphore`，`ReentrantReadWriteLock`， 等等皆是基于 AQS 的。
- `AQS` 是**对 CAS 的一种封装和丰富**，AQS 中引入了**独占锁**、**共享锁**两种模式
  - 独占：同一个时刻只有一个线程能执行，如 `ReentrantLock`；
  - 共享：同一个时刻可以有多个线程同时执行，如 `CountDownLatch`、`Semaphore`

原理概述

- 它使用了一个 int 成员变量（`int state`）表示**同步状态**，通过内置的 `FIFO` 队列（同步队列）来完成资源获取线程的排队操作（如下图所示）

- `AQS` 底层除 “同步队列”外，可能还维护了 “等待队列” （AQS 内部类 `ConditionObject` 中）

  - **同步队列**：管理中获取不到锁的线程的排队和释放；
  - **等待队列**：是在一定场景下，对同步队列的补充。比如，当缓冲区为空时，生产者是无法获取数据的，此时等待队列就会使生产者阻塞。
- AQS 围绕两个队列，**提供了四大场景**，分别是：获得锁、释放锁、等待队列的阻塞，等待队列的唤醒。

  ![image-20220427222700441](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342877.png)

### 4.AQS 使用了什么设计模式？

- `AQS ` 使用了**模板方法模式**（参考下面 “*设计模式-模板方法模式*”），继承自 `AQS` 的子类需要根据需要（独占/共享模式）去重写以下几个方法：

  ```java
  // 该线程是否正在独占资源。只有用到condition才需要去实现它。
  isHeldExclusively()

  //独占方式。尝试获取资源，成功则返回true，失败则返回false。
  tryAcquire(int)
  //独占方式。尝试释放资源，成功则返回true，失败则返回false。
  tryRelease(int)

  //共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
  tryAcquireShared(int)
  //共享方式。尝试释放资源，成功则返回true，失败则返回false。
  tryReleaseShared(int)
  ```

### 5.synchronized 和 ReentrantLock 区别是什么？

==相同点==

- 都是可重入锁（递归锁 :lock:）

==不同点==

- 具体实现不同：`synchronized` **依赖于 JVM**（JDK1.6 中对 `synchronized ` 进行了锁优化，即新增偏向锁、轻量级锁），而 `ReentrantLock` **依赖于 API**（`lock()` 和 `unlock()` 方法配合 `try/finally` 语句块来完成）

- `ReentrantLock` 比 `synchronized` **增加了一些高级功能**

  1. `ReentrantLock ` 中增加了：尝试**非阻塞的获取锁**、**可中断获取锁**、**超时获取锁**

     ![image-20220428195121805](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342094.png)

     ![image-20220428195648142](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342095.png)

  2. `ReentrantLock` 除具有非公平锁外，还具有**公平锁**；而 `synchronized ` 只能非公平锁

  3. **用 ReentrantLock 类结合 Condition 实例可以实现“选择性通知”**：`ReentrantLock` 类线程对象可以**注册在指定的 `Condition` 中**，从而可以有**选择性的进行线程通知**，在调度线程上更加灵活。 在使用`notify()/notifyAll()` 方法进行通知时，被通知的线程是**由 JVM 选择的**

==使用场景==

- 除非需要使用 `ReentrantLock` 的高级功能（上面 3 个），否则优先使用 `synchronized`；
- `synchronized` 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 `ReentrantLock` 不是所有的 JDK 版本都支持（RTL 在 JDK1.5 后的 `JUC` 包中）
- 并且使用 `synchronized` 不用担心没有释放锁而导致死锁问题，因为 **JVM 会确保锁的释放**；而使用 `ReentrantLock` 时需要**在 `finally` 块中使用 `unlock()` 保证锁一定会被释放**。

### 6.synchronized 为什么是非公平锁？非公平体现在哪些地方？



### 7.volatile 的作用是什么？它的底层原理是怎么实现的？

**作用**

- `volatile ` 可以**保证多线程可见性**（即， 变量使用 `volatile` 修饰，可以保证读到的是最新值）
- `volatile ` 会**禁止指令重排序**
  - `happens-before` 规则中的 `volatile` 变量规则：对一个 volatile 域的写，happens- before 于任意后续对这个 volatile 域的读

> 注意：`volatile` 不保证”原子性“

**原理**

- 为了弥补 CPU 和内存速度的巨大差异，CPU 不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2 或其他 cache）后再进行操作，但操作完不知道何时会写到内存
- 如果对声明了 volatile 的变量进行“写操作”，JVM 就会向 CPU 发送一条 “**Lock前缀**” 的指令，将这个变量所在缓存行（cache line）的数据写回到系统内存
  - **Lock 前缀的指令**在 “多核处理器” 下会做两件事情
    1. 将“当前处理器缓存 cache 行”的数据**写回系统内存**；
    2. 这个写回内存的操作会使得 “其他 CPU 里缓存” 了该内存地址的数据 **无效**

- 但是，就算将 `volatile` 变量写回到主内存，如果 "其他处理器缓存的值还是旧的“，再执行计算操作就会有问题。此时，则在多处理器下，为了保证各个处理器的缓存是一致的，就会实现 ”缓存一致性协议”
  - 缓存一致性协议：“每个 CPU” 通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了。当 CPU 发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成 “无效”状态。当 CPU 对这个数据进行修改操作的时候，会"重新"从系统内存中把数据读到处理器缓存里



### 8.Java 中的锁，你都了解哪些？

> 参考：[图解Java中18把锁](https://blog.csdn.net/guoguo527/article/details/118004077)

#### **乐观锁 vs. 悲观锁**

**悲观锁**

- 定义：一个共享数据加了悲观锁，那线程每次想操作这个数据前都会假设其他线程也可能会操作这个数据（*悲观主义*），所以**每次操作前都会上锁**，这样其他线程想操作这个数据拿不到锁只能阻塞了。
- 应用：Java 中 `synchronized` 和 `ReentrantLock `等就是典型的悲观锁。
- 适用场景：悲观锁**适用于写多读少**的场景，即冲突比较严重，线程间竞争激烈

**乐观锁**

- 定义：乐观锁操作数据时**不会上锁**，在更新的时候会判断一下在此期间是否有其他线程去更新这个数据。乐观锁可以使用 `版本号机制` 和 `CAS算法` 实现。
- 应用：在 Java 语言中 `java.util.concurrent.atomic` 包下的原子类就是使用 `CAS` 乐观锁实现的。
- 适用场景：乐观锁**适用于写比较少**（冲突比较小）的场景，因为不用上锁、释放锁，省去了锁的开销，从而提升了吞吐量。

#### 公平锁 vs. 非公平锁

**公平锁**

- 定义：公平锁是指多个线程按照申请锁的顺序来获取锁（即，**排队**）
- 应用：`JUC` 中 `ReentrantLock`（可以通过构造器指定选用公平锁）

**非公平锁**

- 定义：非公平锁是指多个线程获取锁的顺序并**不是按照申请锁的顺序**，有可能后申请的线程比先申请的线程优先获取锁，在高并发环境下，有可能造成优先级翻转，可能会造成某些线程 “**饥饿**”（某个线程一直得不到锁）。
- 应用：`JUC` 中 `ReentrantLock`（默认是非公平锁）

#### 独占锁 vs. 共享锁

**独占锁**

- 定义：独占锁是指锁一次**只能被一个线程所持有**。如果一个线程对数据加上排他锁后，那么**其他线程不能再对该数据加任何类型的锁**。获得独占锁的线程即能读数据又能修改数据。
- 应用：JDK中的 `synchronized` 和 `java.util.concurrent.locks` 包中 `ReentrantLock` 就是独占锁。

**共享锁**

- 定义：共享锁是指锁**可被多个线程所持有**。如果一个线程对数据加上共享锁后，那么**其他线程只能对数据再加共享锁，不能加独占锁**。获得共享锁的线程只能读数据，不能修改数据。
- 应用：在 JDK 中 `ReentrantReadWriteLock` 就是一种共享锁（读锁是共享锁，而写锁则时独占锁）

#### 读锁 vs. 写锁

**读锁**

- 定义：读锁可以在**没有写锁的时候被多个线程同时持有**
- 应用：`ReentrantReadWriteLock` 实现了 `ReadWriteLock`接口，通过 `readLock();` 便可以获取读锁

**写锁**

- 定义：写锁是独占的。写锁的优先级要高于读锁，一个获得了读锁的线程必须能看到前一个释放的写锁所更新的内容。
- 应用：`ReentrantReadWriteLock` 实现了 `ReadWriteLock`接口，通过 `writeLock();` 便可以获取写锁

#### 可重入锁

- 定义：可重入锁又称之为递归锁，是指同一个线程在外层方法获取了锁，在进入内层方法会自动获取锁。
- 应用：Java 中，`ReentrantLock` 和 `synchronized` 都是 可重入锁
- 栗子 :chestnut:

  如果一个线程调用 `methodA` 已经获取了锁再去调用 `methodB` 就不需要再次获取锁了，这就是可重入锁

  ```java
  public synchronized void mehtodA() throws Exception{
      // Do some magic tings
      mehtodB();
  }

  public synchronized void mehtodB() throws Exception{
      // Do some magic tings
  }
  ```

#### 自旋锁

- 定义：自旋锁是指线程在没有获得锁时不是被直接挂起，而是执行一个忙循环，这个忙循环就是所谓的自旋。自旋大多配合 `CAS` 一起使用。

- 自旋锁的目的：是为了**减少线程被挂起的几率**，因为线程的挂起和唤醒也都是耗资源的操作（*需要上下文切换*）

- 应用：JDK中 `AQS` 就大量适用了“自旋”

  比如，`AQS` 中的 `acquireQueued` 方法便使用了自旋锁

  ```java
  final boolean acquireQueued(final Node node, int arg) {
      boolean failed = true;
      try {
          boolean interrupted = false;
          // 自旋
          for (;;) {
              final Node p = node.predecessor();
              if (p == head && tryAcquire(arg)) {
                  setHead(node);
                  p.next = null; // help GC
                  failed = false;
                  return interrupted;
              }
              if (shouldParkAfterFailedAcquire(p, node) &&
                  parkAndCheckInterrupt())
                  interrupted = true;
          }
      } finally {
          if (failed)
              cancelAcquire(node);
      }
  }
  ```

- **自适应自旋**：自旋其实相当于 CPU 空转，这是一个是十分浪费资源的。在 JDK1.6 中，引入了自适应自旋，即 **自旋时间不再固定**，而是由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定。如果 `JVM` 认为这次自旋也很有可能再次成功那就会次序较多的时间，如果自旋很少成功，那以后可能就直接省略掉自旋过程，**避免浪费 CPU 资源**。

#### 分段锁

- 定义：分段锁设计目的是**将锁的粒度进一步细化**，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

- 应用：JDK7中，`CurrentHashMap ` 中的 `Segment` 就是典型的分段锁代表

### 9.Semaphore 场景题：共有 10 个资源，至多只能有 5 个线程能同时执行，怎么做到？

- 答：使用 `JUC` 组件 `Semaphore`

- 代码如下

  ```java
  import java.util.Random;
  import java.util.concurrent.Semaphore;
  import java.util.concurrent.TimeUnit;

  /**
   * @author: posper
   * @version: v1.0
   * @date: 2022/04/29 19:46
   **/
  public class SemaphoreTest {
      public static void main(String[] args) {
          int COUNT = 5;
          int workersNum = 10;
          Semaphore semaphore = new Semaphore(COUNT);
          for (int i = 0; i < workersNum; i++) {
              new Thread(() -> {
                  try {
                      semaphore.acquire();
                      System.out.println(Thread.currentThread().getName() + "获取资源");
                      TimeUnit.SECONDS.sleep(new Random().nextInt(5));
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
                      System.out.println("----------------"
                      	+ Thread.currentThread().getName() + "释放资源");
                      // 释放
                      semaphore.release();
                  }
              }, "Thread" + i).start();
          }
      }
  }
  ```

- 执行结果如下

  <img src="https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342097.png" alt="image-20220429195959645" style="zoom:80%;" align="center" />

### 10.什么是死锁？死锁产生的条件？如何预防、避免和检测死锁？

死锁案例说明:"哲学家进餐”

5个哲学家去吃中餐，坐在一张圆桌旁，他们有5根筷子，并且每两个人中间放一根筷子。哲学家们时而思考，时而进餐，每个人都需要一双筷子才能吃到东西，并且在吃完后将筷子放回原处，继续思考。

>什么是死锁？

- 用案例说明：每个人都立即抓住自己左边的筷子，然后等待自己右边的筷子空出来，但同时又不放下已经拿到的筷子。（即：每个人都拥有其他人需要的资源，同时又等待其他人已经获得的资源，并且每个-人在获得所有需要的资源之前都不会放弃已经拥有的资源。）

- 用术语表达：指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或称系统产生了死锁，这些永远在互相等待的进程称为死锁进程。
>死锁产生的四个必要条件

- 互斥条件：一个资源每次只能被一个进程使用。
- 请求与保持条件：一个进程因请求资源而阻塞，对已获得的资源保持不放。
- 不剥夺条件：进程已经获得的资源，在未使用完之前，不得强行剥夺。
- 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

>如何预防死锁

破坏死锁四个必要条件中的任意一个或多个，就可以避免死锁发生。

- 资源一次性分配：一次性分配所有资源，这样就不会再有请求了（破坏请求条件）
- 只要有一个资源得不到分配，也不给这个进程分配其他的资源（破坏请保持条件）
- 可剥夺资源：即当某进程获得了部分资源，但得不到其它资源，则释放已占有的资源（破坏不可剥夺条件）
- 资源有序分配法：系统给每类资源赋予一个编号，每一个进程按编号递增的顺序请求资源，释放则相反（破坏环路等待条件）
注意：资源互斥的固有特性是无法改变的。

>如何避免死锁

- 基本思想：系统对进程发出每一个系统能够满足的资源申请进行动态检查，并根据检查结果决定是否分配资源，如果分配后系统可能发生死锁，则不予分配，否则分配。这是一种动态策略。

- 典型的避免死锁的算法：有序资源分配法和银行家算法。


>死锁预防、死锁避免、死锁检测和解除
- 死锁的预防和避免都属于事先预防策略。不允许死锁发生。
- 死锁检测和解除属于事后策略。允许死锁发生。

>死锁解除：
一旦检测出死锁，就应该立即采取措施来接触死锁。死锁解除的主要方法有：

- 资源剥夺法。 挂起某些死锁进程，抢占它的资源，分配给其他死锁进程。
- 撤销进程法。 强制撤销部分进程并剥夺这些进程的资源，让其他进程顺利执行。
- 进程回退法。

# <span id='Network'> 计算机网络 </span>
## 一、综合性题目
1.描述一下浏览器从键入URL到显示页面的过程。

简略版：
![image](https://user-images.githubusercontent.com/50728712/182498918-ecb3c352-be5d-495d-b310-b8f7ff49e5c0.png)

详细版：
![有图版：浏览器：键入URL-显示页面的过程](https://user-images.githubusercontent.com/50728712/182498876-2fba47d8-2de1-4892-9e74-9b7dfd18c0db.png)

## 二、TCP相关

### 1.TCP基本认识相关要点：

![image](https://user-images.githubusercontent.com/50728712/182638711-0d11e368-8d27-402e-b74a-af5d0e6f4aef.png)

对上述要点的解答：

![TCP基本认识](https://user-images.githubusercontent.com/50728712/182639284-af0fb361-916e-462c-98b0-771d1daf81d0.png)


### 2.TCP建立连接为什么是三次握手？而不是两次或四次？

**（1）避免历史连接的初始化**

客户端连续多次发送建立连接的SYN报文，在网络拥堵的情况下：

- 一个旧的SYN报文比最新的SYN报文早到达了服务端；
- 那此时服务端就会回一个SYN+ACK报文给客户端；
- 客户端收到后可以根据自身的上下文判断这是一个历史连接（序列号过期或超时），那么客户端旧会发送RST报文给服务端，表示终止这一次连接。

如果是两次握⼿连接，就不能判断当前连接是否是历史连接，三次握⼿则可以在客户端（发送⽅）准备发送第三次 报⽂时，客户端因有⾜够的上下⽂来判断当前连接是否是历史连接：

如果是历史连接（序列号过期或超时），则第三次握⼿发送的报⽂是 RST 报⽂，以此中⽌历史连接；
如果不是历史连接，则第三次发送的报⽂是 ACK 报⽂，通信双⽅就会成功建⽴连接

**（2）同步双方的初始序列号**

序列号的作用：去重+按序+可识别哪些包已被接收

![image](https://user-images.githubusercontent.com/50728712/183411707-b10a1a6c-7d05-41c0-948d-2edfe25ca433.png)

四次握手也可以，但第二部和第三步可以优化成异步，所以仅需三次握手即可。

两次握手只保证了一方的序列号能被对方成功接收，没办法保证双方的初始序列号都能被确认接收。

**（3）避免资源浪费**

如果只有「两次握⼿」，当客户端的 SYN 请求连接在⽹络中阻塞，客户端没有接收到 ACK 报⽂，就会重新发送 SYN ，由于没有第三次握⼿，服务器不清楚客户端是否收到了⾃⼰发送的建⽴连接的 ACK 确认信号，所以 每收到⼀个 SYN 就只能先主动建⽴⼀个连接。

两次握⼿会造成消息滞留情况下，服务器重复接受⽆⽤的连接请求 SYN 报⽂，⽽造成重复分配资源。

**简而言之：**

不使⽤「两次握⼿」和「四次握⼿」的原因：

- 「两次握⼿」：⽆法防⽌历史连接的建⽴，会造成双⽅资源的浪费，也⽆法可靠的同步双⽅序列号；
- 「四次握⼿」：三次握⼿就已经理论上最少可靠连接建⽴，所以不需要使⽤更多的通信次数。

### 3.TCP断开连接，问什么需要四次挥手？

- 关闭连接时，客户端向服务端发送FIN时，仅仅表示客户端不再发送数据了但是还能接收数据。
- 服务器收到客户端的FIN报文时，先回一个ACK应答报文，而服务端可能还有数据需要处理和发送，等服务端不再发送数据时，才发送FIN报文给客户端表示同意现在关闭连接。

故而，服务端通常需要等待完成数据的发送和处理，所以服务端的 ACK 和 FIN ⼀般都会分开发送，从⽽⽐三次握⼿导致多了⼀次。

# <span id='OS'> 操作系统 </span>

## 一、进程与线程

### 1.进程和线程的区别

进程让操作系统的并发性成为了可能，而线程让进程的内部并发成为了可能。多进程的方式确实也可以实现并发，为什么要使用多线程？

使用多线程有以下几个好处：
- 通信比较容易：进程间的通信比较复杂，而进程间的通信比较简单。通常情况下，我们需要使用共享资源，而这些资源在线程间的通信比较容易
- 系统开销更小：进程是重量级的，而线程是轻量级的，故多线程的方式系统开销更小。
- 进程和线程的区别：

进程是⼀个独⽴的运⾏环境，⽽线程是在进程中执⾏的⼀个任务。他们两个本质的区别是是否单独占有内存地址空间及其它系统资源（比如I/O）：

- 数据共享与同步：进程单独占有⼀定的内存地址空间，所以进程间存在内存隔离，数据是分开的，数据共享复杂但是同步简单，各个进程之间互不⼲扰；⽽线程共享所属进程占有的内存地址空间和资源，数据共享简单，但是同步复杂。
- 崩溃后的影响：进程单独占有⼀定的内存地址空间，⼀个进程出现问题不会影响其他进程，不影响主程序的稳定性，可靠性⾼；⼀个线程崩溃可能影响整个程序的稳定性，可靠性较低。
- 系统开销的大小：进程单独占有⼀定的内存地址空间，进程的创建和销毁不仅需要保存寄存器和栈信息，还需要资源的分配回收以及⻚调度，开销较⼤；线程只需要保存寄存器和栈信息，开销较⼩。
- 基本单位：进程是操作系统进⾏资源分配的基本单位，⽽线程是操作系统进⾏调度的基本单位，即CPU分配时间的单位。

### 2.进程调度
![进程调度](https://user-images.githubusercontent.com/50728712/182857877-1f93cf15-9787-4267-9e27-85b835676d4f.png)

### 3.进程基本概念

相关要点：

![image](https://user-images.githubusercontent.com/50728712/183279657-88535a1e-462d-441d-96a0-291c6fce11f0.png)

要点解答：

![进程线程基础知识](https://user-images.githubusercontent.com/50728712/183279673-b7058793-227a-400b-8057-8704d4cf4722.png)

### 4.线程基本概念

相关要点：

![image](https://user-images.githubusercontent.com/50728712/183413301-92849d6f-c427-42aa-8a31-733baa23305c.png)

要点解答：

![线程基础知识 (1)](https://user-images.githubusercontent.com/50728712/183413378-db8a0e38-08f6-4142-b067-8f1b4a5a3e88.png)

### 5.生产者与消费者问题
![image](https://user-images.githubusercontent.com/50728712/184266665-ea9f1293-3193-4ac4-b986-1b532d652f25.png)

生产者消费者问题描述：
- 生产者将生成的数据放入缓冲区
- 消费者从缓冲区取出数据处理
- 任何时刻，只能有一个生产者或消费者可以访问缓冲区

说明：
- 任何时刻只有一个线程操作缓冲区，说明操作缓冲区是临界代码，需要互斥
- 缓冲区为空时，消费者必须等待生产者生成数据；缓冲区满时，生产者必须等待消费者取出数据。说明：生产者和消费者需要同步。

所以需要三个信号量，分别是：

- 互斥信号量 mutex ：⽤于互斥访问缓冲区，初始化值为 1；
- 资源信号量 fullBuffers ：⽤于消费者询问缓冲区是否有数据，有数据则读取数据，初始化值为 0 （表明缓冲区⼀开始为空）；
- 资源信号量 emptyBuffers ：⽤于⽣产者询问缓冲区是否有空位，有空位则⽣成数据，初始化值为 n （缓冲区⼤⼩）；

代码实现：

![image](https://user-images.githubusercontent.com/50728712/184266767-261e9992-b9d2-4355-943f-7befd6dfec71.png)

### 6.进程间通信方式

进程间通信主要包括6种：

（1）管道：大小受限且只能承载无格式的字节流
 - 匿名管道：单向传输，自行创建和销毁
 - 命名管道：创建管道文件，进行写入和读取，不适合频繁地数据通信

（2）消息队列：通信双方约定好消息类型和正文搁置，按照一个个独立单元进行发送。消息队列位于内核中，当写入或读取时，需要进行用户态和内核态的切换，数据拷贝开销太大。

（3）共享内存：不同进程的虚拟地址空间映射到相同的物理地址空间，一个进程的操作可以被另一个进程及时看到，就不需要再拷贝来拷贝去。但在进行写入时，同时操作会产生冲突。

（4）信号量：实际上是个计数器，其主要用来实现进程之间的同步和互斥。信号量定义了两种操作，p操作和v操作，p操作为申请资源，会将数值减去M，表示这部分被他使用了，其他进程暂时不能用。v操作是归还资源操作，告知归还了资源可以用这部分。

（5）信号：在操作系统中，不同信号用不同的值表示，每个信号设置相应的函数。一旦进程发送某一个信号给另一个进程，另一进程将执行相应的函数进行处理。也就是说把可能出现的异常等问题准备好，一旦信号产生就执行相应的逻辑即可。

（6）套接字（Socket)：位于不同主机的进程间通信需要使用socket

相关思维导图：
![进程间通信](https://user-images.githubusercontent.com/50728712/185739019-754980c1-aa60-41e8-85c0-6a99fecda619.png)

## 二、内存管理

### 1.页面置换算法

页面置换算法的功能是，当出现缺页异常，需调入新页面而内存已满时，选择被置换的物理⻚⾯，也就是说选择⼀个物理⻚⾯换出到磁盘，然后把需要访问的⻚⾯换⼊到物理⻚。那其算法目标则是，尽可能减少页面的换入换出的次数，常见的页面置换算法有如下几种：最佳页面置换算法（OPT）、先进先出置换算法（FIFO）、最近最久未使⽤的置换算法（LRU）、时钟⻚⾯置换算法（Lock）、最不常⽤置换算法（LFU)等。

![页面置换算法](https://user-images.githubusercontent.com/50728712/185739422-534a2758-de9e-4c49-9bf0-8f3892b1630e.png)




# <span id='DB'> 关系型数据库 </span>
1.MySQL事务特性（ACID）有哪些？具体含义是什么？

- Atomicity（原子性）：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- Consistency（一致性）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
- Isolation（隔离性）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- Durability（持久性）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

2.Mysql 事务隔离级别都有哪些？各个隔离级别之间区别是什么？
SQL标准的事务隔离级别包括：读未提交（read uncommitted）、 读提交（read committed）、可重复读（repeatable read）和串行化（serializable ）。下面逐 一解释：

- 读未提交是指，一个事务还没提交时，它做的变更就能被别的事务看到。
- 读提交是指，一个事务提交之后，它做的变更才会被其他事务看到。
- 可重复读是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
- 串行化，顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

这四个隔离级别可以解决脏读、幻读、不可重复读三类问题。先简单介绍下这三类问题的概念：

- 脏读：事务 A 读取了事务 B 更新后的数据，但是事务 B 没有提交，然后事务 B 执行回滚操作，那么事务 A 读到的数据就是脏数据。

- 不可重复读：事务 A 进行多次读取操作，事务 B 在事务 A 多次读取的过程中执行更新操作并提交，提交后事务 A 读到的数据不一致。

- 幻读：事务 A 将数据库中所有学生的成绩由 X 改为Y，此时事务 B 手动插入了一条成绩为 X 的记录，在事务 A 更改完毕后，发现还有一条记录没有修改，那么这种情况就叫做出现了幻读。

隔离级别与三类问题之间的对应关系：
![隔离级别](https://user-images.githubusercontent.com/50728712/175560168-80947948-dd18-4430-a4c3-3bf68bcd0150.png)
其中隔离级别由低到高是：读未提交 < 读已提交 < 可重复读 < 串行化。

隔离级别越高，越能够保证数据的完整性和一致性，但是对并发的性能影响越大。大多数数据库的默认级别是`读已提交`（比如 Sql Server、Oracle） ，但是 MySQL 的默认隔离级别是 `可重复读`。

## SELECT 语句是如何执行的？

1. **建立连接**

   通过 `mysql -h$ip -P$port -u$user -p` 命令，可以通过 TCP 和 MySQL 服务器建立连接，其中 “连接器” 负责用户密码校验。若密码认证通过，“连接器” 会到权限表中查出登录用户的所拥有的的权限（用户成功建立连接后，即使修改用户权限，也不会影响本次已连接的权限）。连接过程通常较为复杂，所以尽量使用 “长连接”。但若全部使用长连接，MySQL 占用内存涨得比较快，此时可以通过 “定期断开长连接” 或 “在每次执行一个比较大的操作后，通过执行 `mysql_reset_connection` 来重新初始化连接资源”（MySQL5.7以后版本）。

2. **查询缓存**

   MySQL会将查询过的结果放入缓存中（内存中），但是它比较鸡肋，因为一旦 “查询语句有一丁点改变” 或者 “数据表被更新”，都会造成缓存失效。所以，在 MySQL8.0 版本直接把查询缓存给删掉了。

3. **SQL 分析**

   此时，客户端发送的 SQL 查询语句已经传输到服务器上了，MySQL 会通过 “分析器” 对其进行词法分析（识别出每一项是什么），以及 “语法分析”（判断这个 SQL 语句是否满足 MySQL 语法）。这两个分析都合法后，才会进入下一步操作。

4. **生成执行计划**

   此时，通过语法分析后便可以知道需要干什么了。但是，一条 SQL 与可能具有多种执行方式（比如，表连接顺序、索引的选择等）。优化器会在众多可行的执行方式中选取一个代价最小的，作为“执行计划“。

5. **执行并返回结果**

   具体的执行环节，会根据优化器得到的 “执行计划“ 进行。开始执行后，会先根据连接器查询到的用户权限来判断当前用户是否具有查询的权限。如果有权限，就打开表继续执行，此时执行器会调用具体的存储引擎的接口获取查询结果集，并将之返回给客户端。

## UPDATE 语句是如何执行的？

> 以 `UPDATE student SET age = age + 1 WHERE ID = 2;` 为例.

1~4. 前四步骤和 [SELECT 语句是如何执行的？](https://t.zsxq.com/04IMNjUfm) 中相同（建立连接、查询缓存、SQL 分析、生成执行计划）；

5. **查找行记录**

   执行器调用存储引擎接口查找 `ID = 2` 对应的行记录。如果行记录所在数据页在内存中，则直接返回给执行器；否则， 需要先从磁盘中读入内存，然后再返回。

6. **更新数据**

   执行器将获取到的行记录对应 `age` 值加一，然后调用存储引擎接口将更新后的行记录**更新到内存中**。

7. **写 redo 日志**

   将更新操作记录到 `redo log` 中，此时 `redo log` 处于 **prepare 阶段**。此后，会告知执行器执行完事了，随时可以提交事务。

8. **写 binlog 日志**

   执行器生成这个操作的 `binlog`，并把 `binlog` 写入磁盘。

9. **提交**

   执行器调用存储引擎的**提交事务**接口，把刚刚写入的 `redo log` 改成 ”提交“ （**commit**）状态，更新完成。

## redo、binlog 了解吗？二者有什么区别？

- redo：先写日志，后写磁盘（WAL，Write-Ahead Loggin）。MySQL 宕掉之后，可以根据 redo log 进行重做，redo log 主要是为了实现事务的持久性。
- binlog：记录所有数据库表结构变更（例如 CREATE、ALTER TABLE）以及表数据修改（INSERT、UPDATE、DELETE）的二进制日志，但不会记录查询命令 SELECT。MySQL binlog 以事件形式记录，还包含语句所执行的消耗的时间，MySQL 的二进制日志是事务安全型的。binlog 的主要目的是复制和恢复。

|          | redo                                           | binlog                                                       |
| -------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 归属性   | InnoDB 引擎特有                                | 属于 Server 层，所有存储引擎共有                             |
| 日志内容 | 物理日志，记录的是“在某个数据页上做了什么修改” | 逻辑日志，记录的是语句的原始逻辑                             |
| 写特点   | 循环写。空间固定，会覆盖之前的日志内容         | 追加写。binlog 文件写到一定大小后会切换到下一个，并不会覆盖之前的日志 |



# <span id='NoSQL'> 非关系型数据库Redis </span>

## 1.Redis 为什么不直接使用 C 字符串（以空字符结尾的字符数组），而要自己构建一种字符串抽象类型 SDS(simple dynamic string，简单动态字符串)?

  因为比起C字符串，SDS具有以下优点：
  - 常数复杂度获取字符串长度:C字符串并不记录自身的长度信息，所以需进行遍历以获取字符串长度，时间复杂度为O(n);SDS在len属性中记录了SDS本身的长度，  所以获取字符串长度的时间复杂度仅为O(1)

  - 杜绝缓冲区溢出:SDS的空间分配策略完全杜绝了发生缓冲区溢出的可能性(当SDS API需要对SDS进行修改时，API会先检查SDS的空间是否满足修改所需的要求。若不满足的话，API会自动将SDS的空间扩展至执行修改所需的大小，然后才执行实际的修改操作)

  - 减少修改字符串长度时所需的内存重分配次数:
    - 对C字符串而言：增长字符串之前，需要通过内存重分配来拓展底层数组的空间大小，忘记的话，会产生缓冲出溢出问题；缩短字符串之前，需要通过内存重分配来释放字符串不再使用的那部分空间，忘记的话，会产生内存泄露。
    - 对于SDS而言：通过未使用空间实现的优化策略，包括：空间预分配和惰性空间释放，分别优化了字符串增长操作和字符串缩短操作。1.空间预分配策略，用于优化SDS的字符串增长操作：当SDS的API对一个SDS进行修改，并且需要对SDS进行空间拓展的时候，程序不仅会为SDS分配修改所需的空间，还会为SDS分配额外的未使用空间（通过这种预分配策略,SDS将连续增长N次字符串所需的内存重分配次数从必定N次降低为最多N次）。2.惰性空间释放，用于优化SDS的字符串缩短操作：当SDS的API需要缩短SDS保存的字符串时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用free属性将这些字节的数量记录起来，并等待将来使用（通过惰性空间释放策略，SDS避免了缩短字符串时所需的内存重分配操作，并为将来可能有的增长操作提供了优化）。

  - 二进制安全：C字符串，只能保存文本数据，而不能保存图片、音频、视频、压缩文件这样的二进制数据。SDS使用len属性值来判断字符串是否结束，故在保存特殊数据格式时没有问题.此外，SDS的API都是二进制安全的，所有SDS API都会以处理二进制的方式来处理SDS存放在buf数组里的数据，程序不会对其中的数据做任何限制、过滤或者假设，数据在写入时是什么样，它被读取时就是什么样。

  - 兼容部分C字符串函数：通过遵循C字符串以空字符结尾的惯例，SDS可以在有需要时重用<string.h>函数库，从而避免了不必要的代码重复。

## 2.简述Redis的常用数据结构及其底层实现。

Redis底层实现主要包括六种，即：简单动态字符串（SDS)、双端链表(linkedlist)、字典(ht, 即hashtable)、跳跃表(skiplist)、整数集合(intset)、压缩列表(ziplist)。

Redis常用的数据结构，根据value值进行了划分，包括五种，即：字符串对象(String)，列表对象(List)，哈希对象(Hash)，集合对象(Set)和有序集合对象(Zset)。为了平衡空间和时间效率，不同的对象在底层会采用不同的数据结构。通过这五种不同类型的对象，	Redis可以在执行命令之前，根据对象的类型来判断一个对象是否可以执行给定的命令。此外，还可以针对不同的使用场景，为对象设置多种不同的数据结构实现，从而优化对象在不同场景下的使用效率。下图展示了对象（对外数据类型）和底层实现的映射关系：
![Redis对象与底层mapping关系 (1)](https://user-images.githubusercontent.com/50728712/175186239-23ffe029-8ad3-4735-9918-ee3871c7601d.png)


# <span id='DesignPattern'> 设计模式 </span>
## 模板方法模式


### 概述

**定义**

- 模板方法模式，定义一个操作中的算法的**骨架**，而将**一些步骤延迟到子类中**。

**作用**

- 模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

**模板方法模式结构图**

![image-20220428190407376](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342791.png)

- **抽象类** AbstractClass：其实是一抽象模板，定义并实现了一个**模版方法**
  - 模版方法一般是一个具体方法（非 `abstract`，如后面栗子中的 `run()`），它给出了一个**顶级逻辑的骨架**，而逻辑的**组成步骤**在相应的抽象操作中，**推迟到子类实现**。顶级逻辑也有可能调用一些具体方法。
- **具体实现类** ConcreteClass：**实现父类**所定义的一个或多个**抽象方法**
  - 每一个 AbstractClass 都可以有任意多个 ConcreteClass 与之对应；
  - 而每一个 ConcreteClass 都可以给出这些抽象方法（也就是顶级逻辑的组成步骤）的不同实现，从而使得顶级逻辑的实现各不相同。

### 模板方法模式栗子 :chestnut:

<div align=center><img src="https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342787.png" alt="image-20220428185657036" style="zoom:80%;" /></div>

抽象父类 `Car`

```java
package template;

/**
 * 模板方法抽象类
 *
 * @author: posper
 * @version: v1.0
 * @date: 2022/04/28 18:43
 **/
public abstract class Car {
    // 发动
    public abstract void start();

    // 停止
    public abstract void stop();

    // 鸣笛
    public abstract void alarm();

    //
    public abstract void engineBoom();

    // 车总归要跑（放到子类中会造成代码冗余） --> 不如抽象到父类中
    // public abstract void run();

    /**
     * 模板方法
     */
    public void run() {
        start();
        engineBoom();
        alarm();
        stop();
    }
}
```

具体子类1 `Porsche`

```java
package template;

/**
 * @author: posper
 * @version: v1.0
 * @date: 2022/04/28 18:48
 **/
public class Porsche extends Car {
    @Override
    public void start() {
        System.out.println("保时捷启动");
    }

    @Override
    public void stop() {
        System.out.println("保时捷停止");
    }

    @Override
    public void alarm() {
        System.out.println("保时捷鸣笛");
    }

    @Override
    public void engineBoom() {
        System.out.println("保时捷发动引擎");
    }
}
```

具体子类2 `Ferrari`

```java
package template;

/**
 * @author: posper
 * @version: v1.0
 * @date: 2022/04/28 18:48
 **/
public class Ferrari extends Car {
    @Override
    public void start() {
        System.out.println("法拉利启动");
    }

    @Override
    public void stop() {
        System.out.println("法拉利停止");
    }

    @Override
    public void alarm() {
        System.out.println("法拉利鸣笛");
    }

    @Override
    public void engineBoom() {
        System.out.println("法拉利发动引擎");
    }
}
```

测试类

```java
package template;

/**
 * 测试“模板方法模式”
 *
 * @author: posper
 * @version: v1.0
 * @date: 2022/04/28 18:51
 **/
public class TemplatePatternTest {
    public static void main(String[] args) {
        Car porsche = new Porsche();
        Car ferrari = new Ferrari();
        porsche.run();
        ferrari.run();
    }
}
```

运行结果

<div align=center><img src="https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342792.png" alt="image-20220428190913067" style="zoom:80%;" /></div>

### JDK 源码解析：`AQS` :book:

> `JUC` 包中的 `AQS` 便大量使用了 “模板方法模式”

例如，`AQS` 中 `acquire` 方法便是一个**模板方法**，而 `tryAcquire` 方法由 `AQS` 子类根据自己特点做不同实现

```java
// AQS
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

以 `AQS` 的孙子类，即 `ReentrantLock` 的内部类 `NonfairSync` 为例，其 `tryAcquire` 方法实现如下：

```java
// ReentrantLock 内部类 NonfairSync
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

最终，通过 `acquire` （模板方法）调用的是子类 `NonfairSync` 中重写 `tryAcquire` 方法（非公平获取锁）

### 模板方法模式特点

==优点==

- **提高代码复用性**

  将相同部分的代码放在抽象的父类中

- **提高了拓展性**

  将不同的代码放入不同的子类中，通过对子类的扩展增加新的行为

- **实现了反向控制**

  通过一个父类调用其子类的操作，通过对子类的扩展增加新的行为，实现了反向控制 & 符合“开闭原则”

==缺点==

- 引入了抽象类，每一个不同的实现都需要一个子类来实现，导致类的个数增加，从而增加了系统实现的复杂度。

==适用场景==

- 一次性实现一个算法的不变的部分，并将可变的行为留给子类来实现；
- 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复；
- 控制子类的扩展。

## 迭代器模式

> 参考：《大话数据结构》ch20、[割韭韭-迭代器模式](https://blog.csdn.net/zhengzhb/article/details/7610745?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165123677916781435415088%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165123677916781435415088&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-7610745.142^v9^pc_search_result_control_group,157^v4^control&utm_term=%E8%BF%AD%E4%BB%A3%E5%99%A8%E6%A8%A1%E5%BC%8F&spm=1018.2226.3001.4187)

### 概述

**定义**

- 迭代器模式（Iterator)，提供一种方法**顺序访问一个聚合对象中各个元素，而又不暴露该对象的内部表示**

**作用**

- **将数据结构和操作数据结构的算法分离开**
  - 在迭代器模式之前，数据结构和操作数据结构的算法是分离不开的，一定的数据结构必然有对应的算法，就算是相同的算法，如果数据结构不一样，也不能通用；
  - 自从有了迭代器模式数据结构和算法算是彻底分离开，两者可以独立的发展


**迭代器模式结构图**

- **抽象容器**：一般是一个接口，提供一个 `iterator()` 方法
  - 例如，Java 中的 `Collection` 接口、`Map` 接口
- **具体容器**：就是抽象容器的具体实现类
  - 比如， `List` 接口的实现 `ArrayList`、`Map` 接口的实现 `HashMap`
- **抽象迭代器**：定义遍历元素所需要的方法（*常见的为如下 4 个*）
  - `first()` 方法：取得第一个元素
  - `next()` 方法：取得下一个元素
  - `isDone()` 方法（或叫 `hasNext()`）：判断是否遍历结束的方法
  - `remove()` 方法：移出当前对象
- **迭代器实现类**：实现迭代器接口中定义的方法，完成集合的迭代。

​	![image-20220429212339861](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342790.png)

### JDK 源码解析：`HashMap` :book:

![image-20220429211636966](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342788.png)

1. 抽象容器：`Map` 接口

   ```java
   public interface Map<K,V> {
       Set<Map.Entry<K, V>> entrySet();
       Set<K> keySet();
       Collection<V> values();

       // 省略其他...
   }
   ```

2. 具体容器类：`HashMap` 类

   > 这里仅以 `keySet()` 为例，`values()`、`entrySet()` 同理

   ```java
   public class HashMap<K,V> extends AbstractMap<K,V>
       implements Map<K,V>, Cloneable, Serializable {
       public Set<K> keySet() {
           Set<K> ks = keySet;
           if (ks == null) {
               ks = new KeySet();
               keySet = ks;
           }
           return ks;
       }

       // 省略其他...
   }
   ```

3. 迭代器接口：`Iterator`

   ```java
   public interface Iterator<E> {
       boolean hasNext();

       E next();

       // default方法
       default void remove() {
           throw new UnsupportedOperationException("remove");
       }

       default void forEachRemaining(Consumer<? super E> action) {
           Objects.requireNonNull(action);
           while (hasNext())
               action.accept(next());
       }
   }
   ```

4. 具体迭代器：`HashMap.KeyIterator`

   > 这里仅以 `HashMap.KeyIterator` 为例，`HashMap.ValueIterator`、`HashMap.EntryIterator` 同理

   ```java
   public class HashMap<K,V> extends AbstractMap<K,V>
       implements Map<K,V>, Cloneable, Serializable {
       final class KeySet extends AbstractSet<K> {
           public final Iterator<K> iterator()     {
               return new KeyIterator();
           }

           // 省略其他...
       }

       final class KeyIterator extends HashIterator implements Iterator<K> {
           public final K next() {
               return nextNode().key;
           }
       }

       abstract class HashIterator {
           Node<K,V> next;        // next entry to return
           Node<K,V> current;     // current entry
           int expectedModCount;  // for fast-fail
           int index;             // current slot

           HashIterator() {
               expectedModCount = modCount;
               Node<K,V>[] t = table;
               current = next = null;
               index = 0;
               if (t != null && size > 0) { // advance to first entry
                   do {} while (index < t.length && (next = t[index++]) == null);
               }
           }

           public final boolean hasNext() {
               return next != null;
           }

           final Node<K,V> nextNode() {
               Node<K,V>[] t;
               Node<K,V> e = next;
               if (modCount != expectedModCount)
                   throw new ConcurrentModificationException();
               if (e == null)
                   throw new NoSuchElementException();
               if ((next = (current = e).next) == null && (t = table) != null) {
                   do {} while (index < t.length && (next = t[index++]) == null);
               }
               return e;
           }

           public final void remove() {
               Node<K,V> p = current;
               if (p == null)
                   throw new IllegalStateException();
               if (modCount != expectedModCount)
                   throw new ConcurrentModificationException();
               current = null;
               K key = p.key;
               removeNode(hash(key), key, null, false, false);
               expectedModCount = modCount;
           }
       }

       // 省略其他...
   }
   ```

### 迭代器模式特点

==优点==

- **简化了遍历方式**

  - 对于 `HashMap` 来说，用户遍历起来就比较麻烦。而引入了迭代器方法后，用户用起来就简单的多了。

- **可以提供多种遍历方式**

  - 比如，对于 `LinkedList`，它提供了正序遍历，倒序遍历两种迭代器（可以由 `LinkedList` 的 `listIterator` 方法获取迭代器对象 `ListIterator`）

<div align=center><img src="https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/202205021342789.png" alt="image-20220429214805456" style="zoom:80%;" /></div>

- **封装性良好**：用户只需要得到迭代器就可以遍历，而对于遍历算法则不用去关心

==缺点==

- 对于比较简单的遍历（像数组或者有序列表），使用迭代器方式遍历较为繁琐（像 `ArrayLis`t，我们宁可愿意使用 for 循环和 get 方法来遍历集合）

==适用场景==：迭代器模式大多与集合（容器）一起使用

## 代理模式

**概述**

- 代理类与委托类有同样的接口，**代理类**主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在**关联关系**，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是**通过调用委托类的对象的相关方法，来提供特定的服务**。简单的说就是，我们在访问实际对象时，是**通过代理对象来访问的**，代理模式就是在访问实际对象时引入一定程度的**间接性**，因为这种间接性，可以附加多种用途。

**代理模式结构图**

![代理模式.png](https://s2.loli.net/2022/07/19/WT8vBsCKQDOElow.png)

- `Subject`：提供统一接口，供 `RealSubject` 和 `Proxy` 去实现；
- `RealSubject`：委托类，即实际中进行业务逻辑处理的类；
- `Proxy`：代理类，通过调用 `RealSubject` 来完成具体的业务逻辑，并可以在其基础上增加额外的扩招功能（如，拦截、参数校验等）

> 代理模式主要可以分为 静态代理 和 动态代理。

- 静态代理：由程序员创建或特定工具自动生成源代码，也就是**在编译时就已经将接口、被代理类、代理类等确定下来**。在程序**运行之前，代理类的 `.class` 文件就已经生成**。
- 动态代理：动态代理不用手工编写代理类，**代理类在程序运行时创建**。动态代理可以由 ”JDK 动态代理“ 和 “CGLIB动态代理” 两种实现方式。

**三种代理方式之间对比**

| 代理方式      | 实现                                                         | 优点                                                         | 缺点                                                         | 特点                                                       |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- |
| JDK静态代理   | 代理类与委托类实现同一接口，并且在代理类中需要硬编码接口     | 实现简单，容易理解                                           | 代理类需要硬编码接口，在实际应用中可能会导致重复编码，浪费存储空间并且效率很低 | /                                                          |
| JDK动态代理   | 代理类与委托类实现同一接口，主要是通过代理类实现 `InvocationHandler` 并重写`invoke `方法来进行动态代理的，在 `invoke` 方法中将对方法进行增强处理 | 不需要硬编码接口，代码复用率高                               | **只能**够代理实现了**接口**的委托类                         | 底层使用**反射**机制进行方法的调用                         |
| CGLIB动态代理 | 代理类将委托类作为自己的父类并为其中的非 final 委托方法创建两个方法，一个是与委托方法签名相同的方法，它在方法中会通过`super`调用委托方法；另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了`MethodInterceptor`接口的对象，若存在则将调用 `intercept` 方法对委托方法进行代理 | 可以在运行时对**类或者是接口**进行增强操作，且委托类无需实现接口 | **不能对`final`类以及 final 方法进行代理**                   | 底层将方法全部存入一个数组中，通过数组索引直接进行方法调用 |



# <span id='Spring'> Spring </span>

## BeanFactory 和 ApplicationContext 有什么区别？

> Spring 提供了两种 IoC 容器：BeanFactory 和 ApplicationContext

`BeanFactory`

- 默认使用懒加载（使用到时，才对受管对象进行初始化以及依赖注入操作）
- 优点：应用启动的时候占用资源很少，对资源要求较高的应用，比较有优势；
- 缺点：运行速度会相对来说慢一些；而且有可能会出现空指针异常的错误。

`ApplicationContext`

- 使用即时加载（所有的 Bean 在启动的时候都进行了加载）
- `ApplicationContext` 在 `BeanFactory ` 上进行扩展（间接继承），提供了其他高级特性（比如，时间发布、国家化信息支持等）
- 优点：使用即时加载，系统运行速度快；在系统启动的时候，可以发现系统中的配置问题。
- 缺点：把费时的操作放到系统启动中完成，所有的对象都可以预加载，启动时间较长，内存占用较大

## Spring IoC 容器中 Bean 的生命周期了解吗？



# <span id='Mybatis'> Mybatis </span>

### `#{}` 和 `${}` 的区别?

> 参考：[笑笑师弟](https://blog.csdn.net/qian_qian_123/article/details/92844194?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165314185016782248541337%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165314185016782248541337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-1-92844194-null-null.142^v10^pc_search_result_control_group,157^v4^control&utm_term=%24%7B%7D+%E5%92%8C+%23%7B%7D+%E5%8C%BA%E5%88%AB&spm=1018.2226.3001.4187)、[库森]()

- `#{}` 是**占位符**，会进行**预编译处理**；`${}` 是**拼接符**，**字符串替换**，**没有预编译处理**。

  - SQL 注入是发生在编译的过程中，因为恶意注入了某些特殊字符，最后被编译成了恶意的执行操作。而预编译机制则可以很好的防止 SQL注入攻击。
- Mybatis 在处理 `${param}` 时，**`param` 是原值传入**，相当于 JDBC 中的 **`Statement` 编译**；Mybatis 在处理 `#{param}` 时，`param` 传入参数是**以字符串传入**，会将SQL中的 `#` 替换为 `?` 号，**调 `PreparedStatement`** 的 `setXXX()` 方法按序给 sql 的 `?` 号占位符设置参数值。

  > `PreparedStatement` 不允许在插入参数时改变 SQL 语句的逻辑结构（可以防止 SQL 注入）
- 变量替换后，`#{}`对应的变量自动加上单引号 `''`；变量替换后，`${}` 对应的变量不会加上单引号 `''`
- `#{}` 可以有效的 **防止SQL注入**，提高系统安全性；`${}`**不能防止SQL注入**
- `#{}` 的变量替换是在 DBMS 中；`${}` 的变量替换是在 DBMS 外



# <span id='Algorithm'> 算法 </span>
## 1.排序算法
* [排序算法](https://t.zsxq.com/byvvbUn)
## 2.动态规划
### 2.1 背包问题
* 背包问题总结
![背包问题](https://user-images.githubusercontent.com/50728712/174442524-f9e6996c-163b-4ea5-8c6d-38a03ef6af6a.png)
### 2.2 股票问题
* 股票问题总结
![股票问题总结篇 (1)](https://user-images.githubusercontent.com/50728712/175819959-baafd627-b342-487c-979c-994500c2b523.png)



# <span id='IQ'>  智力题 </span>

### 100 层楼，2 个鸡蛋问题





# <span id='AdvancedpProblem'> 进阶问题 </span>

### 海量数据如何求 Top K ？




# <span id='tmpProblem'> 面试突击问题 </span>



