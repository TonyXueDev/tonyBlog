# JVM上篇：内存与垃圾回收篇

## 1. JVM与Java体系结构

### 1.1 前言

作为Java工程师的你曾被伤害过吗？你是否也遇到过这些问题？

- 运行着的线上系统突然卡死，系统无法访问，甚至直接OOM

- 想解决线上JVM GC问题，但却无从下手

- 新项目上线，对各种JVM参数设置一脸茫然，直接默认吧然后就GG了

- 每次面试之前都要重新背一遍JVM的一些原理概念性的东西，然而面试官却经常问你在实际项目中如何调优VM参数，如何解决GC、OOM等问题，一脸懵逼

![image-20230105185452866](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105185454.png)

大部分Java开发人员，除会在项目中使用到与Java平台相关的各种高精尖技术，对于Java技术的核心Java虚拟机了解甚少。

### 1.2 **开发人员如何看待上层框架**

一些有一定工作经验的开发人员，打心眼儿里觉得SSM、微服务等上层技术才是重点，基础技术并不重要，这其实是一种本末倒置的“病态”。

如果我们把核心类库的API比做数学公式的话，那么Java虚拟机的知识就好比公式的推导过程。

![image-20230105185528322](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105185532.png)

计算机系统体系对我们来说越来越远，在不了解底层实现方式的前提下，通过高级语言很容易编写程序代码。但事实上计算机并不认识高级语言

![image-20230622100520761](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622100520.png)

### 1.3 **我们为什么要学习JVM？**

- 面试的需要（BATJ、TMD，PKQ等面试都爱问）

- 中高级程序员必备技能

- - 项目管理、调优的需求

- 追求极客的精神

- - 比如：垃圾回收算法、JIT、底层原理

### 1.4 **Java vs C++**

![image-20230105185621908](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105185624.png)

垃圾收集机制为我们打理了很多繁琐的工作，大大提高了开发的效率，但是，垃圾收集也不是万能的，懂得JVM内部的内存结构、工作机制，是设计高扩展性应用和诊断运行时问题的基础，也是Java工程师进阶的必备能力。



### 1.5 面向人群及参考书目

![image-20230105185722178](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105185737.png)

![image-20230105185730717](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105185740.png)

![image-20230105185750537](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105185754.png)

![image-20230105185804337](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105185807.png)

### 1.6 Java及JVM简介

**TIOBE语言热度排行榜：**[**index | TIOBE - The Software Quality Company**](https://tiobe.com/tiobe-index/)

![image-20230105190222199](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190225.png)

世界上没有最好的编程语言，只有最适用于具体应用场景的编程语言

**JVM：跨语言的平台**

Java是目前应用最为广泛的软件开发平台之一。随着Java以及Java社区的不断壮大Java 也早已不再是简简单单的一门计算机语言了，它更是一个平台、一种文化、一个社区。

- 作为一个平台，Java虚拟机扮演着举足轻重的作用

- - Groovy、Scala、JRuby、Kotlin等都是Java平台的一部分

- 作为灯种文化，Java几乎成为了“开源”的代名词。

- - 第三方开源软件和框架。如Tomcat、Struts，MyBatis，Spring等。

- - 就连JDK和JVM自身也有不少开源的实现，如openJDK、Harmony。

- 作为一个社区，Java拥有全世界最多的技术拥护者和开源社区支持，有数不清的论坛和资料。从桌面应用软件、嵌入式开发到企业级应用、后台服务器、中间件，都可以看到Java的身影。其应用形式之复杂、参与人数之众多也令人咋舌。

![image-20230105190251008](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190253.png)

每个语言都需要转换成字节码文件，最后转换的字节码文件都能通过Java虚拟机进行运行和处理

![image-20230105190313952](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190316.png)

- 随着Java7的正式发布，Java虚拟机的设计者们通过JSR-292规范基本实现在Java虚拟机平台上运行非Java语言编写的程序。

- Java虚拟机根本不关心运行在其内部的程序到底是使用何种编程语言编写的，它只关心“字节码”文件。也就是说Java虚拟机拥有语言无关性，并不会单纯地与Java语言“终身绑定”，只要其他编程语言的编译结果满足并包含Java虚拟机的内部指令集、符号表以及其他的辅助信息，它就是一个有效的字节码文件，就能够被虚拟机所识别并装载运行。

**字节码**

- 我们平时说的java字节码，指的是用java语言编译成的字节码。准确的说任何能在jvm平台上执行的字节码格式都是一样的。所以应该统称为：jvm字节码。

- 不同的编译器，可以编译出相同的字节码文件，字节码文件也可以在不同的JVM上运行。

- Java虚拟机与Java语言并没有必然的联系，它只与特定的二进制文件格式—Class文件格式所关联，Class文件中包含了Java虚拟机指令集（或者称为字节码、Bytecodes）和符号表，还有一些其他辅助信息。



**多语言混合编程**

- Java平台上的多语言混合编程正成为主流，通过特定领域的语言去解决特定领域的问题是当前软件开发应对日趋复杂的项目需求的一个方向。

- 试想一下，在一个项目之中，并行处理用Clojure语言编写，展示层使用JRuby/Rails，中间层则是Java，每个应用层都将使用不同的编程语言来完成，而且，接口对每一层的开发者都是透明的，各种语言之间的交互不存在任何困难，就像使用自己语言的原生API一样方便，因为它们最终都运行在一个虚拟机之上。

- 对这些运行于Java虚拟机之上、Java之外的语言，来自系统级的、底层的支持正在迅速增强，以JSR-292为核心的一系列项目和功能改进（如Da Vinci Machine项目、Nashorn引擎、InvokeDynamic指令、java.lang.invoke包等），推动Java虚拟机从“Java语言的虚拟机”向 “多语言虚拟机”的方向发展。



**如何真正搞懂JVM？**

Java虚拟机非常复杂，要想真正理解它的工作原理，最好的方式就是自己动手编写一个！

自己动手写一个Java虚拟机，难吗？

天下事有难易乎？

为之，则难者亦易矣；不为，则易者亦难矣

![image-20230105190355669](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190359.png)

### 1.7 Java发展的重大事件

- 1990年，在Sun计算机公司中，由Patrick Naughton、MikeSheridan及James Gosling领导的小组Green Team，开发出的新的程序语言，命名为oak，后期命名为Java

- 1995年，Sun正式发布Java和HotJava产品，Java首次公开亮相。

- 1996年1月23日，Sun Microsystems发布了JDK 1.0。

- 1998年，JDK1.2版本发布。同时，sun发布了JSP/Servlet、EJB规范，以及将Java分成了J2EE、J2SE和J2ME。这表明了Java开始向企业、桌面应用和移动设备应用3大领域挺进。

- 2000年，JDK1.3发布，Java HotSpot Virtual Machine正式发布，成为Java的默认虚拟机。

- 2002年，JDK1.4发布，古老的Classic虚拟机退出历史舞台。

- 2003年年底，Java平台的Scala正式发布，同年Groovy也加入了Java阵营。

- 2004年，JDK1.5发布。同时JDK1.5改名为JavaSE5.0。

- 2006年，JDK6发布。同年，Java开源并建立了OpenJDK。顺理成章，Hotspot虚拟机也成为了openJDK中的默认虚拟机。

- 2007年，Java平台迎来了新伙伴Clojure。

- 2008年，Oracle收购了BEA，得到了JRockit虚拟机。

- 2009年，Twitter宣布把后台大部分程序从Ruby迁移到Scala，这是Java平台的又一次大规模应用。

- 2010年，Oracle收购了Sun，获得Java商标和最真价值的HotSpot虚拟机。此时，Oracle拥有市场占用率最高的两款虚拟机HotSpot和JRockit，并计划在未来对它们进行整合：HotRockit

- 2011年，JDK7发布。在JDK1.7u4中，正式启用了新的垃圾回收器G1。

- 2017年，JDK9发布。将G1设置为默认Gc，替代CMS

- 同年，IBM的J9开源，形成了现在的Open J9社区

- 2018年，Android的Java侵权案判决，Google赔偿Oracle计88亿美元

- 同年，Oracle宣告JavaEE成为历史名词JDBC、JMS、Servlet赠予Eclipse基金会

- 同年，JDK11发布，LTS版本的JDK，发布革命性的ZGC，调整JDK授权许可

- 2019年，JDK12发布，加入RedHat领导开发的shenandoah GC

![image-20230105190437418](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190441.png)

在JDK11之前，OracleJDK中还会存在一些OpenJDK中没有的、闭源的功能。但在JDK11中，我们可以认为OpenJDK和OracleJDK代码实质上已经完全一致的程度。



不过，主流的 JDK 8 在2019年01月之后就被宣布停止更新了。另外， JDK 11 及以后的版本也不再提供免费的长期支持（LTS），而且 JDK 15 和 JDK 16 也不是一个长期支持的版本，最新的 JDK 15 只支持 6 个月时间，到 2021 年 3 月，所以千万不要把 JDK 15 等非长期支持版本用在生产。

![image-20230105190506599](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190509.png)

### 1.8 虚拟机与Java虚拟机

**虚拟机**

所谓虚拟机（Virtual Machine），就是一台虚拟的计算机。它是一款软件，用来执行一系列虚拟计算机指令。大体上，虚拟机可以分为系统虚拟机和程序虚拟机。

- 大名鼎鼎的Visual Box，Mware就属于系统虚拟机，它们完全是对物理计算机的仿真，提供了一个可运行完整操作系统的软件平台。

- 程序虚拟机的典型代表就是Java虚拟机，它专门为执行单个计算机程序而设计，在Java虚拟机中执行的指令我们称为Java字节码指令。

无论是系统虚拟机还是程序虚拟机，在上面运行的软件都被限制于虚拟机提供的资源中。



**Java虚拟机**

- Java虚拟机是一台执行Java字节码的虚拟计算机，它拥有独立的运行机制，其运行的Java字节码也未必由Java语言编译而成。

- JVM平台的各种语言可以共享Java虚拟机带来的跨平台性、优秀的垃圾回器，以及可靠的即时编译器。

- Java技术的核心就是Java虚拟机（JVM，Java Virtual Machine），因为所有的Java程序都运行在Java虚拟机内部。



作用

- Java虚拟机就是二进制字节码的运行环境，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。每一条Java指令，Java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数，处理结果放在哪里。



特点

- 一次编译，到处运行

- 自动内存管理

- 自动垃圾回收功能

**JVM的位置**

![image-20230105190559759](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190602.png)

JVM是运行在操作系统之上的，它与硬件没有直接的交互

![image-20230105190621602](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230105190623.png)

### 1.9 JVM的整体结构(会画)

- HotSpot VM是目前市面上高性能虚拟机的代表作之一。

- 它采用解释器与即时编译器并存的架构。

- 在今天，Java程序的运行性能早已脱胎换骨，已经达到了可以和C/C++程序一较高下的地步。

![image-20230116140317678](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230116140321.png)







### 1.10 Java代码执行流程

![Java代码执行流程](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230116185113.png)

![image-20230124214513953](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230124214514.png)

### 1.11 JVM的架构模型

Java编译器输入的指令流基本上是一种基于`栈的指令集架构`，另外一种指令集架构则是基于`寄存器的指令集`架构。

具体来说：这两种架构之间的区别：

**基于栈式架构的特点**

- 设计和实现更简单，适用于资源受限的系统

- 避开了寄存器的分配难题：使用零地址指令方式分配

- 指令流中的指令大部分是零地址指令，其执行过程依赖于操作栈。指令集更小，编译器容易实现

- 不需要硬件支持，可移植性更好，更好实现跨平台



**基于寄存器架构的特点**

- 典型的应用是x86的二进制指令集：比如传统的PC以及Android的Davlik虚拟机
- 指令集架构则完全依赖硬件，可移植性差
- 性能优秀和执行更高效
- 花费更少的指令去完成一项操作
- 在大部分情况下，基于寄存器架构的指令集往往都以一地址指令、二地址指令和三地址指令为主，而基于栈式架构的指令集却是以零地址指令为主

**举例1**

同样执行2+3这种逻辑操作，其指令分别如下：

基于栈的计算流程（以Java虚拟机为例）：

```java
iconst_2 //常量2入栈
istore_1
iconst_3 // 常量3入栈
istore_2
iload_1
iload_2
iadd //常量2/3出栈，执行相加
istore_0 // 结果5入栈
```

而基于寄存器的计算流程

```java
mov eax,2 //将eax寄存器的值设为1
add eax,3 //使eax寄存器的值加3
```

**举例2**

```java
public int calc(){
    int a=100;
    int b=200;
    int c=300;
    return (a + b) * c;
}
```

```java
> javap -c Test.class
...
public int calc();
    Code:
    Stack=2,Locals=4,Args_size=1
       0: bipush        100
       2: istore_1
       3: sipush        200
       6: istore_2
       7: sipush        300
      10: istore_3
      11: iload_1
      12: iload_2
      13: iadd
      14: iload_3
      15: imul
      16: ireturn
}
```

**总结**

由于跨平台性的设计，`Java的指令都是根据栈来设计的`。不同平台CPU架构不同，所以不能设计为基于寄存器的。优点是跨平台，指令集小，编译器容易实现，缺点是性能下降，实现同样的功能需要更多的指令。

时至今日，尽管嵌入式平台已经不是Java程序的主流运行平台了（准确来说应该是HotSpotVM的宿主环境已经不局限于嵌入式平台了），那么为什么不将架构更换为基于寄存器的架构呢？

### 1.12 JVM的生命周期

**虚拟机的启动**

Java虚拟机的启动是通过引导类加载器（bootstrap class loader）创建一个初始类（initial class）来完成的，这个类是由虚拟机的具体实现（什么厂家的JVM）指定的。

**虚拟机的执行**

* 一个运行中的java虚拟机有着一个清晰的任务：执行Java程序
* 程序开始执行的时候它才运行，程序结束的时候它就停止
* `执行一个所谓的java程序的时候，真真在执行的是一个叫Java虚拟机的进程`

**虚拟机的退出**

有如下的退出

* 程序正常执行结束
* 程序在执行过程中遇到了异常或者错误而异常终止
* 由于操作系统出现错误而导致Java虚拟机进程终止
* 某线程调用Runtime类或者System类的exit方法，或者Runtime类的halt方法，并且java安全管理器也允许这次exit或者halt操作。
* 除此之外，JNI（Java Native Inteface）规范描述了用JNI Invocation API来加载或者卸载Java虚拟机时，Java虚拟机的退出情况。

### 1.13 JVM发展历程

Sun Classic VM

![image-20230622182952699](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622182952.png)

![image-20230622183439717](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622183439.png)

![image-20230622183632192](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622183632.png)

![image-20230622183928987](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622183929.png)

![image-20230622184139916](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622184140.png)

![image-20230622184439321](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622184439.png)

![image-20230622184559267](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622184559.png)

![image-20230622184711058](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622184711.png)

![image-20230622184734887](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622184735.png)

![image-20230622184858144](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622184858.png)

![image-20230622185014397](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622185014.png)

![image-20230622185146883](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622185147.png)

![image-20230622185350063](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622185350.png)

![image-20230622185408495](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622185408.png)

![image-20230622185424220](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622185424.png)

## 2、类加载子系统

### 2.1 内存结构概述

![image-20230622190058130](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622190058.png)

首先需要将class文件加载到内存当中，加载的话就需要类加载子系统，加载到内存当中之后生成class对象，同时对静态的属性进行初始化（主要在方法区）。当真正执行字节码指令的时候，这个时候就是`执行引擎`发挥作用。按照字节码指令依次执行，这里就涉及到了去虚拟机栈中的局部变量表中取数据，包括操作数栈；以及创建对象的时候就用到了堆空间。如果涉及到C的类库那就还需要本地方法栈

![image-20230622190813712](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230622190813.png)

![内存结构](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623004106.png)

### 2.2 类加载器与类的加载过程

#### 2.2.1 类加载子系统的作用

![image-20230623074720209](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623074720.png)

* 类加载器子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识。
* ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。（七大姑（ClassLoader）把女子（class文件）给你推荐了女孩子，至于能不能成就看你（Execution Engine）的本事了）
* 加载的类信息存放于一块称为方法区的内存空间。除了类信息之外，方法区还会存放运行时常量池的信息，可能还包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）

#### 2.2.2 类加载器ClassLoader角色

![image-20230623075517157](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623075517.png)

* class file存在于本地的硬盘上，可以理解为设计师华仔纸上的模板，而最终这个模板在执行的时候是要加载到JVM当中来根据这个文件实例化出n个一模一样的实例。
* class file加载到JVM中被称为DNA元数据模板，放在方法区
* 在.class文件 -> JVM -> 最终成为元数据模板，此过程就要一个运输工具（类加载器 class loader）扮演一个快递员的角色。



#### 2.2.3 类的加载过程

![image-20230623080004765](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623080004.png)

举一个例子：

![image-20230623080131736](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623080131.png)

在运行这个例子的过程当中，咱们的目的是要执行main方法，这个main方法要想执行就需要当前类被加载也就是HelloLoader加载，如果当前的类没有加载就需要相应的classloader来加载，对于自定义的类来说实际上使用的是系统类加载器（应用类加载器），加载过程中如果出现问题了会抛出异常（或者这个字节码文件不是一个合法的文件，那么加载它也会抛出异常），加载成功之后就会在内存中有一个Class对象。然后执行链接，初始化HelloLoader，然后调用HelloLoader的main方法，然后结束

整个过程如下图所示：

![image-20230623080838030](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623080838.png)

* 加载

  * 通过一个类的全限定名获取定义此类的二进制字节流
  * 将这字节流所代表的静态存储结构转化为方法区的运行时数据结构
  * `在内存中生成一个代表这个类的java.lang.Class对象`，作为方法区这个类的各种数据的访问入口

* 链接（linking）

  * 验证（Verify）

    * 目的在于确保Class文件的字节流中包含信息符合当前虚拟机的要求，保证被加载类的正确性，不会危害虚拟机自身的安全

    * 主要包含四种验证，文件格式验证，元数据验证，字节码验证，符号引用验证

      > 为什么会出现不合法呢？字节码文件也是二进制的，黑客可以通过字节码文件来修改文件的内容来攻击我们的系统，所以为了防止字节码文件被恶意修改，所以要有一个验证字节码文件的过程

  * 准备（Prepare）

    * 为类变量分配内存并且设置改类变量的默认初始值，即零值

    * 这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显式初始化；

    * 这里不会为实例变量分配初始化，类变量会分配在方法区，而实例变量是会随着对象一起分配到Java堆中

      

  * 解析（Resolve）

    * 将常量池中的符号引用转换为直接引用的过程
    * 事实上，解析操作往往会伴随着JVM在执行完初始化之后再执行
    * 符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的CLass文件格式中，直接引用就是直接指向目标的指针、相对偏移量或者一个间接定位到目标的句柄。
    * 解析动作主要针对类或者接口、字段、类方法、接口方法、方法类型等。对应常量池中的CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info等

    

* 初始化（initialization）

  * 初始化阶段就是执行类构造器方法<clinit>()的过程
  * 此方法不需要定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来
  * 构造器方法中指令按照语句在源文件中出现的顺序执行
  * <clinit>()不同于类的构造器。（关联：构造器是虚拟机视角下的<init>()）
  * 若该类具有父类，JVM会保证子类的<clinit>()执行前，父类的<clinit>()已经执行完毕
  * 虚拟机必须保证一个类的<clinit>()方法在多线程下被同步加锁。

举一个例子：

```java
package com.txcoder.ssg.jvm.chapter02;

public class ClassInitTest {
    private static int num = 1;

    public static void main(String[] args) {
        System.out.println(ClassInitTest.num);
    }
}

```

然后将这个java文件编译成class文件，然后在idea中安装classlib bytecode的idea插件，然后点中你的java源文件，点击idea中的view -> show byte code with jclasslib

![image-20230623165249068](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623165249.png)

![image-20230623165346456](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623165346.png)

在methods中可以看到<clinit>()，这个方法不是我们自己写的而是它自动帮我们生成的

![image-20230623165550440](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623165550.png)

我们修改下源代码

```java
package com.txcoder.ssg.jvm.chapter02;

public class ClassInitTest {
    private static int num = 1;

    static {
        num ++;
    }

    public static void main(String[] args) {
        System.out.println(ClassInitTest.num);
    }
}

```

![image-20230623165837873](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623165837.png)



我们再修改源代码：

```java
package com.txcoder.ssg.jvm.chapter02;

public class ClassInitTest {
    private static int num = 1;

    static {
        num ++;
        number = 20;
    }

    private static int number = 10;

    public static void main(String[] args) {
        System.out.println(ClassInitTest.num);
        System.out.println(ClassInitTest.number);
    }
}

```

这段代码的运行结果是10，

![image-20230623171415656](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230623171415.png)

在链接的准备阶段，number变量会初始化为0，在initialization的时候会按照代码的顺序将number的值先赋值为20，然后赋值为10



![image-20230624081118046](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624081118.png)

上图中的代码编译不通过，为什么呢？这里的变量可以赋值但是不能被调用

```java
public class ClinitTest {
    private int a = 1;

    public static void main(String[] args) {
        int b = 2;
    }
}

```

上面的代码的字节码中的会有clinit方法吗

![image-20230624083255345](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624083255.png)

答案是没有，因为clinit方法是javac编译器自动收集类中的所有`类变量的赋值动作`和`静态代码块中的语句`合并而来

```java
public class ClinitTest {
    private static int a = 1;

    public static void main(String[] args) {
        int b = 2;
    }
}
```



![image-20230624083522814](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624083522.png)

> 补充：加载.class文件的方式
>
> * 从本地系统中直接加载
> * 通过网络获取，典型场景：Web Applet
> * 从zip压缩包中读取，成为日后jar、war格式的基础
> * 运行时计算生成，使用最多的是：动态代理技术
> * 由其他文件生成，典型场景：JSP应用
> * 从专有数据库中提取.class文件，比较少见
> * 从加密文件中获取，典型的防Class文件被反编译的保护措施

![image-20230624083723405](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624083723.png)

这里的init方法对应的是代码中的构造器方法

### 2.3 类加载器的分类

* JVM支持两种类型的类加载器，分别为`引导类加载器（Bootstrap Classloader）`和`自定义类加载器（User-Defined ClassLoader）`

* 从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是Java虚拟机规范却没有这么定义，`而是将所有派生于抽象类CLassLoader的类加载器都划分为自定义类加载器。`

* 无论类加载器的类型如何划分，在程序中我们最常用的类加载器始终只有3个，如下图所示：

  ![image-20230624085046465](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624085046.png)

```java
package com.txcoder.ssg.jvm.chapter02;

import com.sun.org.apache.bcel.internal.util.ClassLoader;

public class ClassLoaderTest {
    public static void main(String[] args) {
        // 获取系统类加载器
        java.lang.ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader); // sun.misc.Launcher$AppClassLoader@18b4aac2

        // 获取其上层：扩展类加载器
        java.lang.ClassLoader extensionClassLoader = systemClassLoader.getParent();
        System.out.println(extensionClassLoader); // sun.misc.Launcher$ExtClassLoader@24d46ca6

        // 获取其上层：bootstrap classLoader
        java.lang.ClassLoader bootstrapClassLoader = extensionClassLoader.getParent();
        System.out.println(bootstrapClassLoader); // null

        // 对于用户自定义类来说：默认使用系统类加载器进行加载
        java.lang.ClassLoader userDefaultClassLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(userDefaultClassLoader); // sun.misc.Launcher$AppClassLoader@18b4aac2

        // String类是通过引导类加载器加载的----》java的核心类库都是引导类加载器加载的
        java.lang.ClassLoader classLoader = String.class.getClassLoader();
        System.out.println(classLoader); // null

    }
}
```

![image-20230624103853651](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624103853.png)

#### 2.3.1 虚拟机自带的加载器

##### 启动类加载器（引导类加载器，Bootstrap ClassLoader）

* 这个类加载使用C/C++语言实现的，嵌套在JVM内部
* 它用来加载Java的核心库（JAVA_HOME/jre/lib/rt.jar、resources.jar或者sun.boot.class.path路径下的内容），用于提供JVM自身需要的类
* 并不继承自java.lang.ClassLoader没有父加载器
* 加载扩展类和程序类加载器，并指定为他们的父类加载器
* 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类

##### 扩展类加载器（Extension ClassLoader）

* Java语言编写。由sun。misc。Launcher$ExtClassLoader实现
* 派生于ClassLoader类
* 父类加载器为启动类加载器
* 从java.ext.dirs系统属性所指定的目录中加载类库，或者从JDK的安装目录的jre/lib/ext子目录（扩展目录）下加载类库。如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载

##### 应用程序类加载器（系统类加载器，AppClassLoader）

* java语言编写，由sun.misc.Launcher$AppClassLoader实现
* 派生于ClassLoader类
* 父类加载器为扩展类加载器
* 它负责加载环境变量classpath或者系统属性java.class.path指定路径下的类库
* 该类加载是程序中默认的加载器，一般来说，Java应用的类都是由它来完成加载的
* 通过ClassLoader#getSystemClassLoader()方法可以获取到该类加载器

```java
package com.txcoder.ssg.jvm.chapter02;

import com.sun.security.sasl.Provider;
import sun.misc.Launcher;
import sun.security.ec.CurveDB;

import java.net.URL;

public class ClassLoaderTest1 {
    public static void main(String[] args) {
        System.out.println("**********启动类加载器**********");
        // 获取BootstrapClassLoader能够加载的api的路径
        URL[] urLs = Launcher.getBootstrapClassPath().getURLs();
        for (URL url: urLs) {
            System.out.println(url.toExternalForm());
        }

        // 从上面的路径中随意选择一个类，看看它的类加载器是什么：引导类加载器
        ClassLoader classLoader = Provider.class.getClassLoader();
        System.out.println(classLoader);

        System.out.println("**********扩展类加载器**********");
        String extDirs = System.getProperty("java.ext.dirs");
        for (String path: extDirs.split(";")) {
            System.out.println(path);
        }

        // 从上面的路径中随意选择一个类，看看它的类加载器是什么
        ClassLoader classLoader1 = CurveDB.class.getClassLoader();
        System.out.println(classLoader1);
    }
}

```

```tex
**********启动类加载器**********
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/resources.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/rt.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/sunrsasign.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/jsse.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/jce.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/charsets.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/jfr.jar
file:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/classes
null
**********扩展类加载器**********
/Users/xuetao/Library/Java/Extensions:/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home/jre/lib/ext:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java

sun.misc.Launcher$ExtClassLoader@8efb846

```

##### 用户自定义类加载器

* 在java的日常应用程序开发中，类的加载几乎是由上述3种加载器相互配合执行的，在必要的时候，我们还可以自定义类加载器，来定制类的加载方式。
* 为什么要自定义类加载器？
  * 隔离加载类
  * 修改类加载的方式
  * 扩展加载源
  * 防止源码泄漏

##### 用户自定义类加载器的实现步骤

* 开发人员可以通过继承抽象类java.lang.ClassLoader类的方式，实现自己的类加载器，以满足一些特殊的需求
* 在JDK1.2之前，在自定义类加载器的时候总会去继承ClassLoader类并且重写loadClass()方法，从而实现自定义的类加载器，但是在JDK1.2之后已不再建议用户去覆盖loadClass()方法，而是建议把自定义的类加载逻辑卸载findClass()方法中
* 在编写自定义类加载器的时候，如果没有太过于复杂的需求，可以直接继承URLClassLoader类，这样就可以避免自己去编写findClass()方法以及获取字节码流的方式，使自定义类加载器编写更加简洁。

##### 关于ClassLoader

ClassLoader类是一个抽象类，其后所有的类加载器都继承自ClassLoader（不包括启动类加载器）

| 方法名称                                            | 描述                                                         |
| --------------------------------------------------- | ------------------------------------------------------------ |
| getParent()                                         | 返回该类加载器的超类加载器                                   |
| loadClass(String name)                              | 加载名称为name的类，返回结果为java.lang.Class类的实例        |
| findClass(String name)                              | 查找名称为name的类，返回结果为java.lang.Class类的实例        |
| findLoadedClass(String name)                        | 查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例 |
| defineClass(String name,byte[] b, int off, int len) | 把字节数组b中的内容转换为一个Java类，返回结果为java.lang.Class类的实例 |
| resolveClass(Class<?> c)                            | 连接指定的一个Java类                                         |

![image-20230624182629132](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624182629.png)

![image-20230624182636764](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624182636.png)

### 2.4 双亲委派机制

Java虚拟机对class文件采用的是按需加载的方式，也就是说当需要使用该类的时候才会将它的class文件加载到内存生成class对象。而且加载某个类的class文件的时候，Java虚拟机采用的是双亲委派模式，即把请求交给父类处理，它是一种任务委派模式。

#### 2.4.1 工作原理

* 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行。
* 如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器
* 如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，这就是双亲委派模式。

![image-20230624230236523](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624230236.png)

![image-20230624230701441](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624230701.png)

![image-20230624230726176](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624230726.png)

#### 2.4.2 优势

* 避免类的重复加载
* 保护程序安全，防止核心API被随意篡改
  * 自定义类：java.lang.String
  * 自定义类：java.lang.ShkStart

Java.lang.SecurityException: Prohibited package name: java.lang

![image-20230624231123009](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624231123.png)

#### 2.4.3 沙箱安全机制

自定义String类，但是在加载自定义String类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载jdk自带的文件（rt.jar包中java\lang\String.class）报错信息说没有main方法，就是因为加载的是rt.jar保重的String类。这样可以保证对Java核心源代码的保护，这就是沙箱安全机制

![image-20230624230701441](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624230701.png)

#### 2.4.4 其他

* 在JVM中表示两个class对象是否为同一个类存在两个必要条件
  * 类的完整类名必须一致，包括包名
  * 加载这个类的ClassLoader指的是ClassLoader实例对象必须相同
* 换句话说，在JVM中，即使这两个类对象（class对象）来源同一个Class文件，被同一个虚拟机所加载，但是只要加载它们的ClassLoader实例对象不同，那么这两个类对象也是不想等的



#####  对类加载器的引用

JVM必须知道一个类型是由启动加载器加载的还是由用户类加载器加载的。如果一个类型是由用户类加载器加载的，那么JVM会将这个类加载器的一个引用作为类型信息的一部分保存在方法区中。当解析一个类型到另一个类型的引用的时候，JVM需要保证这两个类型的类加载器是相同的。

##### 类的主动使用和被动使用

Java程序对类的使用方式分为：主动使用和被动使用

* 主动使用，又分为七种情况：

  * 创建类的实例

  * 访问某个类或者接口的静态变量，或者对该静态变量赋值

  * 调用类的静态方法

  * 反射（比如：Class.forName("com.atguigu.Test")）

  * 初始化一个类的子类

  * Java虚拟机启动的时候被标明动态语言支持：java.lang.invoke.MethodHandle实例的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic句柄对应的类没有初始化，则初始化

    

* 除了以上七种情况，其他使用Java类的方式都被看作是对类的被动使用，都不会导致类的初始化





## 3、运行时数据区概述及线程

###  3.1 运行时数据区概述

![image-20230624232953875](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624232954.png)

![image-20230624233043272](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624233043.png)

内存是非常重要的系统资源，是硬盘和CPU的中间仓库以及桥梁，承载着操作系统和应用程序的实时运行。JVM内存布局规定了Java在运行过程中内存申请、分配、管理的策略，保证了JVM的高效稳定的运行，不同的JVM对于内存的划分方式和管理机制存在着部分差异。结合JVM虚拟机规范，来探讨一下经典的JVM内存布局。

运行时数据区如下图：

![image-20230624234124259](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624234124.png)

Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则与线程一一对应的，这些与线程对应的数据区域会随着线程开始和结束而创建和销毁。

灰色的为单独线程私有的，红色的为多个线程共享的。即：

* 每个线程：独立包括程序计数器、栈、本地栈
* 线程间共享：堆、堆外内存（永久代或者元空间、代码缓存）

![image-20230624234520978](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624234521.png)

![image-20230624234549415](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624234549.png)

![image-20230624234807877](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624234808.png)

### 3.2 线程

* 线程是一个程序里的运行单元。JVM允许一个应用有多个线程并行的执行
* 在Hotspot JVM里，每个线程都与操作系统的本地线程直接映射。
  * 当一个Java线程准备好执行以后，此时一个操作系统的本地线程也同时创建。Java线程执行终结后，本地线程也会回收
* 操作系统负责所有线程的安排调度到任何一个可用的CPU上。一旦本地线程初始化成功，它就会调用Java线程中的run()方法。

![image-20230624235610204](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624235610.png)

## 4、运行时数据区详解

### 4.1 程序计数器（PC寄存器）

#### 4.1.1 PC寄存器介绍

请参考资料：![image-20230624235845074](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230624235845.png)

![image-20230625000001730](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625000001.png)

JVM中的程序计数寄存器（Program Counter Register）中，Register的命名源于CPU的寄存器，寄存器存储指令相关的现场信息。CPU只有把数据装载到寄存器才能够运行。

这里，并非是广义上所指的物理寄存器，或许将其翻译为PC计数器（或者指令计数器）会更加贴切（也称为程序钩子），并且也不容易引起一些不必要的误会。`JVM的PC寄存器是对物理PC寄存器的一种抽象模拟。`



![image-20230625000528482](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625000528.png)

* 它是一块很小的内存空间，几乎可以忽略不计。也是运行速度最快的存储区域

* 在JVM规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致

* 任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行native方法，则是未指定值（undefined）

* 它是程序控制流的指示器，分之、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成

* 字节码解释器工作时就是通过改变这个计数器的值俩选取下一条需要执行的字节码指令

* 它是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域

* 既没有GC也没有OOM

  

#### 4.1.2 举例说明

```java
package com.txcoder.ssg.jvm.chapter04;

public class PCRegisterTest {
    public static void main(String[] args) {
        int i = 10;
        int j = 20;
        int k = i + j;
    }
}

```

执行命令

```shell
javap -v PCRegisterTest.class
```

```class
Classfile /Users/xuetao/ssg/ssg_jvm/target/classes/com/txcoder/ssg/jvm/chapter04/PCRegisterTest.class
  Last modified Jun 25, 2023; size 543 bytes
  MD5 checksum 8698a8b93865c7da27d6a6e6691f1517
  Compiled from "PCRegisterTest.java"
public class com.txcoder.ssg.jvm.chapter04.PCRegisterTest
  minor version: 0
  major version: 52
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #2                          // com/txcoder/ssg/jvm/chapter04/PCRegisterTest
  super_class: #3                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #3.#22         // java/lang/Object."<init>":()V
   #2 = Class              #23            // com/txcoder/ssg/jvm/chapter04/PCRegisterTest
   #3 = Class              #24            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lcom/txcoder/ssg/jvm/chapter04/PCRegisterTest;
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               args
  #14 = Utf8               [Ljava/lang/String;
  #15 = Utf8               i
  #16 = Utf8               I
  #17 = Utf8               j
  #18 = Utf8               k
  #19 = Utf8               MethodParameters
  #20 = Utf8               SourceFile
  #21 = Utf8               PCRegisterTest.java
  #22 = NameAndType        #4:#5          // "<init>":()V
  #23 = Utf8               com/txcoder/ssg/jvm/chapter04/PCRegisterTest
  #24 = Utf8               java/lang/Object
{
  public com.txcoder.ssg.jvm.chapter04.PCRegisterTest();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/txcoder/ssg/jvm/chapter04/PCRegisterTest;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        10   // 0代表偏移地址/指令地址，bipush指的是操作指令
         2: istore_1
         3: bipush        20
         5: istore_2
         6: iload_1
         7: iload_2
         8: iadd
         9: istore_3
        10: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  args   [Ljava/lang/String;
            3       8     1     i   I
            6       5     2     j   I
           10       1     3     k   I
    MethodParameters:
      Name                           Flags
      args
}
SourceFile: "PCRegisterTest.java"
```

![image-20230625051324751](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625051324.png)

指令地址5存储在PC寄存器中，然后执行引擎会去PC寄存器中的5这个地址中取istore_2这个指令。执行引擎负载操作局部变量表和操作数栈来实现数据的存取等，还有将字节码指令翻译成机器指令然后CPU就可以帮我们执行这些机器指令

#### 4.1.3 两个常见问题解析

使用PC寄存器存储字节码指令地址有什么用呢？为什么使用PC寄存器记录当前线程的执行地址呢？

> 因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行。
>
> JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令

![image-20230625052115107](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625052115.png)

PC寄存器为什么会被设定为线程私有？

> 我们都知道所谓的多线程在一个特定的时间段内只会执行其中某一个线程的方法。CPU会不停的做任务切换，这样必然导致经常中断或者恢复，如何保证丝毫无差呢？为了能够准确的记录各个线程正在执行的当前字节码指令地址，最好的办法是为每一个线程都分配一个PC寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。
>
> 由于CPU时间片轮限制，众多线程在并发执行的过程中，任何一个确定的时刻，一个处理器或者多核处理器中的一个内核只会执行某个线程中的一条指令。
>
> 这样必然导致经常中断或者恢复，如何保证分毫无差呢？每个线程子啊创建之后，都会产生自己的程序计数器和栈帧，程序计数器在各个线程之间互不影响。

CPU时间片

> CPU时间片即CPU分配给各个程序的时间，每个线程都分配一个时间段，称作它的时间片。
>
> 在宏观上：我们可以同时打开多个应用程序，每个程序并行不悖，同时运行。
>
> 但是在微观上：由于只有一个CPU，一次只能处理程序要求的一部分。如何处理公平，一种方法就是引入时间片，每个程序轮流执行。
> ![image-20230625053140167](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625053140.png)

### 4.2 虚拟机栈

#### 4.2.1 虚拟机栈概述

##### 虚拟机栈出现的背景

由于跨平台性的设计，Java的指令都是根据栈来设计的。不同平台CPU架构不同，所以不能设计为基于寄存器的

优点是跨平台，指令集小，编译器容易发现，缺点是性能下降，实现同样的功能需要更多的指令

##### 初步影响

有不少Java开发人员一提到Java内存结构，就会非常粗粒度的将JVM中的内存区理解为仅有Java堆（heap）和Java（Stack）栈。

栈是运行时的单位，而堆是存储的单位

> 即：栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。堆解决的是数据存储的问题，即数据怎么放，放在哪里。

![image-20230625054303272](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625054303.png)

![image-20230625062144746](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625062144.png)

##### 虚拟机栈基本内容

* Java虚拟机栈是什么
  Java虚拟机栈（Java Virtual Machine Stack）早期也叫做Java栈。每个线程在创建的时候都会创建一个虚拟机栈，其内部保存着一个个的栈帧（Stack Frame），对应着一次次的Java方法的调用
  * 是线程私有的
* 生命周期
  生命周期和线程一致
* 作用
  主管Java程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。

##### 栈的特点（优点）

* 栈是一种快速有效的分配存储的方式，访问速度仅次于程序计数器
* JVM直接对Java栈的操作只有两个：
  * 每个方法执行，伴随着进栈（入栈、压栈）
  * 执行结束后的出栈工作
* 对于栈来说不存在垃圾回收的问题（存在OOM，stackoverflow）

![image-20230625062840887](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625062841.png)

##### 栈中可能出现的异常

Java虚拟机规范允许Java栈的大小是动态的或者固定不变的

* 如果采用固定大小的Java虚拟机栈，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过Java虚拟机栈允许的最大容量，Java虚拟机将会抛出一个StackOverFlowError异常。
* 如果Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，活着子啊创建新的线程的时候没有足够的内存区创建对应的虚拟机栈，那么Java虚拟机将会抛出一个OutOfMemoryError异常

设置栈内存大小

* 我们可以使用`-Xss`选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度

#### 4.2.2 栈的存储单元

栈中存储的是什么？

* 每个线程都有自己的栈，栈中的数据都是以`栈帧（Stack Frame）的格式存在`
* 在这个线程上正在执行的每个`方法`都各自对应一个`栈帧`（Stack Frame）
* 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息
* JVM直接对Java栈的操作只有两个，就是对栈帧的压栈和出栈，遵循“先进后出”或者“后进先出”

原则

* 在一条活动线程中，一个时间点上，只会有一个活动的栈帧，即只有当前正在执行的方法的栈帧（栈顶栈帧）是有效的，这个栈帧被称为当前栈帧（Current Frame），与当前栈帧相对应的方法就是当前方法（Current Method），定义这个方法的类就是当前类（Current Class）
* 执行引擎运行的所有字节码指令只针对当前栈帧进行操作
* 如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，成为新的当前栈帧

![image-20230625065430955](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625065431.png)

![image-20230625065834229](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625065834.png)

* 不同线程中所包含的栈帧是不允许存在相互引用的，即不可能在一个栈帧之中引用另一个线程的栈帧
* 如果当前方法调用了其他方法，方法返回之际，当前栈帧会返回此方法的执行结果给前一个栈帧，接着虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧。
* Java方法有两种返回函数的方法，`一种是正常的函数返回，使用return指令；另外一个是抛出异常。不管使用哪种方式，都会导致栈帧被弹出。`



##### 栈帧的内部结构

每个栈帧都存储着：

* `局部变量表（Local Variables）`
  * `操作数栈（Operand Stack）（或者表达式栈)`
* 动态链接（Dynamic Linking）（或者指向运行时常量池的方法引用）
* 方法返回地址（Return Address）（或者方法正常退出或者异常退出的定义）
* 一些附加信息

![image-20230625071335283](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625071335.png)

![image-20230625071721122](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625071721.png)

#### 4.2.3 局部变量表

* 局部变量表也被称为局部变量数组或者本地变量表
* 定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量，这些数据类型包括各类基本数据类型、对象引用（reference）、以及returnAddress类型
* 由于局部变量表是建立在线程的栈上，是线程的私有数据，因此不存在数据安全问题
* 局部变量表所需要的容量大小是在编译期确定下来的，并且保存在方法的Code属性的maximum local variables数据项中。在方法运行期间是不会改变局部变量表的大小的。

![image-20230625152153318](https://tonyxue.oss-cn-shanghai.aliyuncs.com/note_img/20230625152154.png)

方法嵌套调用的次数由栈的大小决定。一般来说，栈越大，方法的嵌套调用次数越多。对于一个函数来说，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需要传递的信息增大的需求。进而函数调用就会占用更多的栈空间，导致其嵌套调用此时就会减少。



局部变量表中的变量值在当前方法调用中有效。在方法执行的时候，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁。

##### 关于Slot的理解

* 参数值的存放总是在局部变量数组的index0开始，到数组长度-1的索引结束
* 局部变量表，最基本的存储单元是Slot（变量槽）
* 局部变量表中存放编译器可知的各种基本数据类型（8种），引用类型（reference），returnAddress类型的变量。
* 在局部变量表中，32位以内的类型只占用一个slot（包括returnAddress类型），64位类型（long和double）占用两个slot
  * byte、short、char在存储前被转换为int，boolean也被转换为int，0表示false，非0表示true
  * long和double则占据两个slot

#### 4.2.4 操作数栈









#### 4.2.5 代码追踪





#### 4.2.6 栈顶缓存技术











#### 4.2.7 动态链接









#### 4.2.8 方法的调用：解析与分派











#### 4.2.9 方法返回地址









#### 4.2.10 一些附加信息









#### 4.2.11 栈的相关面试题


