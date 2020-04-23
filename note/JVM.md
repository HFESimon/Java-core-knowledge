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

​		每一种平台的解释器是不同的，但是实现的虚拟机是相同的。当一个程序从开始运行，这时虚拟机就开始实例化了，多个程序启动就会存在多个虚拟机实例。程序退出或关闭，则虚拟机实例消亡，多个虚拟机实例之间数据不能共享。

![JVM虚拟机实例](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_2.png "JVM虚拟机实例")

---

### 1.1 线程

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

### 1.2 JVM内存区域

​		JVM内存区域主要分为：线程私有区域(Thread Local)【程序计数器、虚拟机栈、本地方法区】、线程共享区(Thread Shared)【Java堆、方法区】、直接内存(Direct Memory)

![JVM内存区域](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_3.png "JVM内存区域")

​		<font color="#dd0000">线程私有(Thread Local)数据区域生命周期与线程相同，依赖用户线程的启动/结束，而 创建/销毁(在 Hotspot VM内)</font>，每个线程都与操作系统的本地线程直接映射，因此这部分内存区域的存、否跟随本地线程的生/死对应。

​		<font color="#dd0000">线程共享区(Thread Share)域随虚拟机的 启动/关闭 而 创建/销毁。</font>

​		<font color="#dd0000">直接内存(direct Memory)并不是JVM运行时数据区的一部分，</font>但也会被频繁地使用：在JDK1.4引入的<font color="#dd0000">NIO提供了基于Channel与Buffer的IO方式，它可以使用Native函数库直接分配堆外内存，然后使用DirectByteBuffer对象作为这块内存的引用进行操作(详见：Java I/O 扩展)，这样就避免了在Java堆和Native堆中来回复制数据，因此在一些场景中可以显著提高性能。</font>

![JVM数据区域](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_4.png "JVM数据区域")

---

#### 1.2.1 程序计数器(线程私有 Thread Local)

​		一块较小的内存空间，<font color="#dd0000">是当前线程所执行的字节码行号指示器</font>，每条线程都要有一个独立的程序计数器，这类内存也成为“线程私有”内存。

​		正在执行Java方法的话，程序计数器记录的是虚拟机字节码指令的地址(当前指令的地址)。如果还是[Native方法](https://www.jianshu.com/p/22517a150fe5)，则为空。

​		这个内存区域是唯一一个在虚拟机中没有规定任何OutOfMemoryError情况的区域。

---

#### 1.2.2 虚拟机栈(线程私有 Thread Local)

​		<font color="#dd0000">是描述Java方法执行的内存模型，每个方法在执行的同时都会创建的一个栈帧(Stack Frame)用于存储局部变量表、操作数栈、动态链接、方法出口等信息。</font><font color="003399">每一个方法从调用直至执行完成的过程，就对应一个栈帧在虚拟机栈中从入栈到出栈的过程。</font>

​		栈帧(Frame)是用来存储数据的部分过程结果的数据结构，同时也被用来处理动态链接(Dynamic Linking)、方法返回值和异常分派(Dispatch Exception)。<font color="003399">栈帧随着方法调用而创建，随着方法结束而销毁</font>——无论方法是正常完成还是异常完成(抛出了在方法内未被捕获的异常)都算作方法结束

![栈帧](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_5.png "栈帧")

---

#### 1.2.3 本地方法栈(线程私有 Thread Local)

​		本地方法栈和 Java Stack 作用类似, 区别是虚拟机栈为执行 Java 方法服务, 而本地方法栈则为 Native 方法服务, 如果一个 VM 实现使用 C-linkage 模型来支持 Native 调用, 那么该栈将会是一个 C 栈，但 Hotspot VM 直接就把本地方法栈和虚拟机栈合二为一。

---

#### 1.2.4 堆(Heap-线程共享 Thread Share)-运行时数据区

​		是被线程共享的一块内存区域，<font color="003399">创建的对象和数组都保存在Java堆内存中，也是垃圾收集器进行垃圾收集的最重要的内存区域。</font>由于现代VM采用分代收集算法，因此Java堆从GC的角度还可以细分为：**新生代**(Eden区、From Survivor区和To Survivor区)和**老年代**。

---

#### 1.2.5 方法区/永久代(线程共享 Thread Share)

​		方法区即我们常说的**永久代(Permanent Generation)**，用于存储**被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码**等数据。Hotspot VM把GC分代收集扩展至方法区，即**使用Java堆的永久代来实现方法区**，这样Hotspot的垃圾收集器就可以像管理Java堆一样管理这部分内存，而不必为方法区开发专门的内存管理器(永久代的内存回收的主要目标是针对**常量池的回收**和**类型的卸载**，因此收益一般很小)。

​		<font color="#dd0000">运行时常量池(Runtime Constant Pool)</font>是方法区的一部分。class文件中除了有类的版本、字段、方法、接口的描述等信息外，还有一项信息是常量池(Constant Pool Table)，用于存放编译期生成的各种字面量和符号引用，这部分内容在类加载后存放到方法区的运行时常量池中。Java虚拟机对class文件的每一部分(包括常量池)的格式都有严格的规定，每一个字节用于存储那种数据都必须符合规范上的要求，这样才会被虚拟机认可、装载和执行。

---

### 1.3 JVM运行时内存

​		Java堆从GC的角度还可以细分为：**新生代**(Eden区、From Survivor区和To Survivor区)和**老年代**。

![Java堆从GC的角度细分](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_6.png "Java堆从GC的角度细分")

---

#### 1.3.1  新生代

​		是用来存放新生的对象。一般占据堆的1/3空间。由于频繁创建对象，所以新生代会频繁出发[Minor GC](https://www.cnblogs.com/williamjie/p/9516264.html)进行垃圾回收。[新生代](https://www.cnblogs.com/jswang/p/9056038.html)又分为：Eden区、From Survivor区、To Survivor三个区。

---

##### 1.3.1.1 Eden 区

​		<font color="003399">Java新对象的出生地</font>(如果新创建的对象占用内存很大，则直接分配到老年代)。当Eden区内存不够的时候会触发Minor GC，对新生代区进行一次垃圾回收。

---

##### 1.3.1.2 From Survivor 区

​		上一次GC的幸存者，作为这一次GC的被扫描者。

---

##### 1.3.1.3 To Survivor 区

​		保留了一次Minor GC过程中的幸存者。

---

##### 1.3.1.4 Minor GC 的过程

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

#### 1.3.2 老年代

​		主要存放应用程序中生命周期长的内存对象。

​		老年代的对象比较稳定，所以 Major GC (又称 Full GC) 不会频繁执行。在进行Major GC前一般都先进行了一次Minor GC，使得有新生代对象晋身入老年代，导致老年代空间不够用时才触发。当无法找到足够大的连续空间分配给新创建的较大对象时也会出发一次Major GC 进行垃圾回收腾出空间。

​		Major GC 采用<font color="003399">标记清除算法</font>：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。Major GC的耗时比较长，因为要扫描再回收。Major GC会产生内存碎片，为了减少内存消耗，我们一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候就会抛出OOM(Out of Memory)异常。

---

#### 1.3.3 永久代

​		指内存的永久保存区域，主要存放class和Meta(元数据)的信息，class在被加载的时候被放入永久区域，它和存放实例的区域不同，<font color="003399"> GC不会在主程序运行期对永久区域进行清理。</font>所以这也导致了永久代的区域会随着class的增多而空间不够，最终抛出OOM异常。

---

##### 1.3.3.1 Java 8 与元数据

​		在Java 8中，<font color="003399">永久代已经被移除，被一个称为“元数据区”(元空间)的区域所取代。</font>元空间的本质和永久代类似，元空间与永久代最大的区别在于：<font color="003399">元空间并不在虚拟机中，而是使用本地内存。</font>因此，默认情况下，元空间的大小只受本地内存限制。<font color="003399">类的元数据放入 native memory，字符串池和类的静态变量放入Java堆中</font>，这样可以加载多少类的元数据就不再由[MaxPermSize](https://www.cnblogs.com/mingforyou/p/2378143.html)控制,而是由系统实际可用空间来控制。

---

### 1.4 垃圾回收与算法

![JVM垃圾回收与算法](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_7.png "JVM垃圾回收与算法")

---

#### 1.4.1 如何确定垃圾

##### 1.4.1.1 引用计数法

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

---

##### 1.4.1.2 可达性分析

​		为了解决引用计数法的循环引用问题，Java使用了可达性分析的方法。通过一系列的“GC roots”对象作为起点搜索。<font color="003399">如果在“GC roots”和一个对象之间没有可达路径，则称该对象是不可达的。</font>要注意的是，不可达对象不等价于可回收对象，<font color="003399">不可达对象变为可回收对象至少要经过两次标记过程。</font>两次标记后仍然是不可达对象，则将面临回收。

![可达性分析](https://images2018.cnblogs.com/blog/1368961/201804/1368961-20180406113127652-1645765895.png "可达性分析")

​		由上图可以得知实例1、2、4、6都具有GC roots可达性，即存活对象，不可被GC回收的对象。而实例3、5虽然连通，但是没有与GC roots相连，即GC roots不可达对象，会被GC回收的对象。

---

#### 1.4.2 标记清除算法(Mark-Sweep)

​		该算法是最基础的垃圾回收算法，其分为两个阶段，<font color="003399">标注和清除</font>。标记阶段标记出所有需要回收的对象，清除阶段回收被标记的对象所占用的空间。如图：

![标记清除示意图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_8.png "标记清除示意图")

​		从图中可以发现，该算法最大的问题是内存碎片化严重，后续可能会出现大对象找不到可利用的空间的问题。

---

#### 1.4.3 复制算法(copying)

​		该算法是为了解决Mark-Sweep算法内存碎片化的缺陷而被提出的算法。按内存容量将内存划分为等大小的两块。每次只使用其中的一块，当这一块内存满后将尚存货的对象复制到另一块上去，把已使用的内存清掉，如图：

![复制算法](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_9.png "复制算法")

​		这种算法虽然实现简单，内存效率高，不易产生碎片，但是最大的问题是可用内存被压缩到了原来的一半。且存活对象增多的话，copying算法效率会很低。

---

#### 1.4.4 标记整理算法(Mark-Compact)

​		该算法结合了Mark-Sweep和copying两个算法，为了避免缺陷而提出。标记阶段和Mark-Sweep相同，<font color="003399">标记后不是清理对象，而是将存活对象移向内存的一端，然后清除存活边界外的对象</font>。如图：

![标记整理算法](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_10.png "标记整理算法")

---

#### 1.4.5 分代收集算法

​		分代收集算法是目前大部分JVM所采用的方法，其核心思想是根据对象存活的不同生命周期将内存划分为不同的域，一般情况下将GC堆划分为老生代(Tenured/Old Generation)和新生代(Young Generation)。<font color="003399">老生代的特点是每次垃圾回收时只有少量对象需要被回收(老年代几乎都是Survivor熬过来的，不那么容易死掉，因此Major GC并不会很频繁)，新生代的特点是每次垃圾回收时都有大量垃圾需要被回收(Java对象大多是朝生夕死的特性，所以新生代的Minor GC会非常频繁)</font>。因此可以根据不同区域选择不同的算法。

---

##### 1.4.5.1 新生代与复制算法

​		目前大部分JVM的GC对于新生代都是使用copying算法，因为新生代中每次垃圾回收都要回收大量对象，需要复制的对象比较少。但通常不是按照1 : 1来划分新生代。一般将新生代划分为一块较大的Eden区和两个较小的Survivor区(From Space、To Space)，每次使用Eden区和其中的一块Survivor区，进行回收时将两块区域还存活的对象复制到另一个Survivor区中。

![新生代区域划分](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_11.png "新生代区域划分")

---

##### 1.4.5.2 老年代与标记整理算法

​		反观老年代因为每次只回收少量的对象，使用Mark-Sweep算法会使内存碎片化严重。所以采用Mark-Compact算法

1. JVM提到过的处于方法区的永生代(Permanent Generation)，它用来存储class类、常量、方法描述等。对永生代的回收主要包括废弃常量和无用的类。
2. 对象的内存分配主要在新生代的Eden Space和From Space(Survivor在GC前存放对象的那一块)，少数情况会直接分配到老年代。
3. 当新生代的Eden Space和From Space空间不足时就会触发一次GC，进行GC后，Eden Space和From Space的存活对象被复制到To Space，清理Eden Space和From Space后会将两个Survivor区互换，To Space作为下次GC的From Space
4. 如果To Space无法足够存储某个对象，则会将这个对象存储到老年代。
5. 当对象在Survivor区躲过一次Minor GC后年龄会+1，<font color="003399">默认15岁会被移到老年代中</font>

---

### 1.5 Java中的四种引用类型

#### 1.5.1 强引用

​		在Java中最常见的就是强引用，如果一个对象具有强引用，它就不会被GC回收。即使当前<font color="#dd0000">内存空间不足</font>，JVM也不会回收它，而是抛出OOM错误(OutOfMemoryError)。因此强引用是造成内存泄露的主要原因之一。如果想中断强引用和某个对象之间的关联，可以显式地将引用赋值为null，这样一来，JVM在合适的时间就会回收该对象。

```java
String str = "hello";	//强引用
str = null;				//取消强引用
```

---

#### 1.5.2 软引用

​		<font color="003399">软引用需要用 SoftReference 类来实现</font>，对于只有软引用的对象来说，当系统内存足够时它不会被回收，不足时会被回收。

```java
SoftReference<String> sr = new SoftReference("hello");
```

---

#### 1.5.3 弱引用

​		弱引用需要用 WeakReference 类来实现，具有弱引用的对象拥有的生命周期更短暂。因为当 JVM 进行GC，一旦发现弱引用对象，无论当前内存空间是否充足，都会将弱引用回收。不过由于垃圾回收器是一个优先级较低的线程，所以并不一定能迅速发现弱引用对象。

```java
WeakReference<String> wr = new WeakReference("hello");
```

---

#### 1.5.4 虚引用

​		虚引用顾名思义，就是形同虚设，如果一个对象仅有虚引用，那么它相当于没有引用，在任何时候都可能被GC回收。

​		虚引用需要 PhantomReference 类来实现，且必须和引用队列关联使用。<font color="003399">虚引用的作用主要是跟踪对象被垃圾回收的状态</font>。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

```java
ReferenceQueue<String> queue = new ReferenceQueue<String>();
PhantomReference<String> pr = new PhantomReference<String>(new String("hello"), queue);
```

---

### 1.6 GC 分代收集算法 与 分区收集算法

#### 1.6.1 分代收集算法

​		详见 2.4.5 分代收集算法

---

#### 1.6.2 分区收集算法

​		<font color="003399">[分区收集算法](https://cloud.tencent.com/developer/article/1390547)将整个堆空间划分为连续的不同小区间，每个小区间独立使用</font>，独立回收。这样做的好处是可以控制一次回收多少个小区间，根据目标停顿时间，合理地回收若干个小区间(而不是整个堆)，从而可以减少一次GC所产生的停顿。

![分区收集算法](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_12.png "分区收集算法")

---

### 1.7 GC 垃圾收集器

​		Java堆内存被划分为新生代和老年代两部分，新生代主要使用复制和Mark-Sweep算法；老年代主要使用Mark-Compact算法。因此Java虚拟机中针对新生代和老年代分别提供了多种不同的垃圾收集器，JDK1.8中 Sun Hotspot虚拟机的垃圾收集器如下：

![JDK1.8 GC垃圾收集器](https://img-blog.csdn.net/20180813194627489?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5OTgyNTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70 "JDK1.8 GC垃圾收集器")

---

#### 1.7.1 Serial 垃圾收集器(单线程 复制算法)

​		Serial 是最基本的垃圾收集器，使用复制算法。它曾经是JDK1.3.1之前新生代唯一的垃圾收集器。Serial是一个单线程的收集器，并且在进行垃圾收集时，必须暂停所有工作线程，直到垃圾收集结束(**GC停顿**)。

![Serial收集器运行示意图](https://img-blog.csdn.net/2018081320010740?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5OTgyNTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70 "Serial收集器运行示意图")

​		Serial垃圾收集器虽然在垃圾收集的过程中需要暂停所有其他的工作线程，但是它简单高效，对于限定单个CPU环境来说，Serial收集器没有线程交互开销，可以获得最高的单线程收集效率。在用户的桌面应用场景中，可用内存一般不大，可以在较短时间内完成垃圾收集(10ms ~ 200ms)，只要不频繁发生是可以接受的。

​		所以Serial垃圾收集器依然是Hotspot在Client模式下默认的新生代垃圾收集器。

---

#### 1.7.2 ParNew 垃圾收集器(Serial + 多线程)

​		ParNew垃圾收集器其实是Serial收集器的多线程版本，也是使用复制算法。除了使用多线程进行垃圾收集之外，其余的行为和Serial收集器完全一样，ParNew垃圾收集器在垃圾收集的过程中同样也需要暂停所有其他工作线程。

​		ParNew收集器默认开启和CPU数目相同的线程数。

​		参数控制：

```java
-XX:+UseConcMarkSweepGC:	//指定使用CMS后，会默认使用ParNew作为新生代收集器;
-XX:+UseParNewGC:			//强制指定使用ParNew;
-XX:ParallelGCThreads:		//指定垃圾收集的线程数量，ParNew默认开启的收集线程与CPU的数量相同;
```

![ParNew 垃圾收集器](https://img-blog.csdn.net/2018081320013053?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5OTgyNTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70 "ParNew 垃圾收集器")

​		<font color="003399">ParNew收集器是许多运行在server模式下的虚拟机中首选的新生代收集器</font>。一个重要原因是，只有ParNew和Serial收集器能和CMS收集器共同工作。无法与JDK1.4中存在的新生代收集器Parallel Scavenge配合工作，所以在JDK1.5中使用CMS来收集老年代的时候，新生代只能选择ParNew和Serial。

---

#### 1.7.3 Parallel Scavenge 收集器(多线程 复制算法)

​		Parallel Scavenge 收集器也是一个新生代垃圾收集器，同样使用复制算法，也是一个多线程垃圾收集器。它重点关注的是程序达到一个可控制的吞吐量(Throughout)：
$$
吞吐量(Throughout) = \frac{运行用户代码时间}{运行用户代码时间 + 垃圾收集时间}
$$
​		高吞吐量可以最高效率地利用CPU时间，尽快地完成程序的运算任务，主要适用于在后台运算而 不需要太多交互的任务。自适应调节策略也是Parallel Scavenge收集器和ParNew收集器一个重要的区别。

​		参数控制：

```java
-XX:GCTimeRatio:			//控制吞吐量
-XX:MaxGCPauseMillus:		//控制垃圾停顿时间
-XX:UseAdaptiveSizePolicy:	//虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大吞吐量(GC自使用的调节策略)。
```

---

#### 1.7.4 Serial Old 收集器(单线程 Mark-Compact算法)

​		Serial Old 是 Serial 垃圾收集器地老年代版本，它与Serial一样是一个单线程收集器，使用标记-整理算法。<font color="003399">该收集器主要也是运行在Client模式下的Java虚拟机默认的老年代垃圾收集器</font>。如果在Server模式，则该收集器主要有两个用途：

1. 在JDK1.5以及之前的版本中与新生代的Parallel Scavenge收集器搭配使用。
2. 作为老年代中使用CMS收集器的后备垃圾收集方案。在并发收集发生 Concurrent Mode Failure时使用。工作流程图如下：

![Serial/Serial Old搭配垃圾收集过程图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_13.png "Serial/Serial Old搭配垃圾收集过程图")

​		新生代 Parallel Scavenge 收集器与 ParNew收集器工作原理类似，都是多线程的收集器，都使用的是复制算法，在垃圾收集的过程中都需要暂停所有的工作线程。新生代 Parallel Scavenge/ParNew 与老年代 Serial Old 垃圾收集过程如图：

![Parallel Scavenge/ParNew 配合 Serial Old垃圾收集过程图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_14.png "Parallel Scavenge/ParNew 配合 Serial Old垃圾收集过程图")

---

#### 1.7.5 Parallel Old 收集器(多线程 Mark-Compact算法)

​		Parallel Old 收集器是 Parallel Scavenge 的老年代版本，使用多线程的标记-整理算法，在JDK1.6之后才提供。

​		在此之前Parallel Scavenge 只有 Serial Old可供选择，只能保证新生代的吞吐量优先，无法保证整体的吞吐量，所以就诞生了此收集器。至此吞吐量优先收集器就有了比较名副其实的应用组合，在注重吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge收集器加上Parallel Old收集器。

![Parallel Scavenge/Parallel Old 收集器](https://img-blog.csdnimg.cn/20200411222246385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NpbW9uXzA5MDEwODE3,size_16,color_FFFFFF,t_70 "Parallel Scavenge/Parallel Old 收集器")

---

#### 1.7.6 CMS 收集器(多线程 Mark-Sweep算法)

​		Concurrent Mark Sweep(CMS)收集器是一种老年代垃圾收集器，其最主要的目标是<font color ="003399">获取最短垃圾回收时间</font>，与Parallel Old 使用的 Mark-Compact 算法不同，它使用多线程 Mark- Sweep 算法。目前很大一部分Java应用集中在互联网站或者B/S结构的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿更短。

​		CMS 的工作机制相比其他垃圾收集器来说更复杂，整个过程分为以下4个阶段：

##### 1.7.6.1 初始标记(CMS initial mark)

​		只是标记一下GC roots能直接关联的对象，速度很快，但是仍然需要暂停所有的工作线程。

##### 1.7.6.2 并发标记(CMS concurrent mark)

​		进行GC roots Tracing 可达性分析过程。在整个过程中耗时最长。

##### 1.7.6.3 重新标记(CMS remark)

​		为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这一阶段的停顿时间比初始标记稍长，但远比并发标记时间短，仍然需要暂停所有工作进程。

##### 1.7.6.4 并发清除(CMS concurrent sweep)

​		清除 GC roots 不可达对象，和用户线程一起工作，不需要暂停工作线程。由于耗时最长的并发标记和并发清除过程中，GC线程可以和用户线程一起并发工作，<font color="#dd000">所以总体上看CMS收集器的内存回收和用户线程是一起并发地执行</font>。

![CMS 收集器](https://img-blog.csdnimg.cn/20200411225638222.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NpbW9uXzA5MDEwODE3,size_16,color_FFFFFF,t_70 "CMS 收集器")

​		**谈谈CMS收集器地优缺点**：

​		优点：

​		<font color="#dd000">并发收集、低停顿</font>。因此也被称为并发低停顿收集器(Concurrent Low Pause Collector)。

​		缺点：

- <font color="#dd000">**对CPU资源非常敏感**</font>。在并发阶段，它虽然不会导致用户线程停顿，但因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。<font color="#dd000">CMS默认启动地回收线程数为(CPU数 + 3) / 4</font>，当CPU在4个以上时，并发回收时垃圾收集线程不少于25%的CPU资源，但是当CPU不足4个时，CMS对用户程序的影响可能变得很大。
- <font color="#dd000">**无法处理浮动垃圾(Floating Garbage)，可能出现Concurrent Mode Failure失败导致另一次Full GC产生**</font>。由于CMS并发清理阶段用户线程还在运行，伴随程序运行自然就还会有新的垃圾不断产生。<font color="#dd000">这部分垃圾出现在标记过程之后，CMS无法在当次收集中处理掉它们，只好留待下一次GC时再清理，这一部分垃圾称为浮动垃圾。</font>由于垃圾收集阶段用户线程还需要运行，就需要预留足够的内存空间给用户线程使用，在JDK1.5默认设置下CMS收集器当老年代使用了68%的空间后就被激活，如果应用的老年代内存增长不是太快可以通过-XX:CMSInitiatingOccupancyFraction的值来提高触发百分比，以便降低内存回收次数从而获取更好的性能，在JDK1.6中，<font color="#dd000">CMS收集器的启动阈值已经提升至92%。要是CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了</font>。
- <font color="#dd000">**Mark-Sweep算法地通病：导致内存碎片化严重**</font>。空间碎片过多往往会出现老年代还有很大空间，但是无法找到足够大的连续空间分配大对象。不得不提前触发一次Full GC。CMS提供-XX：+UseCMSCompactAtFullCollection开关参数（默认开启），用于CMS收集器顶不住要进行Full GC时开启内存碎片的合并整理，但是过程无法并发，停顿时间不得不变长。

---

#### 1.7.7 G1 收集器



















