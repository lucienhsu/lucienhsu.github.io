---
layout: post
title: Java类加载器ClassLoader总结
subtitle: 死磕系列
date: 2019-01-09
author: lucienhsu
header-img: img/post-bg-default.jpg
header-mask: 0.5
stickie: true
catalog: true
tags:
  - Java
---


# JAVA类装载方式，有两种:
1. 隐式装载， 程序在运行过程中当碰到通过new 等方式生成对象时，隐式调用类装载器加载对应的类到jvm中。 
2. 显式装载， 通过class.forname()等方法，显式加载需要的类

# 类加载的动态性体现:
一个应用程序总是由n多个类组成，Java程序启动时，并不是一次把所有的类全部加载后再运行，它总是先把保证程序运行的基础类一次性加载到jvm中，其它类等到jvm用到的时候再加载，这样的好处是节省了内存的开销，因为java最早就是为嵌入式系统而设计的，内存宝贵，这是一种可以理解的机制，而用到时再加载这也是java动态性的一种体现。

# java类装载器
JDK 默认提供了如下几种ClassLoader
1. Bootstrp loader  
Bootstrp加载器是用C++语言写的，它是在Java虚拟机启动后初始化的，它主要负责加载`%JAVA_HOME%/jre/lib`,`-Xbootclasspath`参数指定的路径以及`%JAVA_HOME%/jre/classes`中的类。
2. ExtClassLoader  
Bootstrp loader加载ExtClassLoader,并且将ExtClassLoader的父加载器设置为`Bootstrp loader.ExtClassLoader`是用Java写的，具体来说就是`sun.misc.Launcher$ExtClassLoader`，ExtClassLoader主要加载`%JAVA_HOME%/jre/lib/ext`，此路径下的所有classes目录以及`java.ext.dirs`系统变量指定的路径中类库。
3. AppClassLoader   
Bootstrp loader加载完ExtClassLoader后，就会加载AppClassLoader,并且将AppClassLoader的父加载器指定为 ExtClassLoader。AppClassLoader也是用Java写成的，它的实现类是 `sun.misc.Launcher$AppClassLoader`，另外我们知道ClassLoader中有个`getSystemClassLoader`方法,此方法返回的正是`AppclassLoader.AppClassLoader`主要负责加载classpath所指定的位置的类或者是jar文档，它也是Java程序默认的类加载器。

综上所述，它们之间的关系可以通过下图形象的描述：    
![FAB法则](https://github.com/lucienhsu/lucienhsu.github.io/raw/master/img/post/java-classloader.jpg) 

> 为什么要有三个类加载器，一方面是分工，各自负责各自的区块，另一方面为了实现委托模型。

# 类加载器之间是如何协调工作的

前面说了，java中有三个类加载器，问题就来了，碰到一个类需要加载时，它们之间是如何协调工作的，即java是如何区分一个类该由哪个类加载器来完成呢。 在这里java采用了`委托模型机制`，这个机制简单来讲，就是“**类装载器有载入类的需求时，会先请示其Parent使用其搜索路径帮忙载入，如果Parent 找不到,那么才由自己依照自己的搜索路径搜索类**”.

下面举一个例子来说明，为了更好的理解，先弄清楚几行代码：

``` Java
Public class Test{
    Public static void main(String[] arg){
        //获取Test类的类加载器
        ClassLoader c  = Test.class.getClassLoader();  
        System.out.println(c); 
        //获取c这个类加载器的父类加载器
        ClassLoader c1 = c.getParent();  
        System.out.println(c1);
        //获取c1这个类加载器的父类加载器
        ClassLoader c2 = c1.getParent();
        System.out.println(c2);
  }
}

// 运行结果：
……AppClassLoader……
……ExtClassLoader……
Null
```
可以看出Test是由AppClassLoader加载器加载的，AppClassLoader的Parent 加载器是 ExtClassLoader,但是ExtClassLoader的Parent为 null 是怎么回事呵，朋友们留意的话，前面有提到Bootstrap Loader是用C++语言写的，依java的观点来看，逻辑上并不存在Bootstrap Loader的类实体，所以在java程序代码里试图打印出其内容时，我们就会看到输出为null。

# 类装载器ClassLoader（一个抽象类）描述一下JVM加载class文件的原理机制
类装载器就是寻找类或接口字节码文件进行解析并构造JVM内部对象表示的组件，在java中类装载器把一个类装入JVM，经过以下步骤：
1.  装载：查找和导入Class文件 
2. 链接：其中解析步骤是可以选择的 
    - 检查：检查载入的class文件数据的正确性 
    - 准备：给类的静态变量分配存储空间 
    - 解析：将符号引用转成直接引用 
3. 初始化：对静态变量，静态代码块执行初始化工作

类装载工作由ClassLoder和其子类负责。    
JVM在运行时会产生三个ClassLoader：根装载器，ExtClassLoader(扩展类装载器)和AppClassLoader，其中根装载器不是ClassLoader的子类，由C++编写，因此在java中看不到他，负责装载JRE的核心类库，如JRE目录下的rt.jar,charsets.jar等。   
ExtClassLoader是ClassLoder的子类，负责装载JRE扩展目录ext下的jar类包；   
AppClassLoader负责装载classpath路径下的类包，这三个类装载器存在父子层级关系****，即根装载器是ExtClassLoader的父装载器，ExtClassLoader是AppClassLoader的父装载器。默认情况下使用AppClassLoader装载应用程序的类

Java装载类使用“全盘负责委托机制”。  
“全盘负责”是指当一个ClassLoder装载一个类时，除非显示的使用另外一个ClassLoder，该类所依赖及引用的类也由这个ClassLoder载入；  
“委托机制”是指先委托父类装载器寻找目标类，只有在找不到的情况下才从自己的类路径中查找并装载目标类。
这一点是从安全方面考虑的，试想如果一个人写了一个恶意的基础类（如java.lang.String）并加载到JVM将会引起严重的后果，但有了全盘负责制，java.lang.String永远是由根装载器来装载，避免以上情况发生。     
除了JVM默认的三个ClassLoder以外，第三方可以编写自己的类装载器，以实现一些特殊的需求。类文件被装载解析后，在JVM中都有一个对应的java.lang.Class对象，提供了类结构信息的描述。数组，枚举及基本数据类型，甚至void都拥有对应的Class对象。Class类没有public的构造方法，Class对象是在装载类时由JVM通过调用类装载器中的defineClass()方法自动构造的。

## 为什么要使用这种双亲委托模式呢？
因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。
考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义类型，这样会存在非常大的安全隐患，而双亲委托的方式，就可以避免这种情况，因为String已经在启动时被加载，所以用户自定义类是无法加载一个自定义的ClassLoader。
> 思考：假如我们自己写了一个java.lang.String的类，我们是否可以替换调JDK本身的类？答案是否定的。  
我们不能实现。为什么呢？我看很多网上解释是说双亲委托机制解决这个问题，其实不是非常的准确。因为双亲委托机制是可以打破的，你完全可以自己写一个classLoader来加载自己写的java.lang.String类，但是你会发现也不会加载成功，具体就是因为针对java.*开头的类，jvm的实现中已经保证了必须由bootstrp来加载。

# 定义自已的ClassLoader
 
既然JVM已经提供了默认的类加载器，为什么还要定义自已的类加载器呢？

因为Java中提供的默认ClassLoader，只加载指定目录下的jar和class，如果我们想加载其它位置的类或jar时，比如：我要加载网络上的一个class文件，通过动态加载到内存之后，要调用这个类中的方法实现我的业务逻辑。在这样的情况下，默认的ClassLoader就不能满足我们的需求了，所以需要定义自己的ClassLoader。
 
定义自已的类加载器分为两步：
1. 继承java.lang.ClassLoader
2. 重写父类的findClass方法
 
读者可能在这里有疑问，父类有那么多方法，为什么偏偏只重写findClass方法？
 
因为JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法，当在loadClass方法中搜索不到类时，loadClass方法就会调用findClass方法来搜索类，所以我们只需重写该方法即可。如没有特殊的要求，一般不建议重写loadClass搜索类的算法。

# 线程上下文类加载器
　　线程上下文类加载器（context class loader）是从 JDK 1.2 开始引入的。类 java.lang.Thread中的方法 getContextClassLoader()和 setContextClassLoader(ClassLoader cl)用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。

　　前面提到的类加载器的代理模式并不能解决 Java 应用开发中会遇到的类加载器的全部问题。Java 提供了很多服务提供者接口（Service Provider Interface，SPI），允许第三方为这些接口提供实现。常见的 SPI 有 JDBC、JCE、JNDI、JAXP 和 JBI 等。这些 SPI 的接口由 Java 核心库来提供，如 JAXP 的 SPI 接口定义包含在 javax.xml.parsers包中。这些 SPI 的实现代码很可能是作为 Java 应用所依赖的 jar 包被包含进来，可以通过类路径（CLASSPATH）来找到，如实现了 JAXP SPI 的 Apache Xerces所包含的 jar 包。SPI 接口中的代码经常需要加载具体的实现类。如 JAXP 中的 javax.xml.parsers.DocumentBuilderFactory类中的 newInstance()方法用来生成一个新的 DocumentBuilderFactory的实例。这里的实例的真正的类是继承自 javax.xml.parsers.DocumentBuilderFactory，由 SPI 的实现所提供的。如在 Apache Xerces 中，实现的类是 org.apache.xerces.jaxp.DocumentBuilderFactoryImpl。而问题在于，SPI 的接口是 Java 核心库的一部分，是由引导类加载器来加载的；SPI 实现的 Java 类一般是由系统类加载器来加载的。引导类加载器是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。它也不能代理给系统类加载器，因为它是系统类加载器的祖先类加载器。也就是说，类加载器的代理模式无法解决这个问题。
　　线程上下文类加载器正好解决了这个问题。如果不做任何的设置，Java 应用的线程的上下文类加载器默认就是系统上下文类加载器。在 SPI 接口的代码中使用线程上下文类加载器，就可以成功的加载到 SPI 实现的类。线程上下文类加载器在很多 SPI 的实现中都会用到。



# 类加载器与Web容器
　　对于运行在 Java EE容器中的 Web 应用来说，类加载器的实现方式与一般的 Java 应用有所不同。不同的 Web 容器的实现方式也会有所不同。以 Apache Tomcat 来说，每个 Web 应用都有一个对应的类加载器实例。该类加载器也使用代理模式，所不同的是它是首先尝试去加载某个类，如果找不到再代理给父类加载器。这与一般类加载器的顺序是相反的。这是 Java Servlet 规范中的推荐做法，其目的是使得 Web 应用自己的类的优先级高于 Web 容器提供的类。这种代理模式的一个例外是：Java 核心库的类是不在查找范围之内的。这也是为了保证 Java 核心库的类型安全。 

绝大多数情况下，Web 应用的开发人员不需要考虑与类加载器相关的细节。下面给出几条简单的原则：
1. 每个 Web 应用自己的 Java 类文件和使用的库的 jar 包，分别放在 WEB-INF/classes和 WEB-INF/lib目录下面。
2. 多个应用共享的 Java 类文件和 jar 包，分别放在 Web 容器指定的由所有 Web 应用共享的目录下面。
3. 当出现找不到类的错误时，检查当前类的类加载器和当前线程的上下文类加载器是否正确。



# 类加载器与OSGi


OSGi是 Java 上的动态模块系统。它为开发人员提供了面向服务和基于组件的运行环境，并提供标准的方式用来管理软件的生命周期。OSGi 已经被实现和部署在很多产品上，在开源社区也得到了广泛的支持。Eclipse就是基于OSGi 技术来构建的。            

OSGi 中的每个模块（bundle）都包含 Java 包和类。模块可以声明它所依赖的需要导入（import）的其它模块的 Java 包和类（通过 Import-Package），也可以声明导出（export）自己的包和类，供其它模块使用（通过 Export-Package）。也就是说需要能够隐藏和共享一个模块中的某些 Java 包和类。这是通过 OSGi 特有的类加载器机制来实现的。OSGi 中的每个模块都有对应的一个类加载器。它负责加载模块自己包含的 Java 包和类。当它需要加载 Java 核心库的类时（以 java开头的包和类），它会代理给父类加载器（通常是启动类加载器）来完成。当它需要加载所导入的 Java 类时，它会代理给导出此 Java 类的模块来完成加载。模块也可以显式的声明某些 Java 包和类，必须由父类加载器来加载。只需要设置系统属性`org.osgi.framework.bootdelegation`的值即可。  

假设有两个模块 bundleA 和 bundleB，它们都有自己对应的类加载器 classLoaderA 和 classLoaderB。在 bundleA 中包含类`com.bundleA.Sample`，并且该类被声明为导出的，也就是说可以被其它模块所使用的。bundleB 声明了导入 bundleA 提供的类 com.bundleA.Sample，并包含一个类`com.bundleB.NewSample`继承自`com.bundleA.Sample`。在 bundleB 启动的时候，其类加载器 classLoaderB 需要加载类`com.bundleB.NewSample`，进而需要加载类 com.bundleA.Sample。由于 bundleB 声明了类`com.bundleA.Sample`是导入的，classLoaderB 把加载类`com.bundleA.Sample`的工作代理给导出该类的 bundleA 的类加载器 classLoaderA。classLoaderA 在其模块内部查找类`com.bundleA.Sample`并定义它，所得到的类 `com.bundleA.Sample`实例就可以被所有声明导入了此类的模块使用。对于以 java开头的类，都是由父类加载器来加载的。如果声明了系统属性 `org.osgi.framework.bootdelegation=com.example.core.*`，那么对于包 com.example.core中的类，都是由父类加载器来完成的。


　　OSGi 模块的这种类加载器结构，使得一个类的不同版本可以共存在 Java 虚拟机中，带来了很大的灵活性。不过它的这种不同，也会给开发人员带来一些麻烦，尤其当模块需要使用第三方提供的库的时候。下面提供几条比较好的建议：
1. 如果一个类库只有一个模块使用，把该类库的 jar 包放在模块中，在 Bundle-ClassPath中指明即可。
2. 如果一个类库被多个模块共用，可以为这个类库单独的创建一个模块，把其它模块需要用到的 Java 包声明为导出的。其它模块声明导入这些类。
3. 如果类库提供了 SPI 接口，并且利用线程上下文类加载器来加载 SPI 实现的 Java 类，有可能会找不到 Java 类。如果出现了 NoClassDefFoundError异常，首先检查当前线程的上下文类加载器是否正确。通过 Thread.currentThread().getContextClassLoader()就可以得到该类加载器。该类加载器应该是该模块对应的类加载器。如果不是的话，可以首先通过 class.getClassLoader()来得到模块对应的类加载器，再通过 `Thread.currentThread().setContextClassLoader()`来设置当前线程的上下文类加载器。

# 参考
1. [http://blog.csdn.net/zhoudaxia/article/details/35897057](http://blog.csdn.net/zhoudaxia/article/details/35897057)
2. [http://blog.jobbole.com/96145/](http://blog.jobbole.com/96145/)
3. [http://my.oschina.net/aminqiao/blog/262601](http://my.oschina.net/aminqiao/blog/262601)

> 转载：[https://www.cnblogs.com/doit8791/p/5820037.html](https://www.cnblogs.com/doit8791/p/5820037.html)