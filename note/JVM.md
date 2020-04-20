# JVM

## 基本概念

​		JVM是可运行Java代码的假想计算机，包括一套字节码指令集、一组寄存器、一个栈、一个垃圾回收，堆 和 一个存储方法域。JVM是运行在操作系统之上的，与硬件没有直接的交互。

![JVM基本概念](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_1.png "jvm基本概念")

## 运行过程

​		Java源文件通过编译器，产生.class文件(字节码文件)。字节码文件又通过Java虚拟机中的解释器，编译成特定机器上的机器码。

```java
	①Java源文件——>编译器(javac.exe)——>字节码文件
	②字节码文件——>JVM解释器(java.exe)——>机器码
```

​		每一种平台的解释器是不同的，但是实现的虚拟机是相同的。当一个程序从开始运行，这是虚拟机就开始实例化了，多个程序启动就会存在多个虚拟机实例。程序退出或关闭，则虚拟机实例消亡，多个虚拟机实例之间数据不能共享。

![JVM虚拟机实例](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_2.png "JVM虚拟机实例")

---

### 2.1 线程

​		这里所说的线程指程序执行过程中的一个线程实体。JVM允许一个应用并发执行多个线程。[Hotspot VM](https://www.cnblogs.com/baxianhua/p/9528192.html)中的Java线程与原生操作系统线程有直接的映射关系。<font color="#dd0000">当线程本地存储、缓冲区分配、同步对象、栈、程序计数器等准备好后，就会创建一个操作系统原生线程。Java线程结束，原生线程随之被回收。操作系统负责调度所有线程，并把它们分配到可用的CPU上。当原生线程初始化完毕，就会调用Java线程的run()方法。当线程结束时会释放原生线程和Java线程的所有资源。</font>

​		Hotspot VM后台运行的系统线程主要有下面几个：

<table class="table-striped table-condensed">
    <tr>
        <td>虚拟机线程(VM Thread)</td>
    	<td>这个线程等待JVM到达安全点操作出现。这些操作必须要在独立的线程里执行，因为当堆修改无法进行时，线程都需要JVM位于安全点。这些操作的类型有：stop-the-world垃圾回收、线程栈dump、线程暂停、线程偏向锁(biased locking)解除</td>
    </tr>
    <tr>
        <td>周期性任务线程</td>
    	<td>这个线程负责定时器事件(中断)。用来调度周期性操作的执行</td>
    </tr>
    <tr>
        <td>GC线程</td>
        <td>这些线程支持JVM中不同的垃圾回收活动</td>
    </tr>
    <tr>
        <td>解释器线程(java.exe)</td>
        <td>这些线程在运行时将字节码动态编译成本地平台相关的机器码</td>
    </tr>
    <tr>
        <td>信号分发线程</td>
        <td>这个线程接收发送到JVM的信号并调用适当的JVM方法处理</td>
    </tr>
</table>

---

### 2.2 JVM内存区域

​		JVM内存区域主要分为：线程私有区域(Thread Local)【程序计数器、虚拟机栈、本地方法区】、线程共享区(Thread Shared)【Java堆、方法区】、直接内存(Direct Memory)

![JVM内存区域](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_3.png "JVM内存区域")

​		<font color="#dd0000">线程私有(Thread Local)数据区域生命周期与线程相同，依赖用户线程的启动/结束，而 创建/销毁(在 Hotspot VM内)</font>，每个线程都与操作系统的本地线程直接映射，因此这部分内存区域的存、否跟随本地线程的生/死对应。

​		<font color="#dd0000">线程共享区(Thread Share)域随虚拟机的 启动/关闭 而 创建/销毁。</font>

​		<font color="#dd0000">直接内存(direct Memory)并不是JVM运行时数据区的一部分，</font>但也会被频繁地使用：在JDK1.4引入的<font color="#dd0000">NIO提供了基于Channel与Buffer的IO方式，它可以使用Native函数库直接分配堆外内存，然后使用DirectByteBuffer对象作为这块内存的引用进行操作(详见：Java I/O 扩展)，这样就避免了在Java堆和Native堆中来回复制数据，因此在一些场景中可以显著提高性能。</font>

![JVM数据区域](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_4.png "JVM数据区域")

---

#### 2.2.1 程序计数器(线程私有 Thread Local)

​		一块较小的内存空间，<font color="#dd0000">是当前线程所执行的字节码行号指示器</font>，每条线程都要有一个独立的程序计数器，这类内存也成为“线程私有”内存。

​		正在执行Java方法的话，程序计数器记录的时虚拟机字节码指令的地址(当前指令的地址)。如果还是[Native方法](https://www.jianshu.com/p/22517a150fe5)，则为空。

​		这个内存区域是唯一一个在虚拟机中没有规定任何OutOfMemoryError情况的区域。

---

#### 2.2.2 虚拟机栈(线程私有 Thread Local)

​		<font color="#dd0000">是描述Java方法执行的内存模型，每个方法在执行的同时都会创建的一个栈帧(Stack Frame)用于存储局部变量表、操作数栈、动态链接、方法出口等信息。</font><font color="003399">每一个方法从调用直至执行完成的过程，就对应一个栈帧在虚拟机栈中从入栈到出栈的过程。</font>

​		栈帧(Frame)是用来存储数据的部分过程结果的数据结构，同时也被用来处理动态链接(Dynamic Linking)、方法返回值和异常分派(Dispatch Exception)。<font color="003399">栈帧随着方法调用而创建，随着方法结束而销毁</font>——无论方法是正常完成还是异常完成(抛出了在方法内未被捕获的异常)都算作方法结束

![栈帧](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_5.png "栈帧")

---

#### 2.2.3 本地方法栈(线程私有 Thread Local)

​		本地方法栈和 Java Stack 作用类似, 区别是虚拟机栈为执行 Java 方法服务, 而本地方法栈则为 Native 方法服务, 如果一个 VM 实现使用 C-linkage 模型来支持 Native 调用, 那么该栈将会是一个 C 栈，但 Hotspot VM 直接就把本地方法栈和虚拟机栈合二为一。

---

#### 2.2.4 堆(Heap-线程共享 Thread Share)-运行时数据区

​		是被线程共享的一块内存区域，<font color="003399">创建的对象和数组都保存在Java堆内存中，也是垃圾收集器进行垃圾收集的最重要的内存区域。</font>由于现代VM采用分代收集算法，因此Java堆从GC的角度还可以细分为：**新生代**(Eden区Share、From Survivor区和To Survivor区)和**老年代**。

---

#### 2.2.5 方法区/永久代(线程共享 Thread Share)

​		方法区即我们常说的**永久代(Permanent Generation)**，用于存储**被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码**等数据。Hotspot VM把GC分代收集扩展至方法区，即**使用Java堆的永久代来实现方法区**，这样Hotspot的垃圾收集器就可以像管理Java堆一样管理这部分内存，而不必为方法区开发专门的内存管理器(永久带的内存回收的主要目标是针对**常量池的回收**和**类型的卸载**，因此收益一般很小)。

​		<font color="#dd0000">运行时常量池(Runtime Constant Pool)</font>是方法区的一部分。class文件中除了有类的版本、字段、方法、接口的描述等信息外，还有一项信息是常量池(Constant Pool Table)，用于存放编译期生成的各种字面量和符号引用，这部分内容在类加载后存放到方法区的运行时常量池中。Java虚拟机对class文件的每一部分(包括常量池)的格式都有严格的规定，每一个字节用于存储那种数据都必须符合规范上的要求，这样才会被虚拟机认可、装载和执行。

### 2.3 JVM运行时内存

​		Java堆从GC的角度还可以细分为：**新生代**(Eden区Share、From Survivor区和To Survivor区)和**老年代**。

![Java堆从GC的角度细分](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_6.png "Java堆从GC的角度细分")

#### 2.3.1  新生代























