# Java

## 1.反射

### 1.1.反射的概念

反射(Reflection)是 Java 程序开发语言的特征之一，它允许==运行中==的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。

通过反射机制，可以在运行时访问 Java 对象的属性，方法，构造方法等。

### 1.2.反射的应用场景

反射的主要应用场景有：

- 开发通用框架 - 反射最重要的用途就是开发各种通用框架。很多框架（比如 Spring）都是配置化的（比如通过 XML 文件配置 JavaBean、Filter 等），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。
- 动态代理 - 在切面编程（AOP）中，需要拦截特定的方法，通常，会选择动态代理方式。这时，就需要反射技术来实现了。
- 注解 - 注解本身仅仅是起到标记作用，它需要利用反射机制，根据注解标记去调用注解解释器，执行行为。如果没有反射机制，注解并不比注释更有用。
- 可扩展性功能 - 应用程序可以通过使用完全限定名称创建可扩展性对象实例来使用外部的用户定义类。

### 1.3.反射的缺点

- 性能开销 - 由于反射涉及动态解析的类型，因此无法执行某些 Java 虚拟机优化。因此，反射操作的性能要比非反射操作的性能要差，应该在性能敏感的应用程序中频繁调用的代码段中避免。
- 破坏封装性 - 反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题。
- 内部曝光 - 由于反射允许代码执行在非反射代码中非法的操作，例如访问私有字段和方法，所以反射的使用可能会导致意想不到的副作用，这可能会导致代码功能失常并可能破坏可移植性。反射代码打破了抽象，因此可能会随着平台的升级而改变行为。

### 1.4.反射机制

#### 1.4.1.类加载过程

类加载的完整过程如下：

1. 在编译时，Java 编译器编译好 .java 文件之后，在磁盘中产生 .class 文件。.class 文件是二进制文件，内容是只有 JVM 能够识别的机器码。
2. JVM 中的类加载器读取字节码文件，取出二进制数据，加载到内存中，解析.class 文件内的信息。类加载器会根据类的全限定名来获取此类的二进制字节流；然后，将字节流所代表的静态存储结构转化为方法区的运行时数据结构；接着，在内存中生成代表这个类的 java.lang.Class 对象。
3. 加载结束后，JVM 开始进行连接阶段（包含验证、准备、初始化）。经过这一系列操作，类的变量会被初始化。

#### 1.4.2.Class 对象

要想使用反射，首先需要获得待操作的类所对应的 Class 对象。**Java 中，无论生成某个类的多少个对象，这些对象都会对应于同一个 Class 对象。这个 Class 对象是由 JVM 生成的，通过它能够获悉整个类的结构。** 所以，java.lang.Class 可以视为所有反射 API 的入口点。

反射的本质就是：在运行时，把 Java 类中的各种成分映射成一个个的 Java 对象。

举例来说，假如定义了以下代码：

```java
User user = new User();
```

步骤说明：

1. JVM 加载方法的时候，遇到 new User()，JVM 会根据 User 的全限定名去加载 User.class 。
2. JVM 会去本地磁盘查找 User.class 文件并加载 JVM 内存中。
3. JVM 通过调用类加载器自动创建这个类对应的 Class 对象，并且存储在 JVM 的方法区。注意：一个类有且只有一个 Class 对象。

#### 1.4.3.使用反射

**java.lang.reflect 包:**

Java 中的 java.lang.reflect 包提供了反射功能。java.lang.reflect 包中的类都没有 public 构造方法。

java.lang.reflect 包的核心接口和类如下：

- Member 接口 - 反映关于单个成员(字段或方法)或构造函数的标识信息。
- Field 类 - 提供一个类的域的信息以及访问类的域的接口。
- Method 类 - 提供一个类的方法的信息以及访问类的方法的接口。
- Constructor 类 - 提供一个类的构造函数的信息以及访问类的构造函数的接口。
- Array 类 - 该类提供动态地生成和访问 JAVA 数组的方法。
- Modifier 类 - 提供了 static 方法和常量，对类和成员访问修饰符进行解码。
- Proxy 类 - 提供动态地生成代理类和类实例的静态方法。

**获得 Class 对象:**

获得 Class 的三种方法：

**（1）使用 Class 类的 forName 静态方法:**

使用类的完全限定名来反射对象的类。常见的应用场景为：在 JDBC 开发中常用此方法加载数据库驱动。

```java

package io.github.dunwu.javacore.reflect;

public class ReflectClassDemo01 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class c1 = Class.forName("io.github.dunwu.javacore.reflect.ReflectClassDemo01");
        System.out.println(c1.getCanonicalName());
        Class c2 = Class.forName("[D");
        System.out.println(c2.getCanonicalName());
        Class c3 = Class.forName("[[Ljava.lang.String;");
        System.out.println(c3.getCanonicalName());
    }
}
//Output:
//io.github.dunwu.javacore.reflect.ReflectClassDemo01
//double[]
//java.lang.String[][]
```

**（2）直接获取某一个对象的 class:**

```java

public class ReflectClassDemo02 {
    public static void main(String[] args) {
        Boolean b;
        // Class c = b.getClass(); // 编译错误
        Class c1 = Boolean.class;
        System.out.println(c1.getCanonicalName());
        Class c2 = java.io.PrintStream.class;
        System.out.println(c2.getCanonicalName());
        Class c3 = int[][][].class;
        System.out.println(c3.getCanonicalName());
}
}
//Output:
//boolean
//java.io.PrintStream
//int[][][]

```

**（3）调用 Object 的 getClass 方法:**

Object 类中有 getClass 方法，因为所有类都继承 Object 类。从而调用 Object 类来获取

```java

package io.github.dunwu.javacore.reflect;
import java.util.HashSet;
import java.util.Set;
public class ReflectClassDemo03 {
    enum E {
        A, B
    }
    public static void main(String[] args) {
        Class c = "foo".getClass();
        System.out.println(c.getCanonicalName());
        Class c2 = ReflectClassDemo03.E.A.getClass();
        System.out.println(c2.getCanonicalName());
        byte[] bytes = new byte[1024];
        Class c3 = bytes.getClass();
        System.out.println(c3.getCanonicalName());
        Set<String> set = new HashSet<>();
        Class c4 = set.getClass();
        System.out.println(c4.getCanonicalName());
    }
}
//Output:
//java.lang.String
//io.github.dunwu.javacore.reflect.ReflectClassDemo.E
//byte[]
//java.util.HashSet

```

**判断是否为某个类的实例:**

判断是否为某个类的实例有两种方式：

- 用 instanceof 关键字
- 用 Class 对象的 isInstance 方法（它是一个 Native 方法）

```java

public class InstanceofDemo {
    public static void main(String[] args) {
        ArrayList arrayList = new ArrayList();
        if (arrayList instanceof List) {
            System.out.println("ArrayList is List");
        }
        if (List.class.isInstance(arrayList)) {
            System.out.println("ArrayList is List");
        }
    }
}
//Output:
//ArrayList is List
//ArrayList is List

```

### 1.5.JVM是如何构建一个实例的

- 内存：即JVM内存，栈、堆、方法区啥的都是JVM内存，只是人为划分
- .class文件：就是所谓的字节码文件，这里称.class文件，直观些

```java
//假设main方法中有以下代码：

Person p = new Person();

//很多初学者会以为整个创建对象的过程是下面这样的

javac Person.java
java Person
```

![加载](https://pic1.zhimg.com/80/v2-9cd31ab516bd967e1b8e68736931f8ba_720w.jpg)

稍微细致一点的过程可以是下面这样的

![稍微细致一点的过程可以是下面这样的](https://pic3.zhimg.com/80/v2-eddc430b991c58039dfc79dd6f3139cc_720w.jpg)

通过new创建实例和反射创建实例，都绕不开Class对象。

#### 1.5.1.class文件

比如我现在写一个类

![类](https://pic2.zhimg.com/80/v2-38dcb089ec823b2f217c13cba2b2ba85_720w.jpg)

用vim命令打开.class文件，以16进制显示就是下面这副鬼样子：

![vim类](https://pica.zhimg.com/80/v2-4bd1cd9dd96d7cffff41f63e95ec1cb8_720w.jpg)

在计算机中，任何东西底层保存的形式都是0101代码。

.java源码是给人类读的，而.class字节码是给计算机读的。根据不同的解读规则，可以产生不同的意思。就好比“这周日你有空吗”，合适的断句很重要。

同样的，JVM对.class文件也有一套自己的读取规则，不需要我们操心。总之，0101代码在它眼里的样子，和我们眼中的英文源码是一样的。

![说明](https://pic1.zhimg.com/80/v2-2a3a29ee51a52bb274112872e97dd2b1_720w.jpg)

#### 1.5.2类加载器

在最开始复习对象创建过程时，我们了解到.class文件是由类加载器加载的。但是核心方法只有loadClass()，告诉它需要加载的类名，它会帮你加载：

```java

protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查是否已经加载该类
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 如果尚未加载，则遵循父优先的等级加载机制（所谓双亲委派机制）
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // 模板方法模式：如果还是没有加载成功，调用findClass()
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    // 子类应该重写该方法
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
```

加载.class文件大致可以分为3个步骤：

1. 检查是否已经加载，有就直接返回，避免重复加载
2. 当前缓存中确实没有该类，那么遵循父优先加载机制，加载.class文件
3. 上面两步都失败了，调用findClass()方法加载

需要注意的是，ClassLoader类本身是抽象类，而抽象类是无法通过new创建对象的。所以它的findClass()方法写的很随意，直接抛了异常，反正你无法通过ClassLoader对象调用。也就是说，父类ClassLoader中的findClass()方法根本不会去加载.class文件。

正确的做法是，子类重写覆盖findClass()，在里面写自定义的加载逻辑。比如：

```java

@Override
public Class<?> findClass(String name) throws ClassNotFoundException {
    try {
        /*自己另外写一个getClassData()
                通过IO流从指定位置读取xxx.class文件得到字节数组*/
        byte[] datas = getClassData(name);
        if(datas == null) {
            throw new ClassNotFoundException("类没有找到：" + name);
        }
        //调用类加载器本身的defineClass()方法，由字节码得到Class对象
        return defineClass(name, datas, 0, datas.length);
    } catch (IOException e) {
        e.printStackTrace();
        throw new ClassNotFoundException("类找不到：" + name);
    }
}

```

defineClass()是ClassLoader定义的方法，目的是根据.class文件的字节数组byte[] b造出一个对应的Class对象。我们无法得知具体是如何实现的，因为最终它会调用一个native方法：

![native](https://pic2.zhimg.com/80/v2-87f55d018c506d90c1108dc63788e0b8_720w.jpg)

反正，目前我们关于类加载只需知道以下信息：

![加载信息](https://pic4.zhimg.com/80/v2-b9d39568c0e3f87a5df6a0cbfe753cda_720w.jpg)

#### 1.5.3.Class类

现在，.class文件被类加载器加载到内存中，并且JVM根据其字节数组创建了对应的Class对象。所以，我们来研究一下Class对象。

Class对象是Class类的实例，我们将在这一小节一步步分析Class类的结构。

但是，在看源码之前，我想问问聪明的各位，如果你是JDK源码设计者，你会如何设计Class类？

假设现在有个BaseDto类

![basedto](https://pica.zhimg.com/80/v2-5f8d9252d2d8d65573b342a344fc5697_720w.jpg)

上面类至少包括以下信息（按顺序）：

- 权限修饰符
- 类名
- 参数化类型（泛型信息）
- 接口
- 注解
- 字段（重点）
- 构造器（重点）
- 方法（重点）

最终这些信息在.class文件中都会以0101表示：

![.class](https://pica.zhimg.com/80/v2-3a2c3b0a444bffd6938017106fd5597d_720w.jpg)

整个.class文件最终都成为字节数组byte[] b，里面的构造器、方法等各个“组件”，其实也是字节。

所以，我猜Class类的字段至少是这样的：

![zheyang](https://pic4.zhimg.com/80/v2-de522e669207c17994cf7394a9f23670_720w.jpg)

好了，看一下源码是不是如我所料：

**字段、方法、构造器对象:**

![source](https://pica.zhimg.com/80/v2-fe965a307866fddee50862bbe8b3eabb_720w.jpg)

**注解数据:**

![注解数据](https://pic3.zhimg.com/80/v2-40ce6b148b43a464df1a893fa5dc6978_720w.jpg)

**泛型信息:**

![泛型信息](https://pic3.zhimg.com/80/v2-da2f79b3bb318554d16b5101e1f59427_720w.jpg)

而且，针对字段、方法、构造器，因为信息量太大了，JDK还单独写了三个类，比如Method类：

![method](https://pica.zhimg.com/80/v2-d31b7dedef1bd4eb470305751b287bb7_720w.jpg)

也就是说，Class类准备了很多字段用来表示一个.class文件的信息，对于字段、方法、构造器等，为了更详细地描述这些重要信息，还写了三个类，每个类里面都有很详细的对应。

![说明信息](https://pic2.zhimg.com/80/v2-400bf816f48cba411c2f2ccd2707214c_720w.jpg)

也就是说，原本UserController类中所有信息，都被“解构”后保存在Class类、Method类等的字段中。

大概了解完Class类的字段后，我们看看Class类的方法。

构造器

![构造器](https://pic2.zhimg.com/80/v2-d4cb040cceebe0dab651c95a0c22c7f0_720w.jpg)

可以发现，Class类的构造器是私有的，我们无法手动new一个Class对象，只能由JVM创建。JVM在构造Class对象时，需要传入一个类加载器，然后才有我们上面分析的一连串加载、创建过程。

Class.forName()方法

![Class.forName()方法](https://pic2.zhimg.com/80/v2-db050e2d3b37109a399d29fd9b047753_720w.jpg)

反正还是类加载器去搞呗。

newInstance()

![newInstance()](https://pic1.zhimg.com/80/v2-597031fda156c011daae31152bcb1c29_720w.jpg)

也就是说，newInstance()底层就是调用无参构造对象的newInstance()。

所以，本质上Class对象要想创建实例，其实都是通过构造器对象。如果没有空参构造对象，就无法使用clazz.newInstance()，必须要获取其他有参的构造对象然后调用构造对象的newInstance()。

#### 1.5.4.反射API

没啥好说的，在日常开发中反射最终目的主要两个：

- 创建实例
- 反射调用方法

创建实例的难点在于，很多人不知道clazz.newInstance()底层还是调用Contructor对象的newInstance()。所以，要想调用clazz.newInstance()，必须保证编写类的时候有个无参构造。

反射调用方法的难点，有两个，初学者可能会不理解。

再此之前，先来理清楚Class、Field、Method、Constructor四个对象的关系：

![关系](https://pica.zhimg.com/80/v2-96e87c388c61b8de3a6c526a17128f63_720w.jpg)

Field、Method、Constructor对象内部有对字段、方法、构造器更详细的描述：

![描述](https://pica.zhimg.com/80/v2-57f4b3321a70f5713cec0ab399a51328_720w.jpg)

- 难点一：为什么根据Class对象获取Method时，需要传入方法名+参数的Class类型

![难点一](https://pic4.zhimg.com/80/v2-c7cc3cdbc4d43ad4d81196a6da5ee8b6_720w.jpg)

为什么要传name和ParameterType？

因为.class文件中有多个方法，比如

![比如](https://pic3.zhimg.com/80/v2-8102c423a36a8fe0a469eca4da3a2d4d_720w.jpg)

所以必须传入name，以方法名区分哪个方法，得到对应的Method。

那参数parameterTypes为什么要用Class类型，我想和调用方法时一样直接传变量名不行吗，比如userName， age。

答案是：我们无法根据变量名区分方法

```java

User getUser(String userName, int age);
User getUser(String mingzi, int nianling);

```

这不叫重载，这就是同一个方法。只能根据参数类型。

我知道，你还会问：变量名不行，那我能不能传String, int。

不好意思，这些都是基本类型和引用类型，类型不能用来传递。我们能传递的要么值，要么对象（引用）。而String.class, int.class是对象，且是Class对象。

实际上，调用Class对象的getMethod()方法时，内部会循环遍历所有Method，然后根据方法名和参数类型匹配唯一的Method返回。

循环遍历所有Method，根据name和parameterType匹配

![循环遍历所有Method，根据name和parameterType匹配](https://pic3.zhimg.com/80/v2-f292599938ee37bea75d6d6e3076358f_720w.jpg)

- 难点二：调用method.invoke(obj, args);时为什么要传入一个目标对象？

上面分析过，.class文件通过IO被加载到内存后，JDK创造了至少四个对象：Class、Field、Method、Constructor，这些对象其实都是0101010的抽象表示。

以Method对象为例，它到底是什么，怎么来的？我们上面已经分析过，Method对象有好多字段，比如name（方法名），returnType（返回值类型）等。也就是说我们在.java文件中写的方法，被“解构”以后存入了Method对象中。所以对象本身是一个方法的映射，一个方法对应一个Method对象。

对象的本质就是用来存储数据的。而方法作为一种行为描述，是所有对象共有的，不属于某个对象独有。比如现有两个Person实例

```java

Person p1 = new Person();
Person p2 = new Person();

```

对象 p1保存了"hst"和18，p2保存了"cxy"和20。但是不管是p1还是p2，都会有changeUser()，但是每个对象里面写一份太浪费。既然是共性行为，可以抽取出来，放在方法区共用。

但这又产生了一个棘手的问题，方法是共用的，JVM如何保证p1调用changeUser()时，changeUser()不会跑去把p2的数据改掉呢？

所以JVM设置了一种隐性机制，每次对象调用方法时，都会隐性传递当前调用该方法的对象参数，方法可以根据这个对象参数知道当前调用本方法的是哪个对象！

![jvm](https://pica.zhimg.com/80/v2-ee9b937fa3569e3a874e838475fbacd3_720w.jpg)

同样的，在反射调用方法时，本质还是希望方法处理数据，所以必须告诉它执行哪个对象的数据。

![data](https://pic4.zhimg.com/80/v2-76790b5762a6f1d3f5b2054eb5cbd2bb_720w.jpg)

所以，把Method理解为方法执行指令吧，它更像是一个方法执行器，必须告诉它要执行的对象（数据）。

当然，如果是invoke一个静态方法，不需要传入具体的对象。因为静态方法并不能处理对象中保存的数据。

## 2.多线程

### 2.1.多线程说明

如果对什么是线程、什么是进程仍存有疑惑，请先Google之，因为这两个概念不在本文的范围之内。

用多线程只有一个目的，那就是更好的利用cpu的资源，因为所有的多线程代码都可以用单线程来实现。说这个话其实只有一半对，因为反应“多角色”的程序代码，最起码每个角色要给他一个线程吧，否则连实际场景都无法模拟，当然也没法说能用单线程来实现：比如最常见的“生产者，消费者模型”。

很多人都对其中的一些概念不够明确，如同步、并发等等，让我们先建立一个数据字典，以免产生误会。

- 多线程：指的是这个程序（一个进程）运行时产生了不止一个线程
- 并行与并发：
  - 并行：多个cpu实例或者多台机器同时执行一段处理逻辑，是真正的同时。
  - 并发：通过cpu调度算法，让用户看上去同时执行，实际上从cpu操作层面不是真正的同时。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用TPS或者QPS来反应这个系统的处理能力。

![并发和并行](https://upload-images.jianshu.io/upload_images/1689841-f622a468b2694253.png?imageMogr2/auto-orient/strip%7CimageView2/2)

并发与并行

- 线程安全：经常用来描绘一段代码。指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果。这个时候使用多线程，我们只需要关注系统的内存，cpu是不是够用即可。反过来，线程不安全就意味着线程的调度顺序会影响最终结果，如不加事务的转账代码：

    ```java
    void transferMoney(User from, User to, float amount){
    to.setMoney(to.getBalance() + amount);
    from.setMoney(from.getBalance() - amount);
    }
    ```

- 同步：Java中的同步指的是通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。如上面的代码简单加入<font color=red>@synchronized</font>关键字。在保证结果准确的同时，提高性能，才是优秀的程序。线程安全的优先级高于性能。

分成几部分来总结涉及到多线程的内容：

1. 扎好马步：线程的状态
2. 内功心法：每个对象都有的方法（机制）
3. 太祖长拳：基本线程类
4. 九阴真经：高级多线程控制类

### 2.2.线程的状态

先来两张图：

![线程状态](https://upload-images.jianshu.io/upload_images/1689841-af3e5b75b44e972c.png?imageMogr2/auto-orient/strip%7CimageView2/2)

线程状态

![线程状态转换](https://upload-images.jianshu.io/upload_images/1689841-383f7101e6588094.png?imageMogr2/auto-orient/strip%7CimageView2/2)

线程状态转换

各种状态一目了然，值得一提的是"blocked"这个状态：
线程在Running的过程中可能会遇到阻塞(Blocked)情况

   1. 调用join()和sleep()方法，sleep()时间结束或被打断，join()中断,IO完成都会回到Runnable状态，等待JVM的调度。
   2. 调用wait()，使该线程处于等待池(wait blocked pool),直到notify()/notifyAll()，线程被唤醒被放到锁定池(lock blocked pool )，释放同步锁使线程回到可运行状态（Runnable）。
   3. 对Running状态的线程加同步锁(Synchronized)使其进入(lock blocked pool ),同步锁被释放进入可运行状态(Runnable)。

此外，在runnable状态的线程是处于被调度的线程，此时的调度顺序是不一定的。Thread类中的yield方法可以让一个running状态的线程转入runnable。

### 2.3.每个对象都有的方法（机制）

synchronized, wait, notify 是任何对象都具有的同步工具。让我们先来了解他们

![多线程关键字](https://upload-images.jianshu.io/upload_images/1689841-a8720771d68cb2ba.png?imageMogr2/auto-orient/strip%7CimageView2/2)

monitor

他们是应用于同步问题的人工线程调度工具。讲其本质，首先就要明确monitor的概念，Java中的每个对象都有一个监视器，来监测并发代码的重入。在非多线程编码时该监视器不发挥作用，反之如果在synchronized 范围内，监视器发挥作用。

wait/notify必须存在于synchronized块中。并且，这三个关键字针对的是同一个监视器（某对象的监视器）。这意味着wait之后，其他线程可以进入同步块执行。

当某代码并不持有监视器的使用权时（如图中5的状态，即脱离同步块）去wait或notify，会抛出java.lang.IllegalMonitorStateException。也包括在synchronized块中去调用另一个对象的wait/notify，因为不同对象的监视器不同，同样会抛出此异常。

再讲用法：

- synchronized单独使用：
  - 代码块：如下，在多线程环境下，synchronized块中的方法获取了lock实例的monitor，如果实例相同，那么只有一个线程能执行该块内容

    ```java
    public class Thread1 implements Runnable {
    Object lock;
    public void run() {  
        synchronized(lock){
            ..do something
        }
    }
    }
    ```

  - 直接用于方法： 相当于上面代码中用lock来锁定的效果，实际获取的是Thread1类的monitor。更进一步，如果修饰的是static方法，则锁定该类所有实例。

    ```java
    public class Thread1 implements Runnable {
        public synchronized void run() {  
            ..do something
        }
    }
    ```

- synchronized, wait, notify结合:典型场景生产者消费者问题

```java
/**
   * 生产者生产出来的产品交给店员
   */
  public synchronized void produce()
  {
      if(this.product >= MAX_PRODUCT)
      {
          try
          {
              wait();  
              System.out.println("产品已满,请稍候再生产");
          }
          catch(InterruptedException e)
          {
              e.printStackTrace();
          }
          return;
      }

      this.product++;
      System.out.println("生产者生产第" + this.product + "个产品.");
      notifyAll();   //通知等待区的消费者可以取出产品了
  }

  /**
   * 消费者从店员取产品
   */
  public synchronized void consume()
  {
      if(this.product <= MIN_PRODUCT)
      {
          try 
          {
              wait(); 
              System.out.println("缺货,稍候再取");
          } 
          catch (InterruptedException e) 
          {
              e.printStackTrace();
          }
          return;
      }

      System.out.println("消费者取走了第" + this.product + "个产品.");
      this.product--;
      notifyAll();   //通知等待去的生产者可以生产产品了
  }
```

volatile

多线程的内存模型：main memory（主存）、working memory（线程栈），在处理数据时，线程会把值从主存load到本地栈，完成操作后再save回去(volatile关键词的作用：每次针对该变量的操作都激发一次load and save)。

![volatile](https://upload-images.jianshu.io/upload_images/1689841-d4ab6cfda7042c67.png?imageMogr2/auto-orient/strip%7CimageView2/2)

volatile

针对多线程使用的变量如果不是volatile或者final修饰的，很有可能产生不可预知的结果（另一个线程修改了这个值，但是之后在某线程看到的是修改之前的值）。其实道理上讲同一实例的同一属性本身只有一个副本。但是多线程是会缓存值的，本质上，volatile就是不去缓存，直接取值。在线程安全的情况下加volatile会牺牲性能。

### 2.4.基本线程类

基本线程类指的是Thread类，Runnable接口，Callable接口
Thread 类实现了Runnable接口，启动一个线程的方法：

```java
　MyThread my = new MyThread();
　　my.start();
```

**Thread类相关方法：**

```java
//当前线程可转让cpu控制权，让别的就绪状态线程运行（切换）
public static Thread.yield()
//暂停一段时间
public static Thread.sleep()
//在一个线程中调用other.join(),将等待other执行完后才继续本线程。　　　　
public join()
//后两个函数皆可以被打断
public interrupte()
```

**关于中断：**

它并不像stop方法那样会中断一个正在运行的线程。线程会不时地检测中断标识位，以判断线程是否应该被中断（中断标识值是否为true）。终端只会影响到wait状态、sleep状态和join状态。被打断的线程会抛出InterruptedException。Thread.interrupted()检查当前线程是否发生中断，返回boolean synchronized在获锁的过程中是不能被中断的。

中断是一个状态！interrupt()方法只是将这个状态置为true而已。所以说正常运行的程序不去检测状态，就不会终止，而wait等阻塞方法会去检查并抛出异常。如果在正常运行的程序中添加while(!Thread.interrupted()) ，则同样可以在中断后离开代码体

**Thread类最佳实践：**

写的时候最好要设置线程名称 Thread.name，并设置线程组 ThreadGroup，目的是方便管理。在出现问题的时候，打印线程栈 (jstack -pid) 一眼就可以看出是哪个线程出的问题，这个线程是干什么的。

**如何获取线程中的异常：**

![异常](https://pic3.zhimg.com/v2-e0e6a844e9d28972eb5142a41d597fd6_b.jpg)

不能用try,catch来获取线程中的异常

**Runnable**
与Thread类似

**Callable**
future模式：并发模式的一种，可以有两种形式，即无阻塞和阻塞，分别是isDone和get。其中Future对象用来存放该线程的返回值以及状态

```java
ExecutorService e = Executors.newFixedThreadPool(3);
 //submit方法有多重参数版本，及支持callable也能够支持runnable接口类型.
Future future = e.submit(new myCallable());
future.isDone() //return true,false 无阻塞
future.get() // return 返回值，阻塞直到该线程运行结束
```

### 2.5.高级多线程控制类

Java1.5提供了一个非常高效实用的多线程包:java.util.concurrent, 提供了大量高级工具,可以帮助开发者编写高效、易维护、结构清晰的Java多线程程序。

**1.ThreadLocal类**
用处：保存线程的独立变量。对一个线程类（继承自Thread)
当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。常用于用户登录控制，如记录session信息。

实现：每个Thread都持有一个TreadLocalMap类型的变量（该类是一个轻量级的Map，功能与map一样，区别是桶里放的是entry而不是entry的链表。功能还是一个map。）以本身为key，以目标为value。
主要方法是get()和set(T a)，set之后在map里维护一个threadLocal -> a，get时将a返回。ThreadLocal是一个特殊的容器。

**2.原子类（AtomicInteger、AtomicBoolean……）:**

```java
如果使用atomic wrapper class如atomicInteger，或者使用自己保证原子的操作，则等同于synchronized
//返回值为boolean
AtomicInteger.compareAndSet(int expect,int update)
```

该方法可用于实现乐观锁，考虑文中最初提到的如下场景：a给b付款10元，a扣了10元，b要加10元。此时c给b2元，但是b的加十元代码约为：

```java
if(b.value.compareAndSet(old, value)){
   return ;
}else{
   //try again
   // if that fails, rollback and log
}
```

**AtomicReference**
对于AtomicReference 来讲，也许对象会出现，属性丢失的情况，即oldObject == current，但是oldObject.getPropertyA != current.getPropertyA。
这时候，AtomicStampedReference就派上用场了。这也是一个很常用的思路，即加上版本号

**3.Lock类:**

lock: 在java.util.concurrent包内。共有三个实现：

- ReentrantLock
- ReentrantReadWriteLock.ReadLock
- ReentrantReadWriteLock.WriteLock

主要目的是和synchronized一样， 两者都是为了解决同步问题，处理资源争端而产生的技术。功能类似但有一些区别。

区别如下：

1. lock更灵活，可以自由定义多把锁的枷锁解锁顺序（synchronized要按照先加的后解顺序）
2. 提供多种加锁方案，lock 阻塞式, trylock 无阻塞式, lockInterruptily 可打断式， 还有trylock的带超时时间版本。
3. 本质上和监视器锁（即synchronized是一样的）
4. 能力越大，责任越大，必须控制好加锁和解锁，否则会导致灾难。
5. 和Condition类的结合。
6. 性能更高，对比如下图：
![synchronized和Lock性能对比](https://pic2.zhimg.com/v2-8482266d122da43bd5006fc4dd670d31_b.jpg)

**ReentrantLock**
可重入的意义在于持有锁的线程可以继续持有，并且要释放对等的次数后才真正释放该锁。
使用方法是：

**1.先new一个实例:**

```java
static ReentrantLock r=new ReentrantLock();
```

**2.加锁:**

```java
r.lock()或r.lockInterruptibly();
```

此处也是个不同，后者可被打断。当a线程lock后，b线程阻塞，此时如果是lockInterruptibly，那么在调用b.interrupt()之后，b线程退出阻塞，并放弃对资源的争抢，进入catch块。（如果使用后者，必须throw interruptable exception 或catch）

**3.释放锁:**

```java
r.unlock()
```

必须做！何为必须做呢，要放在finally里面。以防止异常跳出了正常流程，导致灾难。这里补充一个小知识点，finally是可以信任的：经过测试，哪怕是发生了OutofMemoryError，finally块中的语句执行也能够得到保证。

**ReentrantReadWriteLock**
可重入读写锁（读写锁的一个实现）

```java
　ReentrantReadWriteLock lock = new ReentrantReadWriteLock()
　　ReadLock r = lock.readLock();
　　WriteLock w = lock.writeLock();

两者都有lock,unlock方法。写写，写读互斥；读读不互斥。可以实现并发读的高效线程安全代码
```

**4.容器类:**

这里就讨论比较常用的两个：

```java
BlockingQueue
ConcurrentHashMap
```

**BlockingQueue**
阻塞队列。该类是java.util.concurrent包下的重要类，通过对Queue的学习可以得知，这个queue是单向队列，可以在队列头添加元素和在队尾删除或取出元素。类似于一个管　　道，特别适用于先进先出策略的一些应用场景。普通的queue接口主要实现有PriorityQueue（优先队列），有兴趣可以研究

BlockingQueue在队列的基础上添加了多线程协作的功能：

![BlockingQueu](https://upload-images.jianshu.io/upload_images/1689841-c3b87b06b8d86db8.png?imageMogr2/auto-orient/strip%7CimageView2/2)

BlockingQueue

除了传统的queue功能（表格左边的两列）之外，还提供了阻塞接口put和take，带超时功能的阻塞接口offer和poll。put会在队列满的时候阻塞，直到有空间时被唤醒；take在队　列空的时候阻塞，直到有东西拿的时候才被唤醒。用于生产者-消费者模型尤其好用，堪称神器。

常见的阻塞队列有：

```java
ArrayListBlockingQueue
LinkedListBlockingQueue
DelayQueue
SynchronousQueue
```

**ConcurrentHashMap**
高效的线程安全哈希map。请对比hashTable , concurrentHashMap, HashMap

**5.管理类:**

管理类的概念比较泛，用于管理线程，本身不是多线程的，但提供了一些机制来利用上述的工具做一些封装。

了解到的值得一提的管理类：ThreadPoolExecutor和 JMX框架下的系统级管理类 ThreadMXBean

**ThreadPoolExecutor**
如果不了解这个类，应该了解前面提到的ExecutorService，开一个自己的线程池非常方便：

```java
    ExecutorService e = Executors.newCachedThreadPool();
    ExecutorService e = Executors.newSingleThreadExecutor();
    ExecutorService e = Executors.newFixedThreadPool(3);
    // 第一种是可变大小线程池，按照任务数来分配线程，
    // 第二种是单线程池，相当于FixedThreadPool(1)
    // 第三种是固定大小线程池。
    // 然后运行
    e.execute(new MyRunnableImpl());
```

该类内部是通过ThreadPoolExecutor实现的，掌握该类有助于理解线程池的管理，本质上，他们都是ThreadPoolExecutor类的各种实现版本。请参见javadoc：

![ThreadPoolExecutor参数解释](https://upload-images.jianshu.io/upload_images/1689841-1a8f8a70ba646843.png?imageMogr2/auto-orient/strip%7CimageView2/2)

ThreadPoolExecutor参数解释

翻译一下：

```java
corePoolSize:池内线程初始值与最小值，就算是空闲状态，也会保持该数量线程。
maximumPoolSize:线程最大值，线程的增长始终不会超过该值。
keepAliveTime：当池内线程数高于corePoolSize时，经过多少时间多余的空闲线程才会被回收。回收前处于wait状态
unit：
时间单位，可以使用TimeUnit的实例，如TimeUnit.MILLISECONDS　
workQueue:待入任务（Runnable）的等待场所，该参数主要影响调度策略，如公平与否，是否产生饿死(starving)
threadFactory:线程工厂类，有默认实现，如果有自定义的需要则需要自己实现ThreadFactory接口并作为参数传入。
```

## 3.JVM

## 4.分布式锁

## 5.ORM实现
