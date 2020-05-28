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

​		详见 1.4.5 分代收集算法

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

​		**谈谈CMS收集器的优缺点**：

​		优点：

​		<font color="#dd000">并发收集、低停顿</font>。因此也被称为并发低停顿收集器(Concurrent Low Pause Collector)。

​		缺点：

- <font color="#dd000">**对CPU资源非常敏感**</font>。在并发阶段，它虽然不会导致用户线程停顿，但因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。<font color="#dd000">CMS默认启动地回收线程数为(CPU数 + 3) / 4</font>，当CPU在4个以上时，并发回收时垃圾收集线程不少于25%的CPU资源，但是当CPU不足4个时，CMS对用户程序的影响可能变得很大。
- <font color="#dd000">**无法处理浮动垃圾(Floating Garbage)，可能出现Concurrent Mode Failure失败导致另一次Full GC产生**</font>。由于CMS并发清理阶段用户线程还在运行，伴随程序运行自然就还会有新的垃圾不断产生。<font color="#dd000">这部分垃圾出现在标记过程之后，CMS无法在当次收集中处理掉它们，只好留待下一次GC时再清理，这一部分垃圾称为浮动垃圾。</font>由于垃圾收集阶段用户线程还需要运行，就需要预留足够的内存空间给用户线程使用，在JDK1.5默认设置下CMS收集器当老年代使用了68%的空间后就被激活，如果应用的老年代内存增长不是太快可以通过-XX:CMSInitiatingOccupancyFraction的值来提高触发百分比，以便降低内存回收次数从而获取更好的性能，在JDK1.6中，<font color="#dd000">CMS收集器的启动阈值已经提升至92%。要是CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了</font>。
- <font color="#dd000">**Mark-Sweep算法的通病：导致内存碎片化严重**</font>。空间碎片过多往往会出现老年代还有很大空间，但是无法找到足够大的连续空间分配大对象。不得不提前触发一次Full GC。CMS提供-XX：+UseCMSCompactAtFullCollection开关参数（默认开启），用于CMS收集器顶不住要进行Full GC时开启内存碎片的合并整理，但是过程无法并发，停顿时间不得不变长。

---

#### 1.7.7 G1 收集器

​		G1(Garbage-First) 收集器是目前收集器技术发展最前沿的成果之一。它是一款<font color="#dd000">面向服务端应用的垃圾收集器</font>。使用G1收集器时，Java堆内存布局就与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域(Region)，虽然还保留新生代和老年代的概念，但新生代和老年代不再是物理隔离了，而都是一部分Region(不需要连续)的集合。在G1中，堆被划分成 许多个连续的区域(region)。每个区域大小相等，在1M~32M之间。JVM最多支持2000个区域，可推算G1能支持的最大内存为2000*32M=62.5G。区域(region)的大小在JVM初始化的时候决定，也可以用-XX:G1HeapReginSize设置。

![G1收集器对内存的划分](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_15.png "G1收集器对内存的划分")

​		每块区域既有可能是Old区，也有可能是Young区，因此不需要一次就对整个老年代/新生代回收。而是<font color="003399">而是当线程并发寻找可回收对象时，有些区块包含可回收的对象比其他区块多很多虽然在清理这些区块时G1仍然需要暂停应用线程, 但可以用相对较少的时间优先回收垃圾较多的Region(这也是G1命名的来源)。</font>这种方式保证了G1可以在有限的时间内获取尽可能高的收集效率。G1具备如下特点：

- 并行与并发：G1能充分利用多CPU、多线程环境下的硬件优势，使用多个CPU(或核心)来缩短stop-the-world(暂停所有工作线程线程)停顿时间。G1收集器仍然可以通过并发的方式让Java程序继续执行。
- 分代收集：虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了粉黛的概念。它能采用不同方式去处理新创建的对象和已经存活了一段时间的对象，以获取更好的收集效果。
- 空间整合：与CMS的 Mark-Compact 算法不同，<font color="#dd000">G1从整体来看是基于Mark-Compact算法实现的收集器，从局部(两个Region之间)看是基于Copying算法实现的</font>，这两种算法都意味着G1运行期间不会产生内存空间碎片，收集后能提供规整的可用内存。这个特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前出发下一次GC。
- 可预测的停顿：G1建立可预测的停顿时间模型，能让使用者指明在一个长度为M ms的时间片段内，消耗在垃圾收集上的时间不超过 N ms。

##### 1.7.7.1 新生代收集

​		G1的新生代收集和ParNew类似，存活的对象被转移到一个或多个 Survivor Regions。如果存活时间达到阈值，这部分对象就会被提升到老年代。

<img src="http://www.sico-technology.cn:81/images/java_note/jvm/jvm_16.png" alt="G1新生代收集" align="left" style="zoom: 65%"><img src="http://www.sico-technology.cn:81/images/java_note/jvm/jvm_17.png" alt="G1新生代收集" align="" style="zoom: 53%">

​		G1的新生代收集特点如下：

- 一整块堆内存被分为多个Regions
- 存活对象被拷贝到Survivor区或老年代
- 年轻代内存由一组不连续的heap区组成，这种方法使得可以动态调整各代区域尺寸
- Young GC会有 stop-the-world事件
- 多线程并发GC

##### 1.7.7.2 老年代收集

​		G1老年代收集会执行以下阶段：

>注：以下有些阶段也是年轻代垃圾收集的一部分

| num  | Phase                                                 | Description                                                  |
| :--: | :---------------------------------------------------- | ------------------------------------------------------------ |
|  1   | 初始标记 (Initial Mark: Stop the World Event)         | 在G1中, 该操作附着一次年轻代GC, 以标记Survivor中有可能引用到老年代对象的Regions. |
|  2   | 扫描根区域 (Root Region Scanning: 与应用程序并发执行) | 扫描Survivor中能够引用到老年代的references. 但必须在Minor GC触发前执行完. |
|  3   | 并发标记 (Concurrent Marking : 与应用程序并发执行)    | 在整个堆中查找存活对象, 但该阶段可能会被Minor GC中断.        |
|  4   | 重新标记 (Remark : Stop the World Event)              | 完成堆内存中存活对象的标记. 使用snapshot-at-the-beginning(SATB, 起始快照)算法, 比CMS所用算法要快得多(空Region直接被移除并回收, 并计算所有区域的活跃度). |
|  5   | 清理 (Cleanup : Stop the World Event and Concurrent)  | 见下 5-1、2、3                                               |
| 5-1  | (Stop the world)                                      | 在含有存活对象和完全空闲的区域上进行统计                     |
| 5-2  | (Stop the world)                                      | 擦除Remembered Sets.                                         |
| 5-3  | (Concurrent)                                          | 重置空regions并将他们返还给空闲列表(free list)               |
| (*)  | Copying/Cleanup (Stop the World Event)                | 选择”活跃度”最低的区域(这些区域可以最快的完成回收). 拷贝/转移存活的对象到新的尚未使用的regions. 该阶段会被记录在gc-log内(只发生年轻代[GC pause (young)], 与老年代一起执行则被记录为[GC Pause (mixed)]. |

​		G1老年代收集特点如下：

- 并发标记阶段(num 3)：在与应用程序并发执行的过程中会计算活跃度信息。这些活跃度信息标识出哪些Regions适合在stop-the-world期间回收。
- 再次标记阶段(num 4) ：使用Snapshot-at-the-Beginning(SATB)算法比CMS快得多。空Region直接被回收。
- 拷贝清理阶段(Copying/Cleanup - Phase)：年轻代与老年代同时回收。老年代内存回收会基于它的活跃度信息。

##### 1.7.7.3 G1 SATB 和 CMS Incremental Update算法的理解

###### 1.7.7.3.1 什么是着色标记

​		CMS GC 和 G1 GC算法都是通过对GC roots进行遍历，并进行三颜色标记，具体标记算法如下：

- 黑色(black)：节点被遍历完成，而且子节点都遍历完成
- 灰色(gray)：当前正在遍历的节点，而且子节点还没有遍历
- 白色(white)：还没有遍历到的节点，即灰色节点的子节点

###### 1.7.7.3.2 并行GC面对的共同问题

​		CMS GC 和 G1 GC 都有并行执行的阶段。既然有并行，那就有可能在GC过程中标记过的对象引用关系发生改变，比如一个white节点的引用由gray变为black，那么该white节点变为black节点的子节点，因此而被漏掉，导致被错误回收。

​		这就是为什么 CMS 和 G1 都有Remark阶段，都需要对被修改的card重新扫描。

​		那CMS GC和G1 GC各自是如何解决这个问题的呢？这里就要了解CMS Incremental Update 算法和 G1 SATB算法了。

​		我们先描述一下问题场景，并行 GC 在什么情况下会出现漏掉活的对象，根据三色扫描算法，如果有下面两种情况发生，则会出现漏扫描的场景：

1. 把一个white对象的引用存到black对象的字段里，如果这个情况发生，因为标记为black的对象认为是扫描完成的，不会再对它进行扫描
2. 某个白对象失去了所有能从灰对象到达它的引用路径(直接或间接)

如图：

![GC roots](https://upload-images.jianshu.io/upload_images/5419521-bf917d0ad34173ce.png?imageMogr2/auto-orient/strip|imageView2/2/w/826/format/webp "GC roots")

​		如上图对象D，引用存到了black对象A上，同时切断了gray对象B对其的引用，导致对象D被漏扫描了。解决这个问题可以从两个角度入手，指向D的这个引用从源B到目的地址A，所以分别从源和目标的角度来解决就提出了两种算法：

​		**SATB(Snapshot-at-beginning)**：SATB算法认为初始标记的都认为是活的对象，引用B到D改为B到C时，通过<font color="003399">write barrier写屏蔽技术</font>，会把B到D的引用推到GC遍历的执行堆栈上，保证还可以遍历到D对象，相对于D来说引用从B到A，SATB是从引用源入手解决的。SATB即认为初始时所有能遍历到的对象都是需要标记的如果把B = null，那么D就变成垃圾了，SATB算法却依然把D标为black，导致D在本轮GC不能被回收，变成浮动垃圾。

​		**Incremental Update**： Incremental Update算法判断如果一个white对象由一个black对象引用，即white对象是一个black对象的目的，如上图B的目的D变为A的目的，发现这种情况时，也是通过write barrier写屏蔽技术，把黑色的对象重新标记为灰色，让collector 重新来扫描，活着通过mod-union table 来标记。

> mod-union table：https://www.jianshu.com/p/7be320306ed1

###### 1.7.7.3.3 Incremental Update 和 SATB 的区别

​		通过上面的分析，我们知道SATB write barrier 是认为开始标记那一刻认为都是活的，所以有可能有些已经是垃圾的对象就会也被扫描，导致 SATB相对  Incremental Update  会更多的开销，G1 GC 扫描的都是选定的固定个数的Region，所以这个开销应该可控，但是而且浮动垃圾也更多。(参考:[网页链接](https://www.jianshu.com/p/8d37a07277e0))

##### 1.7.7.4 G1 收集器总结

​		上面几个步骤的运作过程和CMS有很多相似之处。初始标记阶段仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS的值，让下一个阶段用户程序并发运行时，能在正确可用的Region中创建新对象，这一阶段需要停顿线程，但是耗时很短，并发标记阶段是从GC Root开始对堆中对象进行可达性分析，找出存活的对象，这阶段时耗时较长，但可与用户程序并发执行。而最终标记阶段则是为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这一阶段需要停顿线程，但是可并行执行。最后在筛选回收阶段首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划。

---

### 1.8 Java IO/NIO/AIO

 #### 1.8.1 IO Model

- Blocking IO - 阻塞 IO 模型
- Non-Blocking IO -  非阻塞 IO 模型
- IO multiplexing - 多路复用 IO 模型
- Signal driven IO - 信号驱动 IO 模型
- Asynchronous IO - 异步 IO 模型

##### 1.8.1.1 Blocking IO

​		Blocking IO 是我们在一开始学习Java都使用过的BIO编程方式，在JDK1.4之前，NIO没有出， Java IO 编程只有这一个方式。一个典型的读操作流程如下：

![Blocking IO](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_18.png "Blocking IO")

1. 程序调用<font color="#dd000">`socket.read()`</font>，这个方法会调用<font color="#dd000">`socket.read()`</font>方法，所以最终是由OS(Operating System)执行read
2. OS得到read指令，命令网卡读取数据
3. 网卡读取数据完成后，将数据传递给内核
4. 内核把读取的数据拷贝到用户空间
5. 程序解除阻塞，完成read函数

> 1. <font color="#dd000">`socket.read()`</font>会调用<font color="#dd000">`socket.read()`</font>,而Java中的native方法会调用底层的dll，而dll是C/C++编写的，图中的<font color="#dd000">`recvfrom`</font>其实是C语言socket编程中的一个方法。所以其实我们在Java中调用<font color="#dd000">`socket.read()`</font>最后也会调用到图中的<font color="#dd000">`recvfrom`</font>方法。
> 2. 应用程序想要读取数据就会调用<font color="#dd000">`recvfrom`</font>，而<font color="#dd000">`recvfrom`</font>会通知OS来执行，OS就会判断**数据包是否准备好**(比如判断是否收到了一个完整的UDP报文，如果收到UDP报文不完整，那么就继续等待)。当数据包准备好了之后，OS就会**将数据从内核空间拷贝到用户空间**(因为我们的用户程序只能获取用户空间的内存，无法直接获取内核空间的内存)。拷贝完成之后<font color="#dd000">`socket.read()`</font>就会解除阻塞，并得到read的结果。

​		在Blocking IO 中我们称作的阻塞也就是阻塞在两个地方：1. OS等待数据报准备好。2. 将数据从内核空间拷贝到用户空间。

##### 1.8.1.2 Non-Blocking IO

​		当我们new了一个socket后我们可以设置其为非阻塞的。例如：

```java
// 初始化一个 serverSocketChannel
serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.bind(new InetSocketAddress(8000));
// 设置serverSocketChannel为非阻塞模式
// 即 accept()会立即得到返回
serverSocketChannel.configureBlocking(false);
```

> ```java
> ServerSocketChannel
> SocketChannel
> FileChannel
> DatagramChannel
> //只有FlieChannel无法设置成非阻塞模式，其他Channel都可以设置为非阻塞模式
> ```

​		当对一个non-blocking socket执行读操作时，流程如图：

![Non-Blocking IO](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_19.png "Non-Blocking IO")

​		从图中可以看出，当socket设置为非阻塞后，<font color="#dd000">`socket.read()`</font>会立即得到一个返回结果(success or failed)。我们可以在失败时做一些其他事情，但事实上这种方式也是低效的，因为不得不使用轮询方法去一直问OS："我的数据报好了没啊？"。

​		Non-Blocking IO 不会在<font color="#dd000">`recvfrom`</font>也就是<font color="#dd000">`socket.read()`</font>的时候阻塞，但是还是会在<font color="#dd000">**将数据从内核空间拷贝到用户空间**</font>阻塞，Non-Blocking IO还是会阻塞的。

##### 1.8.1.3 IO Multiplexing

​		IO multiplexing 即 IO多路复用，传统情况下client与server通信需要3个socket(client的socket、server监听client连接的serversocket，还有一个server与client通信的socket)，而在 IO多路复用中，client与server通信需要的不是socket，而是3个channel。通过channel可以完成与socket一样的操作，channel的底层还是使用的socket进行通信，但是多个channel只对应一个socket(可能不只是一个，但是socket的数量一定少于channel数量)，这样仅仅通过少量的socket就可以完成更多的连接，提高了client容量。

![IO multiplexing](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_20.png "IO multiplexing")

​		不同操作系统有不同的实现：

- Widows: selector

- Linux: epoll

- Mac: kqueue

  ​	其中epoll，kqueue比selector更为高效，这是因为他们监听方式的不同。selector的监听是通过轮询FD_SETSIZE来问每一个`socket`：“你改变了吗？”，假若监听到时间，那么selector就会调用相应的时间处理器进行处理。但是epoll与kqueue不同，他们把socket与事件绑定在一起，当监听到socket变化时，立即可以调用相应的处理。

  ​	selector，epoll，kqueue都属于Reactor IO设计。关于 Reactor与Proactor，可以看：[IO 模式 Reactor与Proactor](https://www.jianshu.com/p/46956e779ad4)

##### 1.8.1.4 Signal driven IO

​		Signal driven IO 是指：当一个进程执行IO操作时，内核马上返回，进程继续运行。当刚才指定的IO操作完成后（或出错），通过信号通知进程。

![Signal driven IO](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_21.png "Signal driven IO")

​		Signal-Driven IO for Sockets分为三个步骤：

1. 为SIGIO建立 signal handler
2. 必须设定socket owner，一般使用fcntl设置F_SETOWN
3. 设置socket允许Signal-driven IO，通过fcntl设置F_SETFL，用来开启O_ASYNC flag(也可用FIOASYNC ioctl进行设置)

​		SIGIO with UDP Sockets： 1. datagram到达。2. asynchronous error产生。当捕获消息后，可调用<font color="#dd000">`recvfrom`</font>读取数据或error。

​		Signal driven IO对TCP没什么用，因为产生的太过频繁，而且不能告诉我们发生了什么。

##### 1.8.1.5 Asynchronous IO

​		Asynchronous IO 异步IO流程如图：

![Asynchronous IO](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_22.png "Asynchronous IO")

​		Asynchronous IO调用中是真正的无阻塞，其他IO model中多少会有点阻塞。程序发起read操作之后，立刻就可以开始去做其它的事。而在内核角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

##### 1.8.1.6 各种IO对比

	1. Blocking IO 与 Non-Blocking IO 区别 ?
	
	阻塞或非阻塞只涉及程序和OS，Blocking IO 会一直block程序知道OS返回，而Non-Block IO在OS内核在准备数据包的情况下会立即得到返回。
	
	2. Asynchronous IO 与 Synchronous IO ?
	
	只要有block(阻塞)就是同步IO，完全没有block则是异步IO。所以我们之前所说的 Blocking IO、Non-Blocking IO、IO Multiplexing 都是 Synchronous IO

![IO 对比](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_23.png "IO 对比")

​		有A，B，C，D四个人在钓鱼：

1. A用的是最老式的鱼竿，所以呢，得一直守着，等到鱼上钩了再拉杆；
2. B的鱼竿有个功能，能够显示是否有鱼上钩，所以呢，B就和旁边的MM聊天，隔会再看看有没有鱼上钩，有的话就迅速拉杆；
3. C用的鱼竿和B差不多，但他想了一个好办法，就是同时放好几根鱼竿，然后守在旁边，一旦有显示说鱼上钩了，它就将对应的鱼竿拉起来；
4. D是个有钱人，干脆雇了一个人帮他钓鱼，一旦那个人把鱼钓上来了，就给D发个短信。

​		参考博客：[IO - 同步，异步，阻塞，非阻塞 ](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fhistoryasamirror%2Farticle%2Fdetails%2F5778378)

#### 1.8.2 Java IO

##### 1.8.2.1 IO的概念和作用

​		Java IO 也称为IO流，IO = 流，它的核心就是对文件的操作，对于字节 、字符类型的输入和输出流。IO是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流，**流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。**

​		Java IO流操作有关的类或接口：

| 类               | 说明           |
| ---------------- | -------------- |
| File             | 文件类         |
| RandomAccessFile | 随机读取文件类 |
| InputStream      | 字节输入流     |
| OutputStream     | 字节输出流     |
| Reader           | 字符输入流     |
| Writer           | 字符输出流     |

​		Java IO类图结构：

![Java IO类图结构](https://pic002.cnblogs.com/images/2012/384764/2012031413373126.jpg "Java IO类图结构")

##### 1.8.2.2 IO流的分类

1. 根据处理数据类型的不同分为：字符流和字节流
2. 根据数据流向不同分为：输入流和输出流

**字符流和字节流**

​		字符流的由来：因为根据编码的不同，而有了对字符进行高校操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。字节流和字符流的区别：

- 读写单位不同：字节流以字节(8 bit)为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。
- 处理对象不同：字节流能处理所有类型的数据（如 .jpg、.flv 等），而字符流只能处理字符类型的数据。

**输入流和输出流**

对输入流只能进行读操作，对输出流只能进行写操作，程序中需要根据待传输数据的不同特性而使用不同的流。

##### 1.8.2.3 Java IO流对象

###### 1.8.2.3.1 输入字节流 InputStream

1. InputStream是所有输入字节流的父类，它是一个抽象类。
2. ByteArrayInputStream, StringBufferInputStream, FileInputStream是三种基本的介质流，它们分别从 Byte数组、StringBuffer、和本地文件中读取数据，PipedInputStream是从与其他线程共用的管道中读取数据。
3. ObjectInputStream和所有FileInputStream 的子类都是装饰流(装饰器模式的主角)。

###### 1.8.2.3.2 输出字节流 OutputStream

1. OutputStream是所有输出字节流的父类，它是一个抽象类。
2. ByteArrayOutputStream, FIleOutputStream是两种基本的介质，它们分别向Byte 数组，和本地文件中写入数据。PipedOutputStream是从与其他线程共用的管道中写入数据。
3. ObjectOutputStream和所有FileOutputStream的子类都是装饰流。

**字节流的输入与输出对应图**

![字节流输入输出对应图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_24.png "字节流输入输出对应图")

​		图中蓝色为主要对应部分，红色为不对应部分，黑色的虚线部分代表这些流一般需要搭配使用。从上图中可以看出Java IO中的字节流是非常对称的，下面介绍一些红色部分的不对称的几个类：

1. LineNumberInputStream 主要完成从流中读取数据时，会得到相应的行号，至于什么时候分行、在哪里分行是由该类主动确定的，并不是在原始中有这样一个行号。在输出部分没有对应的部分，我们完全可以自己建立一个LineNumberOutputStream，在最初写入时会有一个基准的行号，以后每次遇到换行时会在下一行添加一个行号，看起来也是可以的。好像更不入流了。
2. PushbackInputStream 的功能是查看最后一个字节，不满意就放入缓冲区。主要用在编译器的语法、词法分析部分。输出部分的BufferedOutputStream 几乎实现相近的功能。
3. StringBufferInputStream 已经被Deprecated，本身就不应该出现在InputStream 部分，主要因为String 应该属于字符流的范围。已经被废弃了，当然输出部分也没有必要需要它了！还允许它存在只是为了保持版本的向下兼容而已。
4. SequenceInputStream 可以认为是一个工具类，将两个或者多个输入流当成一个输入流依次读取。完全可以从IO 包中去除，还完全不影响IO 包的结构，却让其更“纯洁”――纯洁的Decorator 模式。
5. PrintStream 也可以认为是一个辅助工具。主要可以向其他输出流，或者FileInputStream 写入数据，本身内部实现还是带缓冲的。本质上是对其它流的综合运用的一个工具而已。一样可以踢出IO 包！System.out 和System.out 就是PrintStream 的实例。

###### 1.8.2.3.3 字符输入流 Reader

1. Reader是所有的输入字符流的父类，它是一个抽象类。
2. CharArrayReader, StringReader是两种基本的介质流，它们分别从Char数组、Sting中读取数据。PipedInputReader 是从与其他线程共用的管道中读取数据。
3. BufferedReader 很明显是一个装饰器，它和其子类复制装饰其他Reader对象。
4. FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，回增加一个行号。
5. InputStreamReader 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader 可以说是一个达到此功能常用的工具类，在其源代码中明显使用了将FileInputStream转变为Reader的方法。我们可以从这个类中得到一定的技巧。Reader中各个类的用途和使用方法基本和InputStream中的类使用一致。

###### 1.8.2.3.4 字符输出流 Writer

1. Writer 是所有输出字符流的父类，它是一个抽象类。
2. CharArrayWriter, StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。PipedInputWriter是从与其他线程共用的管道中读取数据。
3. BufferedWriter很明显是一个装饰器，他和其子类复制装饰其他Reader对象。
4. FilterWriter和PrintStream极其类似，功能和使用也非常相似。
5. OutputStreamWriter 是OutputStream 到Writer 转换到桥梁，它的子类FileWriter其实就是一个实现此功能的具体类(具体可以研究一下SourceCode)。功能和使用OutputStream极其类似。

**字符流的输入与输出对应图**

![字符流输入输出对应图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_25.png "字符流输入输出对应图")

###### 1.8.2.3.5 字符流与字节流的转换

转换流的特点：

1. 它是字符流和字节流之间的桥梁。
2. 可对读取到的字节流数据经过指定编码转换成字符。
3. 可对读取到的字读数据经过指定编码转换成字节。

何时使用转换流？：

1. 当字节和字符之间有转换动作时。
2. 流操作的数据需要编码或解码时。

具体的对象体现：

1. InputStreamReader：字节到字符的桥梁。
2. OutputStreamWriter：字符到字节的桥梁。

这两个流对象是字符体系中的成员，它们有转换的作用，本身又是字符流，所以在构造的时候需要传入字节流对象进来。

###### 1.8.2.3.6 File类

​		File类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。File类保存文件或目录的各种数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名、判断文件是否存在、获取当前目录中的文件列表、创建、删除文件和目录等方法。

###### 1.8.2.3.7 RandomAccessFile类

​		该对象不是流体系中的一员，其封装了字节流，同时还封装了一个缓冲区(字符数组)，通过内部的指针来操作字符数组中的数据。该对象特点：

1. 该对象只能操作文件，所以构造函数接受两种数据类型的参数：字符串文件路径、File对象
2. 该对象既可以对文件进行读操作也能进行写操作，在进行对象实例化时可指定操作模式(r, rw)。

注意：**该对象在实例化时，如果要操作的文件不存在，会自动创建；如果文件存在，写数据未指定位置，会从头开始写，即覆盖原有的内容。**可以用于多线程下载或多个线程同时写数据到文件。---参考文章：[Java IO学习整理](https://zhuanlan.zhihu.com/p/25418336)

---

#### 1.8.3 Java NIO

##### 1.8.3.1 NIO简介

​		在JDK1.4之前的IO系统中，提供的都是面向流的I/O系统，系统一次一个字节地处理数据，一个输入流产生一个字节的数据，一个输出流消费一个字节的数据，面向流的I/O速度非常慢，在JDK1.4后推出了NIO即New I/O。这是一个面向块的I/O系统，系统以块的方式处理数据，每一个操作在一步中产生或消费一个数据库，按块处理要比按字节处理数据快的多。

![Java NIO](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_26.png "Java NIO")

##### 1.8.3.2 NIO中的几个基础概念

​		在NIO中有几个比较关键的概念：Channel(通道)，Buffer(缓冲区)，Selector(选择器)。

​		首先从Channel说起吧，通道顾名思义就是通向什么的道路，为IO提供渠道。在传统IO中，我们要读取一个文件中的内容，通常是像下面这样读取的：

```java
public class Test {
    public static void main(String[] args) throws IOException  {
        File file = new File("data.txt");
        InputStream inputStream = new FileInputStream(file);
        byte[] bytes = new byte[1024];
        inputStream.read(bytes);
        inputStream.close();
    }  
}
```

​		这里的`InputStream`实际上就是为读取文件提供一个通道的。因此可以将NIO中的Channel与传统的Stream来类比，但是需要注意的是，在传统IO中，Stream是单向的，比如`InputStream`只能进行读取操作，`OutputStream`只能进行写操作。而Channel是双向的，既可以用来进行读操作，又可用于写操作。

​		Buffer(缓冲区)是NIO中一个非常重要的东西，NIO中所有数据的读和写都离不开Buffer。比如上面的一段代码中，读取的数据时放在byte数组当中，而在NIO中，读取的数据只能放在Buffer中。同样地，写入数据也是先写入到Buffer中。

​		下面简单介绍一下NIO中最核心的一个东西：Selector。可以说它是NIO中最关键的一个部分，Selector的作用就是用来轮询每个注册的Channel，一旦发现Channel有注册的事件发生，便获取事件然后进行处理。例：

![selector](https://images0.cnblogs.com/i/288799/201408/181105032681855.jpg "selector")

​		用单线程处理一个Selector，然后通过<font color="#dd000">`Selector.select()`</font>方法来获取到达事件，在获取了到达事件之后，就可以逐个地对这些事件进行响应处理。

##### 1.8.3.3 Channel(通道)

​		前面已经提到Channel和传统IO中的Stream很相似。主要区别为：通道是双向的，通过一个Channel既可以进行读，也可以进行写；Stream是单向的，通过一个Stream只能进行读或写。

​		常用的几种通道：

- FileChannel - 从文件或向文件写入数据
- SocketChannel - 以TCP协议来向网络连接的两端读写数据
- ServerSocketChannel - 监听客户端发起的TCP连接，并为每个TCP连接创建一个新的SocketChannel来进行数据读写
- DatagramChannel - 以UDP协议来向网络连接的两端读写数据

​		以通过FileChannel来向文件中写入数据为例子：

```java
public class Test {
    public static void main(String[] args) throws IOException  {
        File file = new File("data.txt");
        FileOutputStream outputStream = new FileOutputStream(file);
        FileChannel channel = outputStream.getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        String string = "java nio";
        buffer.put(string.getBytes());
        buffer.flip();     //此处必须要调用buffer的flip方法
        channel.write(buffer);
        channel.close();
        outputStream.close();
    }  
}
```

​		通过上面的程序回想工程目录下data.txt写入字符串`java nio`，<font color="#dd000">注意在调用Channel的`channel.write()`方法之前必须调用Buffer的` buffer.flip()`方法，否则无法正确写入内容。</font>

> 注：<font color="#dd000">`buffer.flip()`</font>英文API: Flips this buffer. The limit is set to the current position and then the position is set to zero. If the mark is defined then it is discarded.中文释义："反转此缓冲区。首先对当前位置设置限制，然后将该位置设置为零。如果已定义了标记，则丢弃该标记。"; 
>
> ​		意思大概是这样的：调换这个buffer的当前位置，并且设置当前位置是0。说的意思就是：将缓存字节数组的指针设置为数组的开始序列即数组下标0。这样就可以从buffer开头，对该buffer进行遍历(读取)了。 
>
> ​		buffer中的flip方法涉及到buffer中的<font color="#dd000">Capacity,Position和Limit</font>三个概念。其中Capacity在读写模式下都是固定的，就是我们分配的缓冲大小,Position类似于读写指针，表示当前读(写)到什么位置,Limit在写模式下表示最多能写入多少数据，此时和Capacity相同，在读模式下表示最多能读多少数据，此时和缓存中的实际数据大小相同。在写模式下调用flip方法，那么limit就设置为了position当前的值(即当前写了多少数据),position会被置为0，以表示读操作从缓存的头开始读。<font color="#dd000">也就是说调用flip之后，读写指针指到缓存头部，并且设置了最多只能读出之前写入的数据长度(而不是整个缓存的容量大小)。</font>

​		如《Think in Java》P552:

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class GetChannel {
    private static final int SIZE = 1024;

    public static void main(String[] args) throws Exception {
        // 获取通道，该通道允许写操作
        FileChannel fc = new FileOutputStream("data.txt").getChannel();
        // 将字节数组包装到缓冲区中
        fc.write(ByteBuffer.wrap("Some text".getBytes()));
        // 关闭通道
        fc.close();

        // 随机读写文件流创建的管道
        fc = new RandomAccessFile("data.txt", "rw").getChannel();
        // fc.position()计算从文件的开始到当前位置之间的字节数
        System.out.println("此通道的文件位置：" + fc.position());
        // 设置此通道的文件位置,fc.size()此通道的文件的当前大小,该条语句执行后，通道位置处于文件的末尾
        fc.position(fc.size());
        // 在文件末尾写入字节
        fc.write(ByteBuffer.wrap("Some more".getBytes()));
        fc.close();

        // 用通道读取文件
        fc = new FileInputStream("data.txt").getChannel();
        ByteBuffer buffer = ByteBuffer.allocate(SIZE);
        // 将文件内容读到指定的缓冲区中
        fc.read(buffer);
        buffer.flip();//此行语句一定要有
        while (buffer.hasRemaining()) {
            System.out.print((char)buffer.get());
        }
                  fc.close();
    }
}
```

​		<font color="#dd000">注意：buffer.flip();一定得有，因为你写完数据后position是指向数据的尾部的。如果没有，就是从文件最后开始读取的，通过buffer.flip();这个语句，就能把buffer的当前位置更改为buffer缓冲区的第一个位置，并把limit值设为数据的尾部即反转缓冲区。这样读出的数据就是正确的</font>参考博客：[java.nio.Buffer flip()方法的用法详解](https://www.cnblogs.com/woshijpf/articles/3723364.html)

##### 1.8.3.4 Buffer(缓冲区)

​		Buffer顾名思义缓冲区，实际上是一个容器，一个连续的数组。Buffer 类是 Java NIO 的构造基础。一个 Buffer 对象是固定数量的数据的容器，其作用是一个存储器，或者分段运输区，在这里，数据可被存储并在之后用于检索。缓冲区可以被写满或释放。对于每个非布尔原始数据类型都有一个缓冲区类。Channel提供从文件、网络读取和写入数据的渠道，但是读取或写入的数据都必须经由Buffer。具体看下面这张图即可理解：

![buffer](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_27.png "buffer")

​		上图描述了从一个客户端向服务端发送数据，然后服务端接收数据的过程。客户端发送数据时，必须先将数据存入Buffer中，然后将Buffer中的内容写入Channel。服务端这边接收数据必须通过Channel将数据读入Buffer中，然后再从Buffer中取出数据来处理。

​		在BIO中，Buffer是一个顶层父类，它是一个抽象类，常用的Buffer子类有：

- ByteBuffer
- IntBuffer
- CharBuffer
- LongBuffer
- DoubleBuffer
- FloatBuffer
- ShortBuffer

​		如果是对于文件读写，上面几种Buffer都可能会用到。但是对于网络读写来说，用的最多的是ByteBuffer。<font color="#dd000">注意，是没有BooleanBuffer之说的</font>。尽管缓冲区作用于它们存储的原始数据类型，但缓冲区十分倾向于处理字节。非字节缓冲区可以在后台执行从字节或到字节的转换，这取决于缓冲区是如何创建的。 

​		缓冲区的四个属性，所有的缓冲区都具有四个属性来提供关于其所包含的数据元素的信息，这四个属性尽管简单，但其至关重要，需熟记于心：

1. 容量(Capacity)：缓冲区能够容纳的数组元素的最大容量。这一容量在缓冲区创建时被设定，并且永远不能改变。
2. 上界(Limit)：缓冲区的第一个不能被读或写的元素。缓冲区创建时，limit值等于capacity的值。假设capacity=1024，我们在程序中设置limit=512，则说明Buffer容量为1024，但是从512后既不能读也不能写，可以理解为Buffer实际容量为512。
3. 位置(Position)：下一个要被读或写的元素的索引，位置会自动由相应的get()和put()函数更新。
4. 标记(Mark)：一个备忘位置。标记在未设定前为未定义(undefined)。使用场景是，假设缓冲区有10个元素，position=2，现在只想发送6-10之间的数据，此时可以使用`buffer.mark(buffer.position())`把当前的position = 2 记入mark中，然后`buffer.position(6)`修改position值为6，此时发给Channel的数据就是6-10的数据。发送完成后调用`buffer.reset()`使得position = mark = 2，mark用于临时记录位置。

​		**在使用Buffer的时候，实际操作的就是这四个属性的值。**Buffer 类并没有包括 get() 或 put() 函数。但是，每一个Buffer 的子类都有这两个函数，但它们所采用的参数类型，以及它们返回的数据类型，对每个子类来说都是唯一的，所以它们不能在顶层 Buffer 类中被抽象地声明。它们的定义必须被特定类型的子类所遵从。若不加特殊说明，我们在下面讨论的一些内容，都是以 ByteBuffer 为例。

​		**相对存取和绝对存取：**

```java
public abstract class ByteBuffer extends Buffer implements Comparable {  
    // This is a partial API listing  
    public abstract byte get( );   
    public abstract byte get (int index);   
    public abstract ByteBuffer put (byte b);   
    public abstract ByteBuffer put (int index, byte b);  
}  
```

​		如上段代码，有不带索引参数的方法和带索引参数的方法。不带索引的get和put，这些调用执行完后，position的值会自动前进。对于put，如果多次调用导致位置超出limit，则会抛出BufferOverflowException异常；对于get，如果位置不小于limit，则会抛出BufferUnderflowException异常。不带索引参数的方法称为相对存取，相对存取会自动影响缓冲区的位置属性。带索引的方法，称为绝对存取，绝对存储不会影响缓冲区的位置属性，但如果你提供的索引值超出范围(负数或不小于上界)，也将抛出IndexOutOfBoundsException异常。 

​		**翻转：**

​		我们把hello这个串通过put存入一个ByteBuffer中

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);  
buffer.put((byte)'H').put((byte)'e').put((byte)'l').put((byte)'l').put((byte)'o'); 
```

​		此时position = 5，limit = capacity = 1024。现在我们要从正确的位置从buffer读数据，我们可以把position置为0。如果把上界设置成当前position的位置即5，那么limit就是结束的位置。上界指明了缓冲区有效内容的末端。手动实现缓冲区翻转：

```java
buffer.limit(buffer.position()).position(0);	//这行代码即是buffer.flip();的手动实现
```

​		此外，`rewind()`函数与`flip()`相似，但不会将limit值置为当前position值，只是将position值置为0。

​		**释放(Drain)：**

​		这里的释放，指的是缓冲区通过 put 填充数据后，然后被读出的过程。上面讲了，要读数据，首先得翻转。那么怎么读呢？<font color="#dd000">`hasRemaining() `</font>函数会在释放缓冲区时告诉你是否已经达到缓冲区的上界：

```java
for (int i = 0; buffer.hasRemaining(); i++) {  
    myByteArray[i] = buffer.get();  
}  
```

​		很明显，上面的代码每次循环都要判断元素是否到达上界，<font color="#dd000">`remaining()`</font>函数将告知你从当前位置到上界还剩余的元素数目，因此可做如下改进：

```java
int count = buffer.remaining();  
for (int i = 0; i < count; i++) {  
    myByteArray[i] = buffer.get();  
}  
```

​		上述代码虽然高效，但是缓冲区并不是多线程安全的。如果你想以多线程同时存取特定的缓冲区，你需要在存取缓冲区前进行数据同步。因此，使用第二段代码的前提是，你对缓冲区有专门的控制。

​		**`buffer.clear()`：**

​		`clear()`函数将缓冲区重置为空状态。它并不改变缓冲区的任何数据元素，而是仅仅将 limit 设为容量的值，并把 position 设回 0。

​		**压缩(Compact)：**

​		有时候我们只想释放部分数据，即只读取部分数据。当然，你可以把 position 指向你要读取的第一个数据的位置，将 limit 设置成最后一个元素的位置 + 1。但是，一旦缓冲区对象完成填充并释放，它就可以被重新使用了。所以，缓冲区一旦被读取出来，已经没有使用价值了。 

​		以 Mellow 为例，填充后为 Mellow，但如果我们仅仅想读取 llow。读取完后，缓冲区就可以重新使用了。Me 这两个位置对于我们而言是没用的。我们可以将 llow 复制至 0 - 3 上，Me 则被冲掉。但是 4 和 5 仍然为 o 和 w。这个事我们当然可以自行通过 get 和 put 来完成，但 API 给我们提供了一个`compact()`的函数，此函数比我们自己使用 get 和 put 要高效的多。 

​		compact之前的缓冲区：

![compact之前的缓冲区](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_28.png "compact之前的缓冲区")

​		执行`buffer.compact()`后会使缓冲区的状态图如下图所示：

![compact()之后的缓冲区](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_29.png "compact()之后的缓冲区")

> 这里发生了几件事情：
>
> 1. 数据元素 2 - 5 被复制到 0 - 3 位置，位置 4 和 5 不受影响，但现在正在或已经超出了当前位置，因此是“死的”。它们可以被之后的 put() 调用重写。
> 2. Position 已经被设为被复制的数据元素的数目，也就是说，缓冲区现在被定位在缓冲区中最后一个“存活”元素的后一个位置。
> 3. 调用`compact()`的作用是丢弃已经释放的数据，保留未释放的数据，并使缓冲区对重新填充容量准备就绪。该例子中，你当然可以将 Me 之前已经读过，即已经被释放过。

​		**缓冲区的比较：**

​		有时候比较两个缓冲区所包含的数据是很有必要的。所有类型的缓冲区都提供了一个常规的`equals()`函数用以测试两个缓冲区的是否相等，以及一个`compareTo()`函数用以比较缓冲区。

​		两个缓冲区被认为相等的充要条件：

- 两个对象类型相同。包含不同类型的buffer永远不会相等，而且buffer 绝不会等于非 buffer对象。
- 两个对象都剩余同样数量(num = limit - position)的元素。Buffer的容量不需要相同，而且缓冲区中剩余数据的索引也不必相同。
- 在每个缓冲区中应被`get()`函数返回的剩余数据元素序列([position, limit - 1] 位置对应的元素序列)必须一致。

​		缓冲区也支持用`compareTo()`函数以词典顺序进行比较，当然，这是所有的缓冲区实现了 java.lang.Comparable 语义化的接口。这也意味着缓冲区数组可以通过调用`java.util.Arrays.sort()`函数按照它们的内容进行排序。

​		 与`equals()`相似，`compareTo()`不允许不同对象间进行比较。但`compareTo()`更为严格：如果你传递一个类型错误的对象，它会抛出 ClassCastException 异常，但`equals()`只会返回 false。 

​		比较是针对每个缓冲区你剩余数据(从 position 到 limit)进行的，与它们在`equals()`中的方式相同，直到不相等的元素被发现或者到达缓冲区的上界。如果一个缓冲区在不相等元素发现前已经被耗尽，较短的缓冲区被认为是小于较长的缓冲区。这里有个顺序问题：下面小于零的结果(表达式的值为 true)的含义是 buffer2 < buffer1。切记，这代表的并不是 buffer1 < buffer2。

```java
if (buffer1.compareTo(buffer2) < 0) {  
    // do sth, it means buffer2 < buffer1，not buffer1 < buffer2  
    doSth();  
}  
```

​		**批量移动：**

​		缓冲区的设计目的就是为了能够高效传输数据，一次移动一个数据元素并不高效。如你在下面的程序清单中看到的那样，Buffer API提供了向缓冲区意外批量移动数据元素的函数：

```java
public abstract class ByteBuffer extends Buffer implements Comparable {  
    public ByteBuffer get(byte[] dst);  
    public ByteBuffer get(byte[] dst, int offset, int length);  
    public final ByteBuffer put(byte[] src);   
    public ByteBuffer put(byte[] src, int offset, int length);  
}
```

​		以 get 为例，它将缓冲区中的内容复制到指定的数组中，当然是从 position 开始。第二种形式使用 offset 和 length 参数来指定复制到目标数组的子区间。这些批量移动的合成效果与前文所讨论的循环是相同的，但是这些方法可能高效得多，因为这种缓冲区实现能够利用本地代码或其他的优化来移动数据。 

​		 批量移动总是具有指定的长度。也就是说，你总是要求移动固定数量的数据元素。因此，`get(dist)`和`get(dist, 0, dist.length)`是等价的。 

​		对于以下几种情况的数据复制会发生异常：

- 如果你所要求的数量的数据不能被传送，那么不会有数据被传递，缓冲区的状态保持不变，同时抛出BufferUnderflowException异常。
- 如果缓冲区中的数据不够完全填满数组，你会得到一个异常。这意味着如果你想将一个小型缓冲区传入一个大型数组，你需要明确地指定缓冲区中剩余的数据长度。

​		如果缓冲区存有比数组能容纳的数量更多的数据，你可以重复利用如下代码进行读取：

```java
byte[] smallArray = new Byte[10];  

while (buffer.hasRemaining()) {  
    int length = Math.min(buffer.remaining(), smallArray.length);  
    // length = 两个数中小些的那个数
    buffer.get(smallArray, 0, length);  
    // 每取出一部分数据后，即调用 processData 方法，length 表示实际上取到了多少字节的数据  
    processData(smallArray, length);  
} 
```

​		 `put()`的批量版本工作方式相似，只不过它是将数组里的元素写入 buffer 中而已，这里不再赘述。

​		**创建缓冲区：**

​		Buffer 的七种子类，没有一种能够直接实例化，它们都是抽象类，但是都包含静态工厂方法来创建相应类的新实例。这部分讨论中，将以 CharBuffer 类为例，对于其它六种主要的缓冲区类也是适用的。下面是创建一个缓冲区的关键函数，对所有的缓冲区类通用：

```java
public abstract class CharBuffer extends Buffer implements CharSequence, Comparable {  
    // This is a partial API listing  
  
    public static CharBuffer allocate (int capacity);  
    public static CharBuffer wrap (char [] array);  
    public static CharBuffer wrap (char [] array, int offset, int length);  
  
    public final boolean hasArray();  
    public final char [] array();  
    public final int arrayOffset();  
} 
```

​		新的缓冲区是由分配(allocate)或包装(wrap)创建的。分配(allocate)操作创建一个缓冲区对象并分配一个私有的空间来储存容量大小的数据元素。包装(wrap)操作创建一个缓冲区对象但是不分配任何空间来储存数据元素。它使用你所提供的数组作为存储空间来储存缓存区中的数据元素。demo如下：

```java
// 这段代码隐含地从堆空间中分配了一个 char 型数组作为备份存储器来储存 100 个 char 变量。  
CharBuffer charBuffer = CharBuffer.allocate (100);  
/** 
 * 这段代码构造了一个新的缓冲区对象，但数据元素会存在于数组中。这意味着通过调用 put() 函数造成的对缓 
 * 冲区的改动会直接影响这个数组，而且对这个数组的任何改动也会对这个缓冲区对象可见。 
 */  
char [] myArray = new char [100];   
CharBuffer charbuffer = CharBuffer.wrap (myArray);  
/** 
 * 带有 offset 和 length 作为参数的 wrap() 函数版本则会构造一个按照你提供的 offset 和 length 参 
 * 数值初始化 position 和 limit 的缓冲区。 
 * 
 * 这个函数并不像你可能认为的那样，创建了一个只占用了一个数组子集的缓冲区。这个缓冲区可以存取这个数组 
 * 的全部范围；offset 和 length 参数只是设置了初始的状态。调用 clear() 函数，然后对其进行填充， 
 * 直到超过 limit，这将会重写数组中的所有元素。 
 * 
 * slice() 函数可以提供一个只占用备份数组一部分的缓冲区。 
 * 
 * 下面的代码创建了一个 position 值为 12，limit 值为 54，容量为 myArray.length 的缓冲区。 
 */  
CharBuffer charbuffer = CharBuffer.wrap (myArray, 12, 42);  
```

​		通过`allocte()`或者`wrap()`函数创建的缓冲区通常都是间接的。间接的缓冲区使用备份数组，你可以通过上面列出的API函数获得对这些数组的存取权。

​		boolean型函数`hasArray()`告诉你这个缓冲区是否有一个可存取的备份数组。如果这个函数返回true，`array()`函数会返回这个缓冲区对象所使用的数组存储空间的引用。如果`hasArray()`返回false，若调用`array()`函数或`arrayOffset()`函数会报 UnsupportedOperationException 异常。

​		如果一个缓冲区是只读的，它的备份数组将会超出limit的，即使一个数组对象被提供给`wrap()`函数。调用`array()`函数或`arrayOffset()`函数会抛出 ReadOnlyBufferException 异常以阻止你得到存取权来修改只读缓冲区的内容。如果你通过其它的方式获得了对备份数组的存取权限，对这个数组的修改也会直接影响到这个只读缓冲区。 

​		`arrayOffset()`，返回缓冲区数据在数组中存储的开始位置的偏移量(从数组头0开始算)。如果你使用了带有三个参数的版本的`wrap()`函数来创建一个缓冲区，对于这个缓冲区，`arrayOffset()`会一直返回0。offset 和 length 只是指示了当前的 position 和 limit，是一个瞬间值，可以通过`clear()`来从 0 重新存数据，所以`arrayOffset()`返回的是 0。当然，如果你切分(`slice()`函数)了由一个数组提供存储的缓冲区，得到的缓冲区可能会有一个非 0 的数组偏移量。

​		**复制缓冲区：**

​		缓冲区不限于管理数组中的外部数据，它们也能管理其他缓冲区中的外部数据。当一个管理其他缓冲器所包含的数据元素的缓冲器被创建时，这个缓冲器被称为视图缓冲器。

​		视图存储器总是通过调用已存在的存储器实例中的函数来创建。使用已存在的存储器实例中的工厂方法意味着视图对象为原始存储器的你部实现细节私有。数据元素可以直接存取，无论它们是存储在数组中还是以一些其他的方式，而不需经过原始缓冲区对象的 get()/put() API。如果原始缓冲区是直接缓冲区，该缓冲区(视图缓冲区)的视图会具有同样的效率优势。

​		 继续以 CharBuffer 为例，但同样的操作可被用于任何基本的缓冲区类型。用于复制缓冲区的API：

```java
public abstract class CharBuffer extends Buffer implements CharSequence, Comparable {  
    // This is a partial API listing  
      
    public abstract CharBuffer duplicate();  
    public abstract CharBuffer asReadOnlyBuffer();  
    public abstract CharBuffer slice();  
}  
```

​		复制一个缓冲区会创建一个新的 Buffer 对象，但并不复制数据。原始缓冲区和副本都会操作同样的数据元素。

​		`duplicate()`函数创建了一个与原始缓冲区相似的新缓冲区。两个缓冲区共享数据元素，拥有同样的容量，但每个缓冲区拥有各自的 position、limit 和 mark 属性。对一个缓冲区你的数据元素所做的改变会反映在另外一个缓冲区上。这一副本缓冲区具有与原始缓冲区同样的数据视图。如果原始的缓冲区为只读，或者为直接缓冲区，新的缓冲区将继承这些属性。`duplicate()`复制缓冲区：

```java
CharBuffer buffer = CharBuffer.allocate(8);  
buffer.position(3).limit(6).mark().position (5);  
CharBuffer dupeBuffer = buffer.duplicate();  
buffer.clear(); 
```

​		`duplicate()`复制3-5数据到新缓冲区示意图：

![duplicate()复制3-5数据到新缓冲区](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_30.png "duplicate()复制3-5数据到新缓冲区")

​		使用`asReadOnlyBuffer()`函数生成一个只读的缓冲区视图。这与`duplicate()`相同，除了这个新的缓冲区bu允许使用`put()`，并且其`isReadOnly()`函数将返回true。

​		如果一个只读的缓冲区与一个可写的缓冲区共享数据，或者有包装好的备份数组，那么对这个可写的缓冲区或直接对这个数组的改变将同时反映在所有的关联缓冲区上，包括只读缓冲区。

​		`slice()`分割缓冲区与复制缓冲区相似，但`slice()`创建一个从原始缓冲区的当前position开始的新缓冲区，并且其 capacity(容量)是原始缓冲区的剩余元素数量(limit - position)。这个新缓冲区与原始缓冲区共享一段数据元素子序列。分割出来的缓冲区也会继承只读属性和直接属性。使用`slice()`分割缓冲区实例如下：

```java
CharBuffer buffer = CharBuffer.allocate(8);
buffer.position(3).limit(5);  
CharBuffer sliceBuffer = buffer.slice();
```

![使用slice()创建分割缓冲区](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_31.png "使用slice()创建分割缓冲区")

​		**字节缓冲区：**

​		ByteBuffer只是Buffer的一个子类，但字节缓冲区有字节的独特之处。字节缓冲区跟其他缓冲区类型最明显的不同在于，它可以成为通道所执行的I/O的源头或目标，后面你会发现通道直接收ByteBuffer作为参数。

​		字节是操作系统及其I/O设备使用的基本数据类型。当在JVM和操作系统间传递数据时，将其他的数据类型拆分成构成他们的字节是十分必要的，系统层次的I/O面向字节的性质可以在整个缓冲区的设计以及它们互相配合的服务中感受到。同时，操作系统是在内存区域中进行I/O操作。这些内存区域就操作系统方面而言，是相连的字节序列。于是毫无疑问，只有字节缓冲区有资格参与I/O操作。

​		非字节类型的基本类型，除了布尔型都是由组合在一起的几个字节组成的。那么必然要引出另外一个问题：字节顺序。

​		多字节数值被存储在内存中的方式一般被称为 endian-ness（字节顺序）。如果数字数值的最高字节 － big end（大端），位于低位地址(即 big end 先写入内存，先写入的内存的地址是低位的，后写入内存的地址是高位的)，那么系统就是大端字节顺序。如果最低字节最先保存在内存中，那么系统就是小端字节顺序。在 java.nio 中，字节顺序由 ByteOrder 类封装：

```java
package java.nio;  
  
public final class ByteOrder {  
    public static final ByteOrder BIG_ENDIAN;  
    public static final ByteOrder LITTLE_ENDIAN;  
  
    public static ByteOrder nativeOrder();  
    public String toString();  
} 
```

​		ByteOrder 类定义了决定从缓冲区中存储或检索多字节数值时使用哪一字节顺序的常量。如果你需要知道 JVM 运行的硬件平台的固有字节顺序，请调用静态类函数` nativeOrder()`。

​		每一个缓冲区类都有一个能够通过调用`order()`查询的当前字节顺序：

```java
public abstract class CharBuffer extends Buffer implements Comparable, CharSequence {  
    // This is a partial API listing  
  
    public final ByteOrder order();  
}  
```

​		这个函数从ByteOrder返回两个常量之一。对于除了ByteBuffer之外的其他缓冲区类，字节顺序是一个只读属性，并且可能根据缓冲过去的建立方式而采用不同的值。除了ByteBuffer，其他通过`allocate()`或`wrap()`一个数组所创建的缓冲区将从`order()`返回与`ByteOrder.nativeOrder()`相同的数值。这是因为包含在缓冲区中的元素在 JVM 中将会被作为基本数据直接存取。

​		ByteBuffer 类有所不同：默认字节顺序总是 ByteBuffer.BIG_ENDIAN，无论系统的固有字节顺序是什么。Java 的默认字节顺序是大端字节顺序，这允许类文件等以及串行化的对象可以在任何 JVM 中工作。如果固有硬件字节顺序是小端，这会有性能隐患。在使用固有硬件字节顺序时，将 ByteBuffer 的内容当作其他数据类型存取很可能高效得多。

​		 为什么 ByteBuffer 类需要一个字节顺序？字节不就是字节吗？ByteBuffer 对象像其他基本数据类型一样，具有大量便利的函数用于获取和存放缓冲区内容。这些函数对字节进行编码或解码的方式取决于 ByteBuffer 当前字节顺序的设定。ByteBuffer 的字节顺序可以随时通过调用以 ByteOrder.BIG_ENDIAN 或 ByteOrder.LITTL_ENDIAN 为参数的 order() 函数来改变：

```java
public abstract class ByteBuffer extends Buffer implements Comparable {  
    // This is a partial API listing  
  
    public final ByteOrder order();  
    public final ByteBuffer order(ByteOrder bo);  
} 
```

​		如果一个缓冲区被创建为一个 ByteBuffer 对象的视图，，那么 order() 返回的数值就是视图被创建时其创建源头的 ByteBuffer 的字节顺序。视图的字节顺序设定在创建后不能被改变，而且如果原始的字节缓冲区的字节顺序在之后被改变，它也不会受到影响。

​		**直接缓冲区：**

​		内核空间(与之相对的是用户空间，如JVM)是操作系统所在区域，它能与设备控制器(硬件)通讯，控制着用户区域进程(如JVM)的运行状态。最重要的是，所有的I/O都直接(物理内存)或间接(虚拟内存)通过内核空间。

​		当进程(如JVM)请求I/O操作的时候，它执行一个系统调用将控制权移交给内核。当内核以这种方式被调用，它随即采取任何必要步骤，找到进程所需数据，并把数据传送到用户空间你的指定缓冲区。内核试图对数据进行高速缓存或预读取，因此进程所需数据可能已经在内核空间里了。如果是这样，该数据只需简单地拷贝出来即可。如果数据不在内核空间，则进程被挂起，内核着手吧数据读进内存。

![I/O缓冲区操作简图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_32.png "I/O缓冲区操作简图")

​		从图中你可能会觉得，把数据从内核空间拷贝到用户空间似乎有些多余。为什么不直接让磁盘控制器把数据送到用户空间的缓冲区呢？首先，硬件通常不能直接访问用户空间。其次，像磁盘这样基于块存储的硬件设备操作的是固定大小的数据块，而用户进程请求的可能是任意大小的或非对齐的数据块。在数据往来于用户空间与存储设备的过程中，内核负责数据的分解、再组合工作，因此充当着中间人的角色。

​		因此，操作系统是在内存区域中进行 I/O 操作。这些内存区域，就操作系统方面而言，是相连的字节序列，这也意味着I/O操作的目标内存区域必须是连续的字节序列。在 JVM中，字节数组可能不会在内存中连续存储(因为 JAVA 有 GC 机制)，或者无用存储单元(会被垃圾回收)收集可能随时对其进行移动。

​		出于这个原因，引入了直接缓冲区的概念。直接字节缓冲区通常是 I/O 操作最好的选择。非直接字节缓冲区(即通过`allocate()`或`wrap()`创建的缓冲区)可以被传递给通道，但是这样可能导致性能损耗。通常非直接缓冲不可能成为一个本地 I/O 操作的目标。

​		如果你向一个通道中传递一个非直接 ByteBuffer 对象用于写入，通道可能会在每次调用中隐含地进行下面的操作：

1. 创建一个临时的直接 ByteBuffer 对象。
2. 将非直接缓冲区的内容复制到临时直接缓冲区中。
3. 使用临时直接缓冲区执行低层 I/O 操作。
4. 临时直接缓冲区对象离开作用域，并最终成为被回收的无用数据。

​        这可能导致缓冲区在每个 I/O 上复制并产生大量对象，而这种事都是我们极力避免的。如果你仅仅为一次使用而创建了一个缓冲区，区别并不是很明显。另一方面，如果你将在一段高性能脚本中重复使用缓冲区，分配直接缓冲区并重新使用它们会使你游刃有余。

​		直接缓冲区可能比创建非直接缓冲区要花费更高的成本，它使用的内存是通过调用本地操作系统方面的代码分配的，绕过了标准 JVM 堆栈，不受垃圾回收支配，因为它们位于标准 JVM 堆栈之外。

​		直接 ByteBuffer 是通过调用具有所需容量的` ByteBuffer.allocateDirect()`函数产生的。注意，`wrap()` 函数所创建的被包装的缓冲区总是非直接的。**与直接缓冲区相关的 API：**

```java
public abstract class ByteBuffer extends Buffer implements Comparable {  
    // This is a partial API listing  
  
    public static ByteBuffer allocateDirect (int capacity);  
    public abstract boolean isDirect();  
}  
```

​		所有的缓冲区都提供了一个叫做`isDirect()`的 boolean 函数，来测试特定缓冲区是否为直接缓冲区。但是，**ByteBuffer 是唯一可以被分配成直接缓冲区的 Buffer。**尽管如此，如果基础缓冲区是一个直接 ByteBuffer，对于非字节视图缓冲区，`isDirect()`可以是 true。

​		**视图缓冲区：**

​		I/O基本上可以归结成组字节数据的四处传递，在进行大数据量的 I/O 操作时，很又可能你会使用各种 ByteBuffer 类去读取文件内容，接收来自网络连接的数据，等等。ByteBuffer 类提供了丰富的 API 来创建视图缓冲区。

​		视图缓冲区通过已存在的缓冲区对象实例的工厂方法来创建。这种视图对象维护它自己的属性，容量，位置，上界和标记，但是和原来的缓冲区共享数据元素。

​		每一个工厂方法都在原有的 ByteBuffer 对象上创建一个视图缓冲区。调用其中的任何一个方法都会创建对应的缓冲区类型，这个缓冲区是基础缓冲区的一个切分，由基础缓冲区的位置和上界决定。新的缓冲区的容量是字节缓冲区中存在的元素数量除以视图类型中组成一个数据类型的字节数，在切分中任一个超过上界的元素对于这个视图缓冲区都是不可见的。视图缓冲区的第一个元素从创建它的 ByteBuffer 对象的位置开始（`positon()`函数的返回值）。**来自 ByteBuffer 创建视图缓冲区的工厂方法：**

```java
public abstract class ByteBuffer extends Buffer implements Comparable {  
    // This is a partial API listing  
  
    public abstract CharBuffer asCharBuffer();    
    public abstract CharBuffer asShortBuffer();    
    public abstract CharBuffer asIntBuffer();    
    public abstract CharBuffer asLongBuffer();    
    public abstract CharBuffer asFloatBuffer();    
    public abstract CharBuffer asDoubleBuffer(); 
}
```

​		下面的代码创建了一个 ByteBuffer 缓冲区的 CharBuffer 视图。**演示 7 个字节的 ByteBuffer 的 CharBuffer 视图：**

```java
/** 
 * 1 char = 2 byte，因此 7 个字节的 ByteBuffer 最终只会产生 capacity 为 3 的 CharBuffer。 
 * 
 * 无论何时一个视图缓冲区存取一个 ByteBuffer 的基础字节，这些字节都会根据这个视图缓冲区的字节顺序设 
 * 定被包装成一个数据元素。当一个视图缓冲区被创建时，视图创建的同时它也继承了基础 ByteBuffer 对象的 
 * 字节顺序设定，这个视图的字节排序不能再被修改。字节顺序设定决定了这些字节对是怎么样被组合成字符 
 * 型变量的，这样可以理解为什么 ByteBuffer 有字节顺序的概念了吧。 
 */  
ByteBuffer byteBuffer = ByteBuffer.allocate(7).order (ByteOrder.BIG_ENDIAN);  
CharBuffer charBuffer = byteBuffer.asCharBuffer();
```

![7个字节的ByteBuffer的CharBuffer视图](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_33.png "7个字节的ByteBuffer的CharBuffer视图")

​		**数据元素视图：**

​		ByteBuffer类为每一种原始数据类型提供了存取和转化的方法：

```java
public abstract class ByteBuffer extends Buffer implements Comparable {  
    public abstract short getShort( );   
    public abstract short getShort(int index);  
    public abstract short getInt( );   
    public abstract short getInt(int index);  
    ......  
  
    public abstract ByteBuffer putShort(short value);   
    public abstract ByteBuffer putShort(int index, short value);  
    public abstract ByteBuffer putInt(int value);   
    public abstract ByteBuffer putInt(int index, int value);  
    .......  
} 
```

​		这些函数从当前位置开始存取 ByteBuffer 的字节数据，就好像一个数据元素被存储在那里一样。根据这个缓冲区的当前的有效的字节顺序，这些字节数据会被排列或打乱成需要的原始数据类型。

​		如果`getInt()`函数被调用，从当前的位置开始的四个字节会被包装成一个 int 类型的变量然后作为函数的返回值返回。实际的返回值取决于缓冲区的当前的比特排序(byte-order)设置。**不同字节顺序取得的值是不同的：**

```java
// 大端顺序  
int value = buffer.order(ByteOrder.BIG_ENDIAN).getInt();  
// 小端顺序  
int value = buffer.order(ByteOrder.LITTLE_ENDIAN).getInt();  
  
// 上述两种方法取得的 int 是不一样的，因此在调用此类方法前，请确保字节顺序是你所期望的 
```

​		如果你试图获取的原始类型需要比缓冲区中存在的字节数更多的字节，会抛出 BufferUnderflowException。参考文章：[NIO-Buffer](https://www.iteye.com/blog/zachary-guo-1457542)

##### 1.8.3.5 Selector(选择器)

​		Selector类是NIO的核心类，Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理。这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。

​		参照Java doc中Selector描述的第一句话，Selector的作用是Java NIO中管理一组多路复用的SelectableChannel对象，并能够识别通道是否为诸如读写事件做好准备的组件。

![Selector](http://www.sico-technology.cn:81/images/java_note/jvm/jvm_34.png "Selector")

​		Selector的创建过程如下：

```java
// 1.创建Selector
Selector selector = Selector.open();

// 2.将Channel注册到选择器中
// ....... new channel的过程 ....

//Notes：channel要注册到Selector上就必须是非阻塞的，所以FileChannel是不可以使用Selector的，因为FileChannel是阻塞的
channel.configureBlocking(false);

// 第二个参数指定了我们对 Channel 的什么类型的事件感兴趣
SelectionKey key = channel.register(selector , SelectionKey.OP_READ);

// 也可以使用或运算|来组合多个事件，例如
SelectionKey key = channel.register(selector , SelectionKey.OP_READ | SelectionKey.OP_WRITE);

// 不过值得注意的是，一个 Channel 仅仅可以被注册到一个 Selector 一次, 如果将 Channel 注册到 Selector 多次, 那么其实就是相当于更新 SelectionKey 的 interest set.
```

一个Channel在Selector注册其代表的是一个`SelectionKey`事件，`SelectionKey`的类型包括：

- `OP_READ`：可读事件；值为：`1<<0`
- `OP_WRITE`：可写事件；值为：`1<<2`
- `OP_CONNECT`：客户端连接服务端的事件(tcp连接)，一般为创建`SocketChannel`客户端channel；值为：`1<<3`
- `OP_ACCEPT`：服务端接收客户端连接的事件，一般为创建`ServerSocketChannel`服务端channel；值为：`1<<4`

一个Selector内部维护了三组keys：

1. `key set`:当前channel注册在Selector上所有的key；可调用`keys()`获取
2. `selected-key set`:当前channel就绪的事件；可调用`selectedKeys()`获取
3. `cancelled-key`:主动触发`SelectionKey#cancel()`方法会放在该集合，前提条件是该channel没有被取消注册；不可通过外部方法调用

Selector类中总共包含以下10个方法：

- `open()`:创建一个Selector对象
- `isOpen()`:是否是open状态，如果调用了`close()`方法则会返回`false`
- `provider()`:获取当前Selector的`Provider`
- `keys()`:如上文所述，获取当前channel注册在Selector上所有的key
- `selectedKeys()`:获取当前channel就绪的事件列表
- `selectNow()`:获取当前是否有事件就绪，该方法立即返回结果，不会阻塞；如果返回值>0，则代表存在一个或多个
- `select(long timeout)`:selectNow的阻塞超时方法，超时时间内，有事件就绪时才会返回；否则超过时间也会返回
- `select()`:selectNow的阻塞方法，直到有事件就绪时才会返回
- `wakeup()`:调用该方法会时，阻塞在`select()`处的线程会立马返回；(ps：下面一句划重点)即使当前不存在线程阻塞在`select()`处，那么下一个执行`select()`方法的线程也会立即返回结果，相当于执行了一次`selectNow()`方法
- `close()`: 用完`Selector`后调用其`close()`方法会关闭该Selector，且使注册到该`Selector`上的所有`SelectionKey`实例无效。channel本身并不会关闭。

​		与Selector有关的一个关键类是SelectionKey，一个SelectionKey表示一个到达的事件，这2个类构成了服务端处理业务的关键逻辑。往Channel注册Selector会返回一个SelectionKey对象，这个对象包含了如下内容：

- interest set，当前Channel感兴趣的事件集，即在调用`register`方法设置的interest set
- ready set
- channel
- selector
- attached object，可选的附加对象

**interest set**

可以通过SelectionKey类中的方法来获取和设置interest set

```java
// 返回当前感兴趣的事件列表
int interestSet = key.interestOps();

// 也可通过interestSet判断其中包含的事件
boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;    

// 可以通过interestOps(int ops)方法修改事件列表
key.interestOps(interestSet | SelectionKey.OP_WRITE);
```

**ready set**
当前Channel就绪的事件列表

```java
int readySet = key.readyOps();

// 也可通过四个方法来分别判断不同事件是否就绪
key.isReadable();    //读事件是否就绪
key.isWritable();    //写事件是否就绪
key.isConnectable(); //客户端连接事件是否就绪
key.isAcceptable();  //服务端连接事件是否就绪
```

**channel和selector**
我们可以通过SelectionKey来获取当前的channel和selector

```java
// 返回当前事件关联的通道，可转换的选项包括:`ServerSocketChannel`和`SocketChannel`
Channel channel = key.channel();

//返回当前事件所关联的Selector对象
Selector selector = key.selector();
```

**attached object**
我们可以在selectionKey中附加一个对象:

```java
key.attach(theObject);
Object attachedObj = key.attachment();
```

或者在注册时直接附加:

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

**一个完整的Selector例子**

一个Selector的基本使用流程包括：

1. 创建一个Selector

2. 将Channel注册到Selector中，并设置监听的interest set

3. loop

   - 执行select()方法

   - 调用selector.selectedKeys()获取当前就绪的key
   - 迭代selectedKeys
     - 从key中获取对应的Channel和附加信息(if exist)
     - 判断是哪些 IO 事件已经就绪了, 然后处理它们. 如果是 OP_ACCEPT 事件, 则调用 "SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept()" 获取 SocketChannel, 并将它设置为 非阻塞的, 然后将这个 Channel 注册到 Selector 中.
     - 根据需要更改 selected key 的监听事件.
     - 将已经处理过的 key 从 selected keys 集合中删除.

```java
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.nio.charset.Charset;
import java.util.Iterator;
import java.util.Set;

/**
 * Created by locoder on 2019/2/28.
 */
public class SelectorDemo {

    public static void main(String[] args) throws IOException {
        // create a Selector
        Selector selector = Selector.open();

        // new Server Channel
        ServerSocketChannel ssc = ServerSocketChannel.open();
        // config async
        ssc.configureBlocking(false);

        ssc.socket().bind(new InetSocketAddress(8080));

        // register to selector
        // Notes：这里只能注册OP_ACCEPT事件，否则将会抛出IllegalArgumentException,详见AbstractSelectableChannel#register方法
        ssc.register(selector, SelectionKey.OP_ACCEPT);

        // loop
        for (; ; ) {
            int nKeys = selector.select();

            if (nKeys > 0) {
                Set<SelectionKey> keys = selector.selectedKeys();

                for (Iterator<SelectionKey> it = keys.iterator(); it.hasNext(); ) {
                    SelectionKey key = it.next();

                    // 处理客户端连接事件
                    if (key.isAcceptable()) {
                        ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();

                        SocketChannel clientChannel = serverSocketChannel.accept();

                        clientChannel.configureBlocking(false);

                        clientChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024 * 1024));

                    } else if (key.isReadable()) {
                        SocketChannel socketChannel = (SocketChannel) key.channel();

                        ByteBuffer buf = (ByteBuffer) key.attachment();

                        int readBytes = 0;
                        int ret = 0;

                        try {
                            while ((ret = socketChannel.read(buf)) > 0) {
                                readBytes += ret;
                            }

                            if (readBytes > 0) {
                                String message = decode(buf);
                                System.out.println(message);

                                // 这里注册写事件，因为写事件基本都处于就绪状态；
                                // 从处理逻辑来看，一般接收到客户端读事件时也会伴随着写，类似HttpServletRequest和HttpServletResponse
                                key.interestOps(key.interestOps() | SelectionKey.OP_WRITE);

                            }
                        } finally {
                            // 将缓冲区切换为待读取状态
                            buf.flip();
                        }

                    } else if (key.isValid() && key.isWritable()) {
                        SocketChannel socketChannel = (SocketChannel) key.channel();

                        ByteBuffer buf = (ByteBuffer) key.attachment();

                        if (buf.hasRemaining()) {
                            socketChannel.write(buf);
                        } else {
                            // 取消写事件，否则写事件内的代码会不断执行
                            // 因为写事件就绪的条件是判断缓冲区是否有空闲空间，绝大多时候缓存区都是有空闲空间的
                            key.interestOps(key.interestOps() & (~SelectionKey.OP_WRITE));
                        }

                        // 丢弃本次内容
                        buf.compact();
                    }
                    // 注意, 在每次迭代时, 我们都调用 "it.remove()" 将这个 key 从迭代器中删除,
                    // 因为 select() 方法仅仅是简单地将就绪的 IO 操作放到 selectedKeys 集合中,
                    // 因此如果我们从 selectedKeys 获取到一个 key, 但是没有将它删除, 那么下一次 select 时, 这个 key 所对应的 IO 事件还在 selectedKeys 中.
                    it.remove();
                }
            }

        }
    }

    /**
     * 将ByteBuffer转换为String
     *
     * @param in
     * @return
     * @throws UnsupportedEncodingException
     */
    private static String decode(ByteBuffer in) throws UnsupportedEncodingException {
        String receiveText = new String(in.array(), 0, in.capacity(), Charset.defaultCharset());
        int index = -1;
        if ((index = receiveText.lastIndexOf("\r\n")) != -1) {
            receiveText = receiveText.substring(0, index);
        }
        return receiveText;
    }

}

```

**深入Selector源码**

​		创建过程：

![创建过程](https://upload-images.jianshu.io/upload_images/4589271-3dcc0c6cbe9c86c5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp "创建过程")

>从上图上可以比较清晰得看到，openjdk中Selector的实现是SelectorImpl,
 然后SelectorImpl又将职责委托给了具体的平台，比如图中框出的linux2.6以后才有的EpollSelectorImpl, Windows平台则是WindowsSelectorImpl, MacOSX平台是KQueueSelectorImpl.

```java
public static Selector open() throws IOException {
    return SelectorProvider.provider().openSelector();
}
// 创建是依赖SelectorProvider.provider()系统级提供
// 我们来看SelectorProvider.provider()方法
// 从系统配置java.nio.channels.spi.SelectorProvider获取
if (loadProviderFromProperty()) return provider;
// 从ServiceLoader#load
if (loadProviderAsService()) return provider;
// 如果还不存在则使用默认provider，即KQueueSelectorProvider
provider = sun.nio.ch.DefaultSelectorProvider.create();
```

​		注册过程：

```java
// AbstractSelectableChannel#register方法
SelectionKey register(Selector sel, int ops,Object att){
        synchronized (regLock) {
            // 判断当前Channel是否关闭
            if (!isOpen())
                throw new ClosedChannelException();
            // 判断参数ops是否只包含OP_ACCEPT
            if ((ops & ~validOps()) != 0)
                throw new IllegalArgumentException();
            // 使用Selector则Channel必须是非阻塞的
            if (blocking)
                throw new IllegalBlockingModeException();
            // 根据Selector找到SelectionKey，它是可复用的，一个Selector只能有一个SelectionKey，如果存在则直接覆盖ops和attachedObject
            SelectionKey k = findKey(sel);
            if(key != null) {
                ....
            }
            // 如果不存在则直接实例化一个SelectionKeyImpl对象，并为ops和attachedObject赋值；实际调用AbstractSelector的register方法
            // 将Selector和SelectionKey绑定
            if (k == null) {
                // New registration
                synchronized (keyLock) {
                    if (!isOpen())
                        throw new ClosedChannelException();
                    k = ((AbstractSelector)sel).register(this, ops, att);
                    addKey(k);
                }
            }
            return k;
        }
    }
```

​		select过程：

​		select是Selector模型中最关键的一步，下面让我们来研究一下其过程

```java
// 首先来看select的调用链
// SelectorImpl#select -> SelectorImpl#lockAndDoSelect -> 具体provider提供的Selector中的doSelect方法
// 值得注意的是：在lockAndDoSelect方法中执行了`synchronized(this)`操作，故select操作是阻塞的

// open过程中我们知道，Selector有好几种实现，但基本都包含以下操作;感兴趣的同学可以具体看看这位大神写的博客：https://juejin.im/entry/5b51546df265da0f70070b93；这里就不深入写这部分了，篇幅有点长

int doSelect(long timeout) {
    // close判断，如果closed，则抛出ClosedSelectorException
    
    // 处理掉被cancel掉的SelectionKey，即`cancelled-key`
    this.processDeregisterQueue();
    
    try {
        // 设置中断器，实际调用的是AbstractSelector.this.wakeup();方法
    // 调用的是方法AbstractInterruptibleChannel.blockedOn(Interruptible);
        this.begin();
        // 从具体的模型中(kqueue、poll、epoll)选择
        this.pollWrapper.poll(...);
    }finally {
        // 关闭中断器
        this.end();
    }
    // 重新处理被cancel的key
    this.processDeregisterQueue();
    // 更新各个事件的状态
    int selectedKeys = this.updateSelectedKeys();
    // 可能还有一些操作
    .....
    return selectedKeys;
}

```

> 参考文献:[Selector详解](https://www.jianshu.com/p/f26f1eaa7c8e)、[[Java NIO之Selector（选择器）](https://www.cnblogs.com/snailclimb/p/9086334.html)

---



