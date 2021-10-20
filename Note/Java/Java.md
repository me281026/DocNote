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

### 3.1.JVM的基础概念

JVM的中文名称叫Java虚拟机，它是由软件技术模拟出计算机运行的一个虚拟的计算机。

JVM也充当着一个翻译官的角色，我们编写出的Java程序，是不能够被操作系统所直接识别的，这时候JVM的作用就体现出来了，它负责把我们的程序翻译给系统“听”，告诉它我们的程序需要做什么操作。

我们都知道Java的程序需要经过编译后，产生.Class文件，JVM才能识别并运行它，JVM针对每个操作系统开发其对应的解释器，所以只要其操作系统有对应版本的JVM，那么这份Java编译后的代码就能够运行起来，这就是Java能一次编译，到处运行的原因。

### 3.2.JVM的生命周期

JVM在Java程序开始执行的时候，它才运行，程序结束的时它就停止。

一个Java程序会开启一个JVM进程，如果一台机器上运行三个程序，那么就会有三个运行中的JVM进程。

JVM中的线程分为两种：守护线程和普通线程守护线程是JVM自己使用的线程，比如垃圾回收（GC）就是一个守护线程。

普通线程一般是Java程序的线程，只要JVM中有普通线程在执行，那么JVM就不会停止。权限足够的话，可以调用exit()方法终止程序。

### 3.3.JVM的结构体系

![JVM的结构体系](https://pic1.zhimg.com/v2-bf438382f881edb81d49a2ddb8888d68_b.png)

### 3.4.JVM的启动过程

#### 3.4.1.JVM的装入环境和配置

在学习这个之前，我们需要了解一件事情，就是JDK和JRE的区别。

JDK是面向开发人员使用的SDK，它提供了Java的开发环境和运行环境，JDK中包含了JRE。JRE是Java的运行环境，是面向所有Java程序的使用者，包括开发者。

JRE = 运行环境 = JVM。

如果安装了JDK，会发现电脑中有两套JRE，一套位于/Java/jre.../下，一套位于/Java/jdk.../jre下。

那么问题来了，一台机器上有两套以上JRE，谁来决定运行那一套呢？这个任务就落到java.exe身上，java.exe的任务就是找到合适的JRE来运行java程序。

java.exe按照以下的顺序来选择JRE：

1. 自己目录下有没有JRE
2. 父目录下有没有JRE
3. 查询注册表： HKEY_LOCAL_MACHINE\SOFTWARE\JavaSoft\Java Runtime Environment\"当前JRE版本号"\JavaHome

这几步的主要核心是为了找到JVM的绝对路径。

jvm.cfg的路径为：JRE路径\lib\"CPU架构"\jvm.fig

jvm.cfg的内容大致如下：

- client KNOWN
- server KNOWN
- hotspot ALIASED_TO -client
- classic WARN
- native ERROR
- green ERROR

KNOWN 表示存在 、IGNORE 表示不存在 、ALIASED_TO 表示给别的JVM去一个别名
WARN 表示不存在时找一个替代 、ERROR 表示不存在抛出异常

#### 3.4.2.装载JVM

通过第一步找到JVM的路径后，Java.exe通过LoadJavaVM来装入JVM文件。LoadLibrary装载JVM动态连接库，然后把JVM中的到处函数JNI_CreateJavaVM和JNI_GetDefaultJavaVMIntArgs 挂接到InvocationFunction 变量的CreateJavaVM和GetDafaultJavaVMInitArgs 函数指针变量上。JVM的装载工作完成。

#### 3.4.3.初始化JVM，获得本地调用接口

调用InvocationFunction -> CreateJavaVM也就是JVM中JNI_CreateJavaVM方法获得JNIEnv结构的实例。

#### 3.4.4.运行Java程序

JVM运行Java程序的方式有两种：jar包 与 Class运行jar 的时候，Java.exe调用GetMainClassName函数，该函数先获得JNIEnv实例然后调用JarFileJNIEnv类中getManifest()，从其返回的Manifest对象中取getAttrebutes("Main-Class")的值，即jar

包中文件：META-INF/MANIFEST.MF指定的Main-Class的主类名作为运行的主类。

之后main函数会调用Java.c中LoadClass方法装载该主类（使用JNIEnv实例的FindClass）。

运行Class的时候，main函数直接调用Java.c中的LoadClass方法装载该类。

**Class文件**
Class文件由Java编译器生成，我们创建的.Java文件在经过编译器后，会变成.Class的文件，这样才能被JVM所识别并运行。

**类加载子系统**
类加载子系统也可以称之为类加载器，JVM默认提供三个类加载器：

1. BootStrap ClassLoader ：称之为启动类加载器，是最顶层的类加载器，负责加载JDK中的核心类库，如 rt.jar、resources.jar、charsets.jar等。
2. Extension ClassLoader：称之为扩展类加载器，负责加载Java的扩展类库，默认加载$JAVA_HOME中jre/lib/*.jar 或 -Djava.ext.dirs指定目录下的jar包。
3. App ClassLoader：称之为系统类加载器，负责加载应用程序classpath目录下所有jar和class文件。

除了Java默认提供的三个ClassLoader（加载器）之外，我们还可以根据自身需要自定义ClassLoader，自定义ClassLoader必须继承java.lang.ClassLoader 类。除了BootStrap ClassLoader 之外的另外两个默认加载器都是继承自java.lang.ClassLoader 。BootStrap ClassLoader 不是一个普通的Java类，它底层由C++编写，已嵌入到了JVM的内核当中，当JVM启动后，BootStrap ClassLoader 也随之启动，负责加载完核心类库后，并构造Extension ClassLoader 和App ClassLoader 类加载器。

类加载器子系统不仅仅负责定位并加载类文件，它还严格按照以下步骤做了很多事情：

1. 加载：寻找并导入Class文件的二进制信息
2. 连接：进行验证、准备和解析
   1. 验证：确保导入类型的正确性
   2. 准备：为类型分配内存并初始化为默认值
   3. 解析：将字符引用解析为直接引用
3. 初始化：调用Java代码，初始化类变量为指定初始值

### 3.5.Java类加载机制

**概述:**

Class文件由类装载器装载后，在JVM中将形成一份描述Class结构的元信息对象，通过该元信息对象可以获知Class的结构信息：如构造函数，属性和方法等，Java允许用户借由这个Class相关的元信息对象间接调用Class对象的功能。

虚拟机把描述类的数据从class文件加载到内存，并对数据进行校验，转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

**什么是类的加载:**

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。

类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

#### 3.5.1.类的加载过程

JVM将类的加载分为3个步骤：

1. 装载（Load）

2. 链接（Link）

3. 初始化（Initialize）

其中 链接（Link）又分3个步骤，如下图所示：

![链接（Link）](https://pic4.zhimg.com/80/v2-5473646d79609214433ee7a66e594603_720w.png)

**装载：查找并加载类的二进制数据（查找和导入Class文件）:**

加载是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：

1. 通过一个类的全限定名来获取其定义的二进制字节流。

2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。

3. 在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口。

相对于类加载的其他阶段而言，加载阶段（准确地说，是加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载。

加载阶段完成后，虚拟机外部的 二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在Java堆中也创建一个java.lang.Class类的对象，这样便可以通过该对象访问方法区中的这些数据。

**链接（分3个步骤）:**

**1、验证**：确保被加载的类的正确性

验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作：

文件格式验证：验证字节流是否符合Class文件格式的规范；例如：是否以0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。

元数据验证：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了java.lang.Object之外。

字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。

符号引用验证：确保解析动作能正确执行。

验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

**2、准备**：为类的静态变量分配内存，并将其初始化为默认值

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

1、这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。

2、这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、null、false等），而不是被在Java代码中被显式地赋予的值。

假设一个类变量的定义为：public static int value = 3； 那么变量value在准备阶段过后的初始值为0，而不是3，因为这时候尚未开始执行任何Java方法，而把value赋值为3的putstatic指令是在程序编译后，存放于类构造器\<clinit>（）方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行。

**3、解析**：把类中的符号引用转换为直接引用

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符7类符号引用进行。符号引用就是一组符号来描述目标，可以是任何字面量。
直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

**初始化：对类的静态变量，静态代码块执行初始化操作:**

初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。在Java中对类变量进行初始值设定有两种方式：

①声明类变量是指定初始值。

②使用静态代码块为类变量指定初始值。

#### 3.5.2.类的初始化

类什么时候才被初始化：

1. 创建类的实例，也就是new一个对象

2. 访问某个类或接口的静态变量，或者对该静态变量赋值

3. 调用类的静态方法

4. 反射（Class.forName("com.lyj.load")）

5. 初始化一个类的子类（会首先初始化子类的父类）

6. JVM启动时标明的启动类，即文件名和类名相同的那个类 只有这6中情况才会导致类的类的初始化。

类的初始化步骤 / JVM初始化步骤：

1. 如果这个类还没有被加载和链接，那先进行加载和链接

2. 假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一次），那就初始化直接的父类（不适用于接口）

3. 假如类中存在初始化语句（如static变量和static块），那就依次执行这些初始化语句。

#### 3.5.3.类的加载

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个这个类的Java.lang.Class对象，用来封装类在方法区类的对象。

![类的加载](https://pic4.zhimg.com/80/v2-50658534f3b3fcbab4b89af0a334619b_720w.jpg)

类的加载的最终产品是位于堆区中的Class对象。 Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。

加载类的方式有以下几种：

1. 从本地系统直接加载

2. 通过网络下载.class文件

3. 从zip，jar等归档文件中加载.class文件

4. 从专有数据库中提取.class文件

5. 将Java源文件动态编译为.class文件（服务器）

6. 命令行启动应用时候由JVM初始化加载

7. 通过Class.forName()方法动态加载

8. 通过ClassLoader.loadClass()方法动态加载

#### 3.5.4.加载器

JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：

![加载器](https://pic4.zhimg.com/80/v2-4b379b6a26ae8567dedae02295805cf3_720w.png)

1. Bootstrap ClassLoader 负责加载$JAVA_HOME中 jre/lib/rt.jar 里所有的class或Xbootclassoath选项指定的jar包。由C++实现，不是ClassLoader子类。

2. Extension ClassLoader 负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar 或 -Djava.ext.dirs指定目录下的jar包。

3. App ClassLoader 负责加载classpath中指定的jar包及 Djava.class.path 所指定目录下的类和jar包。

4. Custom ClassLoader 通过java.lang.ClassLoader的子类自定义加载class，属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader。

加载过程中会先检查类是否被已加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap ClassLoader逐层检查，只要某个classloader已加载，就视为已加载此类，保证此类只所有ClassLoader加载一次。而加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类。

#### 3.5.5.结束生命周期

在如下几种情况下，Java虚拟机将结束生命周期

1. 执行了System.exit()方法

2. 程序正常执行结束

3. 程序在执行过程中遇到了异常或错误而异常终止

4. 由于操作系统出现错误而导致Java虚拟机进程终止

### 3.6.方法区（Method Area）

在JVM中，类型信息和类静态变量都保存在方法区中，类型信息是由类加载器在类加载的过程中从类文件中提取出来的信息。

需要注意的一点是，常量池也存放于方法区中。

程序中所有的线程共享一个方法区，所以访问方法区的信息必须确保线程是安全的。如果有两个线程同时去加载一个类，那么只能有一个线程被允许去加载这个类，另一个必须等待。

在程序运行时，方法区的大小是可以改变的，程序在运行时可以扩展。

方法区也可以被垃圾回收，但条件非常严苛，必须在该类没有任何引用的情况下

**类型信息包括什么？:**

1. 类型的全名（The fully qualified name of the type）

2. 类型的父类型全名（除非没有父类型，或者父类型是java.lang.Object）（The fully qualified name of the typeís direct superclass）

3. 该类型是一个类还是接口（class or an interface）（Whether or not the type is a class ）

4. 类型的修饰符（public，private，protected，static，final，volatile，transient等）（The typeís modifiers）

5. 所有父接口全名的列表（An ordered list of the fully qualified names of any direct superinterfaces）

6. 类型的字段信息（Field information）

7. 类型的方法信息（Method information）

8. 所有静态类变量（非常量）信息（All class (static) variables declared in the type, except constants）

9. 一个指向类加载器的引用（A reference to class ClassLoader）

10. 一个指向Class类的引用（A reference to class Class）

11. 基本类型的常量池（The constant pool for the type）

**方法列表（Method Tables）**
为了更高效的访问所有保存在方法区中的数据，在方法区中，除了保存上边的这些类型信息之外，还有一个为了加快存取速度而设计的数据结构：方法列表。每一个被加载的非抽象类，Java虚拟机都会为他们产生一个方法列表，这个列表中保存了这个类可能调用的所有实例方法的引用，保存那些父类中调用的方法。

### 3.7.Java堆（JVM堆、Heap）和Java栈（JVM栈、Stack）

#### 3.7.1.Java堆（JVM堆、Heap）

当Java创建一个类的实例对象或者数组时，都在堆中为新的对象分配内存。

虚拟机中只有一个堆，程序中所有的线程都共享它。

堆占用的内存空间是最多的。

堆的存取类型为管道类型，先进先出。

在程序运行中，可以动态的分配堆的内存大小。

堆的内存资源回收是交给JVM GC进行管理的

#### 3.7.2.Java栈（JVM栈、Stack）

在Java栈中只保存基础数据类型和自定义对象的引用，注意只是对象的引用而不是对象本身哦，对象是保存在堆区中的。

拓展知识：像String、Integer、Byte、Short、Long、Character、Boolean这六个属于包装类型，它们是存放于堆中的。

栈的存取类型为类似于水杯，先进后出。

栈内的数据在超出其作用域后，会被自动释放掉，它不由JVM GC管理。

每一个线程都包含一个栈区，每个栈中的数据都是私有的，其他栈不能访问。

每个线程都会建立一个操作栈，每个栈又包含了若干个栈帧，每个栈帧对应着每个方法的每次调用，每个栈帧包含了三部分：

局部变量区（方法内基本类型变量、变量对象指针）

操作数栈区（存放方法执行过程中产生的中间结果）

运行环境区（动态连接、正确的方法返回相关信息、异常捕捉）

#### 3.7.3.本地方法栈

本地方法栈的功能和JVM栈非常类似，用于存储本地方法的局部变量表，本地方法的操作数栈等信息。

栈的存取类型为类似于水杯，先进后出。

栈内的数据在超出其作用域后，会被自动释放掉，它不由JVM GC管理。

每一个线程都包含一个栈区，每个栈中的数据都是私有的，其他栈不能访问。

本地方法栈是在程序调用或JVM调用本地方法接口（Native）时候启用。

本地方法都不是使用Java语言编写的，比如使用C语言编写的本地方法，本地方法也不由JVM去运行，所以本地方法的运行不受JVM管理。

HotSpot VM将本地方法栈和JVM栈合并了。

#### 3.7.4.程序计数器

在JVM的概念模型里，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

JVM的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，为了各条线程之间的切换后计数器能恢复到正确的执行位置，所以每条线程都会有一个独立的程序计数器。

程序计数器仅占很小的一块内存空间。

当线程正在执行一个Java方法，程序计数器记录的是正在执行的JVM字节码指令的地址。如果正在执行的是一个Natvie（本地方法），那么这个计数器的值则为空（Underfined）。

程序计数器这个内存区域是唯一一个在JVM规范中没有规定任何OutOfMemoryError（内存不足错误）的区域。

#### 3.7.5.JVM执行引擎

Java虚拟机相当于一台虚拟的“物理机”，这两种机器都有代码执行能力，其区别主要是物理机的执行引擎是直接建立在处理器、硬件、指令集和操作系统层面上的。而JVM的执行引擎是自己实现的，因此程序员可以自行制定指令集和执行引擎的结构体系，因此能够执行那些不被硬件直接支持的指令集格式。

在JVM规范中制定了虚拟机字节码执行引擎的概念模型，这个模型称之为JVM执行引擎的统一外观。JVM实现中，可能会有两种的执行方式：解释执行（通过解释器执行）和编译执行（通过即时编译器产生本地代码）。有些虚拟机只采用一种执行方式，有些则可能同时采用两种，甚至有可能包含几个不同级别的编译器执行引擎。

输入的是字节码文件、处理过程是等效字节码解析过程、输出的是执行结果。在这三点上每个JVM执行引擎都是一致的。

#### 3.7.6.本地方法接口（JNI）

JNI是Java Native Interface的缩写，它提供了若干的API实现了Java和其他语言的通信（主要是C和C++）。

**JNI的适用场景**
当我们有一些旧的库，已经使用C语言编写好了，如果要移植到Java上来，非常浪费时间，而JNI可以支持Java程序与C语言编写的库进行交互，这样就不必要进行移植了。或者是与硬件、操作系统进行交互、提高程序的性能等，都可以使用JNI。需要注意的一点是需要保证本地代码能工作在任何Java虚拟机环境。

**JNI的副作用**
一旦使用JNI，Java程序将丢失了Java平台的两个优点：

1. 程序不再跨平台，要想跨平台，必须在不同的系统环境下程序编译配置本地语言部分。2. 程序不再是绝对安全的，本地代码的使用不当可能会导致整个程序崩溃。一个通用规则是，调用本地方法应该集中在少数的几个类当中，这样就降低了Java和其他语言之间的耦合。

### 3.8.JVM GC（垃圾回收机制）

在学习Java GC 之前，我们需要记住一个单词：stop-the-world 。它会在任何一种GC算法中发生。stop-the-world 意味着JVM因为需要执行GC而停止了应用程序的执行。当stop-the-world 发生时，除GC所需的线程外，所有的线程都进入等待状态，直到GC任务完成。GC优化很多时候就是减少stop-the-world 的发生。

**JVM GC回收哪个区域内的垃圾？**

需要注意的是，JVM GC只回收堆区和方法区内的对象。而栈区的数据，在超出作用域后会被JVM自动释放掉，所以其不在JVM GC的管理范围内。

**JVM GC怎么判断对象可以被回收了？**

- 对象没有引用

- 作用域发生未捕获异常

- 程序在作用域正常执行完毕

- 程序执行了System.exit()

- 程序发生意外终止（被杀线程等）

在Java程序中不能显式的分配和注销缓存，因为这些事情JVM都帮我们做了，那就是GC。

有些时候我们可以将相关的对象设置成null 来试图显示的清除缓存，但是并不是设置为null 就会一定被标记为可回收，有可能会发生逃逸。

将对象设置成null 至少没有什么坏处，但是使用System.gc() 便不可取了，使用System.gc() 时候并不是马上执行GC操作，而是会等待一段时间，甚至不执行，而且System.gc() 如果被执行，会触发Full GC ，这非常影响性能。

**JVM GC什么时候执行？**

eden区空间不够存放新对象的时候，执行Minro GC。升到老年代的对象大于老年代剩余空间的时候执行Full GC，或者小于的时候被HandlePromotionFailure 参数强制Full GC 。调优主要是减少 Full GC 的触发次数，可以通过 NewRatio 控制新生代转老年代的比例，通过MaxTenuringThreshold 设置对象进入老年代的年龄阀值（后面会介绍到）。

**按代的垃圾回收机制:**

**新生代（Young generation）**：绝大多数最新被创建的对象都会被分配到这里，由于大部分在创建后很快变得不可达，很多对象被创建在新生代，然后“消失”。对象从这个区域“消失”的过程我们称之为：Minor GC 。

**老年代（Old generation）**：对象没有变得不可达，并且从新生代周期中存活了下来，会被拷贝到这里。其区域分配的空间要比新生代多。也正由于其相对大的空间，发生在老年代的GC次数要比新生代少得多。对象从老年代中消失的过程，称之为：Major GC 或者 Full GC。

**持久代（Permanent generation）**也称之为 方法区（Method area）：用于保存类常量以及字符串常量。注意，这个区域不是用于存储那些从老年代存活下来的对象，这个区域也可能发生GC。发生在这个区域的GC事件也被算为 Major GC 。只不过在这个区域发生GC的条件非常严苛，必须符合以下三种条件才会被回收：

1. 所有实例被回收

2. 加载该类的ClassLoader 被回收

3. Class 对象无法通过任何途径访问（包括反射）

可能我们会有疑问：

如果老年代的对象需要引用新生代的对象，会发生什么呢？

为了解决这个问题，老年代中存在一个 card table ，它是一个512byte大小的块。所有老年代的对象指向新生代对象的引用都会被记录在这个表中。当针对新生代执行GC的时候，只需要查询 card table 来决定是否可以被回收，而不用查询整个老年代。这个 card table 由一个write barrier 来管理。write barrier给GC带来了很大的性能提升，虽然由此可能带来一些开销，但完全是值得的。

默认的新生代（Young generation）、老年代（Old generation）所占空间比例为 1 : 2 。

新生代空间的构成与逻辑

为了更好的理解GC，我们来学习新生代的构成，它用来保存那些第一次被创建的对象，它被分成三个空间：

· 一个伊甸园空间（Eden）

· 两个幸存者空间（Fron Survivor、To Survivor）

默认新生代空间的分配：Eden : Fron : To = 8 : 1 : 1

每个空间的执行顺序如下：

1. 绝大多数刚刚被创建的对象会存放在伊甸园空间（Eden）。

2. 在伊甸园空间执行第一次GC（Minor GC）之后，存活的对象被移动到其中一个幸存者空间（Survivor）。

3. 此后，每次伊甸园空间执行GC后，存活的对象会被堆积在同一个幸存者空间。

4. 当一个幸存者空间饱和，还在存活的对象会被移动到另一个幸存者空间。然后会清空已经饱和的哪个幸存者空间。

5. 在以上步骤中重复N次（N = MaxTenuringThreshold（年龄阀值设定，默认15））依然存活的对象，就会被移动到老年代。

从上面的步骤可以发现，两个幸存者空间，必须有一个是保持空的。如果两个两个幸存者空间都有数据，或两个空间都是空的，那一定是你的系统出现了某种错误。

我们需要重点记住的是，对象在刚刚被创建之后，是保存在伊甸园空间的（Eden）。那些长期存活的对象会经由幸存者空间（Survivor）转存到老年代空间（Old generation）。

也有例外出现，对于一些比较大的对象（需要分配一块比较大的连续内存空间）则直接进入到老年代。一般在Survivor 空间不足的情况下发生。

老年代空间的构成与逻辑

老年代空间的构成其实很简单，它不像新生代空间那样划分为几个区域，它只有一个区域，里面存储的对象并不像新生代空间绝大部分都是朝闻道，夕死矣。这里的对象几乎都是从Survivor 空间中熬过来的，它们绝不会轻易的狗带。因此，Full GC（Major GC）发生的次数不会有Minor GC 那么频繁，并且做一次Major GC 的时间比Minor GC 要更长（约10倍）。

#### 3.8.1.JVM GC 算法讲解

**1、根搜索算法:**

根搜索算法是从离散数学中的图论引入的，程序把所有引用关系看作一张图，从一个节点GC ROOT 开始，寻找对应的引用节点，找到这个节点后，继续寻找这个节点的引用节点。当所有的引用节点寻找完毕后，剩余的节点则被认为是没有被引用到的节点，即无用的节点。

![1、根搜索算法](https://pic1.zhimg.com/80/v2-de35903caa0764079dc5980c21610cbc_720w.png)

上图红色为无用的节点，可以被回收。

目前Java中可以作为GC ROOT的对象有：

1. 虚拟机栈中引用的对象（本地变量表）

2. 方法区中静态属性引用的对象

3. 方法区中常亮引用的对象

4. 本地方法栈中引用的对象（Native对象）

基本所有GC算法都引用根搜索算法这种概念。

**2、标记 - 清除算法:**

![标记 - 清除算法](https://pic1.zhimg.com/80/v2-1e4e744442d9dc1adefd6805a1bfdea0_720w.png)

标记-清除算法采用从根集合进行扫描，对存活的对象进行标记，标记完毕后，再扫描整个空间中未被标记的对象进行直接回收，如上图。

标记-清除算法不需要进行对象的移动，并且仅对不存活的对象进行处理，在存活的对象比较多的情况下极为高效，但由于标记-清除算法直接回收不存活的对象，并没有对还存活的对象进行整理，因此会导致内存碎片。

**3、复制算法:**

![3、复制算法](https://pic2.zhimg.com/80/v2-b95db02b09702215228e2dc5575f499d_720w.png)

复制算法将内存划分为两个区间，使用此算法时，所有动态分配的对象都只能分配在其中一个区间（活动区间），而另外一个区间（空间区间）则是空闲的。
复制算法采用从根集合扫描，将存活的对象复制到空闲区间，当扫描完毕活动区间后，会的将活动区间一次性全部回收。此时原本的空闲区间变成了活动区间。下次GC时候又会重复刚才的操作，以此循环。

复制算法在存活对象比较少的时候，极为高效，但是带来的成本是牺牲一半的内存空间用于进行对象的移动。所以复制算法的使用场景，必须是对象的存活率非常低才行，而且最重要的是，我们需要克服50%内存的浪费。

**4、标记 - 整理算法:**

![4、标记 - 整理算法](https://pic1.zhimg.com/80/v2-c5ac27ceecebacef2d56cdcbc839cf2c_720w.png)

标记-整理算法采用 标记-清除 算法一样的方式进行对象的标记、清除，但在回收不存活的对象占用的空间后，会将所有存活的对象往左端空闲空间移动，并更新对应的指针。标记-整理 算法是在标记-清除 算法之上，又进行了对象的移动排序整理，因此成本更高，但却解决了内存碎片的问题。
JVM为了优化内存的回收，使用了分代回收的方式，对于新生代内存的回收（Minor GC）主要采用复制算法。而对于老年代的回收（Major GC），大多采用标记-整理算法。

#### 3.8.2.垃圾回收器简介

需要注意的是，每一个回收器都存在Stop The World 的问题，只不过各个回收器在Stop The World 时间优化程度、算法的不同，可根据自身需求选择适合的回收器。

**1、Serial（-XX:+UseSerialGC）:**

从名字我们可以看出，这是一个串行收集器。

Serial收集器是Java虚拟机中最基本、历史最悠久的收集器。在JDK1.3之前是Java虚拟机新生代收集器的唯一选择。目前也是ClientVM下ServerVM 4核4GB以下机器默认垃圾回收器。Serial收集器并不是只能使用一个CPU进行收集，而是当JVM需要进行垃圾回收的时候，需暂停所有的用户线程，直到回收结束。

使用算法：复制算法

![使用算法：复制算法](https://pic2.zhimg.com/80/v2-33af6f44a6a5464f2541bb49e0752159_720w.jpg)

JVM中文名称为Java虚拟机，因此它像一台虚拟的电脑在工作，而其中的每一个线程都被认为是JVM的一个处理器，因此图中的CPU0、CPU1实际上为用户的线程，而不是真正的机器CPU，不要误解哦。
Serial收集器虽然是最老的，但是它对于限定单个CPU的环境来说，由于没有线程交互的开销，专心做垃圾收集，所以它在这种情况下是相对于其他收集器中最高效的。

**2、SerialOld（-XX:+UseSerialGC）:**

SerialOld是Serial收集器的老年代收集器版本，它同样是一个单线程收集器，这个收集器目前主要用于Client模式下使用。如果在Server模式下，它主要还有两大用途：一个是在JDK1.5及之前的版本中与Parallel Scavenge收集器搭配使用，另外一个就是作为CMS收集器的后备预案，如果CMS出现Concurrent Mode Failure，则SerialOld将作为后备收集器。

使用算法：标记 - 整理算法

运行示意图与上图一致。

**3、ParNew（-XX:+UseParNewGC）:**

ParNew其实就是Serial收集器的多线程版本。除了Serial收集器外，只有它能与CMS收集器配合工作。

使用算法：复制算法

![ParNew](https://pic1.zhimg.com/80/v2-1afd63adcd9100ba1c8b2fc4f389a270_720w.jpg)

ParNew是许多运行在Server模式下的JVM首选的新生代收集器。但是在单CPU的情况下，它的效率远远低于Serial收集器，所以一定要注意使用场景。

**4、ParallelScavenge（-XX:+UseParallelGC）:**

ParallelScavenge又被称为吞吐量优先收集器，和ParNew 收集器类似，是一个新生代收集器。

使用算法：复制算法

ParallelScavenge收集器的目标是达到一个可控件的吞吐量，所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量 = 运行用户代码时间 / （运行用户代码时间 + 垃圾收集时间）。如果虚拟机总共运行了100分钟，其中垃圾收集花了1分钟，那么吞吐量就是99% 。

**5、ParallelOld（-XX:+UseParallelOldGC）:**

ParallelOld是并行收集器，和SerialOld一样，ParallelOld是一个老年代收集器，是老年代吞吐量优先的一个收集器。这个收集器在JDK1.6之后才开始提供的，在此之前，ParallelScavenge只能选择SerialOld来作为其老年代的收集器，这严重拖累了ParallelScavenge整体的速度。而ParallelOld的出现后，“吞吐量优先”收集器才名副其实！

使用算法：标记 - 整理算法

![使用算法：标记 - 整理算法](https://pic1.zhimg.com/80/v2-cb8bf7030009c8bdff0638ea96a27eb8_720w.jpg)

在注重吞吐量与CPU数量大于1的情况下，都可以优先考虑ParallelScavenge + ParalleloOld收集器。

6、CMS （-XX:+UseConcMarkSweepGC）

CMS是一个老年代收集器，全称 Concurrent Low Pause Collector，是JDK1.4后期开始引用的新GC收集器，在JDK1.5、1.6中得到了进一步的改进。它是对于响应时间的重要性需求大于吞吐量要求的收集器。对于要求服务器响应速度高的情况下，使用CMS非常合适。

CMS的一大特点，就是用两次短暂的暂停来代替串行或并行标记整理算法时候的长暂停。

使用算法：标记 - 清理

CMS的执行过程如下：

- 初始标记（STW initial mark）

在这个阶段，需要虚拟机停顿正在执行的应用线程，官方的叫法STW（Stop Tow World）。这个过程从根对象扫描直接关联的对象，并作标记。这个过程会很快的完成。

- 并发标记（Concurrent marking）

这个阶段紧随初始标记阶段，在“初始标记”的基础上继续向下追溯标记。注意这里是并发标记，表示用户线程可以和GC线程一起并发执行，这个阶段不会暂停用户的线程哦。

- 并发预清理（Concurrent precleaning）

这个阶段任然是并发的，JVM查找正在执行“并发标记”阶段时候进入老年代的对象（可能这时会有对象从新生代晋升到老年代，或被分配到老年代）。通过重新扫描，减少在一个阶段“重新标记”的工作，因为下一阶段会STW。

- 重新标记（STW remark）

这个阶段会再次暂停正在执行的应用线程，重新重根对象开始查找并标记并发阶段遗漏的对象（在并发标记阶段结束后对象状态的更新导致），并处理对象关联。这一次耗时会比“初始标记”更长，并且这个阶段可以并行标记。

- 并发清理（Concurrent sweeping）

这个阶段是并发的，应用线程和GC清除线程可以一起并发执行。

- 并发重置（Concurrent reset）

这个阶段任然是并发的，重置CMS收集器的数据结构，等待下一次垃圾回收。

**CMS的缺点：**

1. 内存碎片。由于使用了 标记-清理 算法，导致内存空间中会产生内存碎片。不过CMS收集器做了一些小的优化，就是把未分配的空间汇总成一个列表，当有JVM需要分配内存空间的时候，会搜索这个列表找到符合条件的空间来存储这个对象。但是内存碎片的问题依然存在，如果一个对象需要3块连续的空间来存储，因为内存碎片的原因，寻找不到这样的空间，就会导致Full GC。

2. 需要更多的CPU资源。由于使用了并发处理，很多情况下都是GC线程和应用线程并发执行的，这样就需要占用更多的CPU资源，也是牺牲了一定吞吐量的原因。

3. 需要更大的堆空间。因为CMS标记阶段应用程序的线程还是执行的，那么就会有堆空间继续分配的问题，为了保障CMS在回收堆空间之前还有空间分配给新加入的对象，必须预留一部分空间。CMS默认在老年代空间使用68%时候启动垃圾回收。可以通过-XX:CMSinitiatingOccupancyFraction=n来设置这个阀值。

**7、GarbageFirst（G1）:**

这是一个新的垃圾回收器，既可以回收新生代也可以回收老年代，SunHotSpot1.6u14以上EarlyAccess版本加入了这个回收器，Sun公司预期SunHotSpot1.7发布正式版本。通过重新划分内存区域，整合优化CMS，同时注重吞吐量和响应时间。杯具的是Oracle收购这个收集器之后将其用于商用收费版收集器。因此目前暂时没有发现哪个公司使用它，这个放在之后再去研究吧。

整理一下新生代和老年代的收集器。

**新生代收集器：**

Serial （-XX:+UseSerialGC）

ParNew（-XX:+UseParNewGC）

ParallelScavenge（-XX:+UseParallelGC）

G1 收集器

**老年代收集器：**

SerialOld（-XX:+UseSerialOldGC）

ParallelOld（-XX:+UseParallelOldGC）

CMS（-XX:+UseConcMarkSweepGC）

G1 收集器

## 4.分布式锁

## 5.ORM实现
