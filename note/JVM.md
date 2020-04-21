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

​		是被线程共享的一块内存区域，<font color="003399">创建的对象和数组都保存在Java堆内存中，也是垃圾收集器进行垃圾收集的最重要的内存区域。</font>由于现代VM采用分代收集算法，因此Java堆从GC的角度还可以细分为：**新生代**(Eden区、From Survivor区和To Survivor区)和**老年代**。

---

#### 2.2.5 方法区/永久代(线程共享 Thread Share)

​		方法区即我们常说的**永久代(Permanent Generation)**，用于存储**被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码**等数据。Hotspot VM把GC分代收集扩展至方法区，即**使用Java堆的永久代来实现方法区**，这样Hotspot的垃圾收集器就可以像管理Java堆一样管理这部分内存，而不必为方法区开发专门的内存管理器(永久带的内存回收的主要目标是针对**常量池的回收**和**类型的卸载**，因此收益一般很小)。

​		<font color="#dd0000">运行时常量池(Runtime Constant Pool)</font>是方法区的一部分。class文件中除了有类的版本、字段、方法、接口的描述等信息外，还有一项信息是常量池(Constant Pool Table)，用于存放编译期生成的各种字面量和符号引用，这部分内容在类加载后存放到方法区的运行时常量池中。Java虚拟机对class文件的每一部分(包括常量池)的格式都有严格的规定，每一个字节用于存储那种数据都必须符合规范上的要求，这样才会被虚拟机认可、装载和执行。

---

### 2.3 JVM运行时内存

​		Java堆从GC的角度还可以细分为：**新生代**(Eden区、From Survivor区和To Survivor区)和**老年代**。

![Java堆从GC的角度细分](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_6.png "Java堆从GC的角度细分")

---

#### 2.3.1  新生代

​		是用来存放新生的对象。一般占据堆的1/3空间。由于频繁创建对象，所以新生代会频繁出发[Minor GC](https://www.cnblogs.com/williamjie/p/9516264.html)进行垃圾回收。[新生代](https://www.cnblogs.com/jswang/p/9056038.html)又分为：Eden区、From Survivor区、To Survivor三个区。

---

##### 2.3.1.1 Eden 区

​		<font color="003399">Java新对象的出生地</font>(如果新创建的对象占用内存很大，则直接分配到老年代)。当Eden区内存不够的时候会触发Minor GC，对新生代区进行一次垃圾回收。

---

##### 2.3.1.2 From Survivor 区

​		上一次GC的幸存者，作为这一次GC的被扫描者。

---

##### 2.3.1.3 To Survivor 区

​		保留了一次Minor GC过程中的幸存者。

---

##### 2.3.1.4 Minor GC 的过程

```java
	复制——>清空——>互换
```

​		Minor GC采用<font color="003399">复制算法</font>。

**1. Eden、From Survivor 复制到 To Survivor，年龄 +1**

​		首先，把Eden和From Survivor区域中存货的对象复制到To Survivor区域(如果有对象的年龄已经达到了老年的标准，则复制到老年代区)，同时把这些对象的年龄+1(如果To Survivor位置不够了就放到老年区)。

**2. 清空Eden、From Survivor**

​		然后清空Eden和From Survivor中的对象。

**3. To Survivor和 From Survivor互换**

​		最后，To Survivor 和 From Survivor互换，原To Survivor成为下一次GC时的 From Survivor区。

---

#### 2.3.2 老年代

​		主要存放应用程序中生命周期长的内存对象。

​		老年代的对象比较稳定，所以 Major GC 不会频繁执行。在进行Major GC前一般都先进行了一次Minor GC，使得有新生代对象晋身入老年代，导致老年代空间不够用时才触发。当无法找到足够大的连续空间分配给新创建的较大对象时也会出发一次Major GC 进行垃圾回收腾出空间。

​		Major GC 采用<font color="003399">标记清除算法</font>：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。Major GC的耗时比较长，因为要扫描再回收。Major GC会产生内存碎片，为了减少内存消耗，我们一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候就会抛出OOM(Out of Memory)异常。

---

#### 2.3.3 永久代

​		指内存的永久保存区域，主要存放class和Meta(元数据)的信息，class在被加载的时候被放入永久区域，它和存放实例的区域不同，<font color="003399"> GC不会在主程序运行期对永久区域进行清理。</font>所以这也导致了永久代的区域会随着class的增多而空间不够，最终抛出OOM异常。

---

##### 2.3.3.1 Java 8 与元数据

​		在Java 8中，<font color="003399">永久代已经被移除，被一个称为“元数据区”(元空间)的区域所取代。</font>元空间的本质和永久代类似，元空间与永久代最大的区别在于：<font color="003399">元空间并不在虚拟机中，而是使用本地内存。</font>因此，默认情况下，元空间的大小只受本地内存限制。<font color="003399">类的元数据放入 native memory，字符串池和类的静态变量放入Java堆中</font>，这样可以加载多少类的元数据就不再由[MaxPermSize](https://www.cnblogs.com/mingforyou/p/2378143.html)控制,而是由系统实际可用空间来控制。

---

### 2.4 垃圾回收与算法

![JVM垃圾回收与算法](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_7.png "JVM垃圾回收与算法")

---

#### 2.4.1 如何确定垃圾

##### 2.4.1.1 引用计数法

​		在Java中，引用和对象是有关联的。如果要操作对象则必须用引用进行。因此可以通过引用计数来判断一个对象是否可以回收。简单说，即一个<font color="003399">对象如果没有任何与之关联的引用，即他们的引用计数都为0，则说明对象为垃圾对象，可被GC回收。</font>

```java
public class GcDemo{
    public static void main(String[] args){
        GcObject obj1 = new GcObject();
        GcObject obj2 = new GcObject();
        
        obj1.instance = obj2;
        obj2.instance = obj1;
        
        obj1 = null;
        obj2 = null;
    }
}

class GcObject{
    public Object instance = null;
}
```

​		上述代码中obj1和obj2指向的对象都已经不可能再被访问，彼此互相引用对方导致引用计数都不为0，最终无法被GC回收，而可达性算法能解决这个问题。

##### 2.4.1.2 可达性分析

​		为了解决引用计数法的循环引用问题，Java使用了可达性分析的方法。通过一系列的“GC roots”对象作为起点搜索。<font color="003399">如果在“GC roots”和一个对象之间没有可达路径，则称该对象是不可达的。</font>要注意的是，不可达对象不等价于可回收对象，<font color="003399">不可达对象变为可回收对象至少要经过两次标记过程。</font>两次标记后仍然是不可达对象，则将面临回收。

![可达性分析](https://images2018.cnblogs.com/blog/1368961/201804/1368961-20180406113127652-1645765895.png "可达性分析")

​		由上图可以得知实例1、2、4、6都具有GC roots可达性，即存活对象，不可被GC回收的对象。而实例3、5虽然连通，但是没有与GC roots相连，即GC roots不可达对象，会被GC回收的对象。

#### 2.4.2 标记清除算法(Mark-Sweep)

​		该算法是最基础的垃圾回收算法，其分为两个阶段，<font color="003399">标注和清除</font>。标记阶段标记出所有需要回收的对象，清除阶段回收被标记的对象所占用的空间。如图：

![标记清除示意图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_8.png "标记清除示意图")

​		从图中可以发现，该算法最大的问题是内存碎片化严重，后续可能会出现大对象找不到可利用的空间的问题。

#### 2.4.3 复制算法(copying)

​		该算法是为了解决Mark-Sweep算法内存碎片化的缺陷而被提出的算法。按内存容量将内存划分为等大小的两块。每次只使用其中的一块，当这一块内存满后将尚存货的对象复制到另一块上去，把已使用的内存清掉，如图：

![复制算法](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_9.png "复制算法")

​		这种算法虽然实现简单，内存效率高，不易产生碎片，但是最大的问题是可用内存被压缩到了原来的一半。且存活对象增多的话，copying算法效率会很低。



























