## JVM

![preview](https://pic3.zhimg.com/v2-dfc3b65b983a82e84420e1335a507e2a_r.jpg)

1，运行时数据区：堆，栈（虚拟机栈，本地方法栈），方法区，程序计数器

### 程序计数器：

当前线程所执行字节码的行号指示器，指向下一个将要执行的指令代码，为了确保线程切换后（上下文切换）能恢复到正确的执行位置，每个线程都有一个独立的程序计数器（私有的），如果执行Native方法则为undefined

### 栈 Stacks

限定仅在表头进行插入和删除操作的线性表，后进先出，栈是线程私有，每个线程拥有独立的栈空间

栈帧是栈的元素，栈帧存放 局部变量表，操作数栈，动态连接和方法出口等信息

#### java虚拟机栈

为JVM执行java方法服务，线程私有的

**它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：**每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。**每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。**

局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同于对象本身，根据不同的虚拟机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。

其中64位长度的long和double类型的数据会占用2个局部变量空间（Slot），其余的数据类型只占用1个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

操作数栈：通常进行算数运算的时候是通过操作数栈来进行，又或者调用其他方法的时候通过操作数栈进行参数传递，操作数栈可以理解为栈帧中用于计算的临时数据存储区

#### 本地方法栈

本地方法栈为JVM使用到native方法服务

 

### 堆（heap）

堆是内存共享，主要存放new关键字创建的对象

堆分为年轻代，老年代；年轻代又分为伊甸区（eden）和幸存去（survivor）;幸存区又分为FromSurvivor和ToSurvivor

年轻代存储新创建的对象，当年轻内存占满后会触发 minorGC,清理年轻代内存

老年代存储长期存活和大对象，年轻代中经过多次GC仍然存活的对象会移动到老年代中，老年代空间满后，会触发Full GC

 

### 方法区

线程共享，静态变量，常量，类信息（版本，方法，字段等）,运行时常量池存在方法区中。

常量池存储 字面量和符号引用

###  控制参数

- -Xms设置堆的最小空间大小。
- -Xmx设置堆的最大空间大小。
- -XX:NewSize设置新生代最小空间大小。
- -XX:MaxNewSize设置新生代最大空间大小。
- -XX:PermSize设置永久代最小空间大小。
- -XX:MaxPermSize设置永久代最大空间大小。
- -Xss设置每个线程的堆栈大小。

## 垃圾回收

### 对象存活判断

判断对象是否存活一般有两种方式：

**引用计数**：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。
**可达性分析**（Reachability Analysis）：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。不可达对象。

在Java语言中，GC Roots包括：

- 虚拟机栈中引用的对象。
- 方法区中类静态属性实体引用的对象。
- 方法区中常量引用的对象。
- 本地方法栈中JNI引用的对象。

### 垃圾收集算法

#### 标记-清除

算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的。

它的主要缺点有两个：一个是效率问题，标记和清除过程的效率都不高；另外一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjorBnVw0K7iaaPwhicLhrpb8nibkgia2lkHzovEgPo2VC23QTr8EMpiaPxuFGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 复制算法

“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

这样使得每次都是对其中的一块进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半，持续复制长生存期的对象则导致效率降低。

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjorcDwjZkYkrJ4fpgTibYjMEDGVK81YIQWDpW0k1S9ibjxLvRz3848v91qg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 标记-压缩算法

复制收集算法在对象存活率较高时就要执行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjorH0wLYIx1oZ7fGo2uNUgB6dwGLYV3h7pJtMgficMpicMOhUENpStgSCog/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 垃圾收集器

> 如果说收集算法是内存回收的方法论，垃圾收集器就是内存回收的具体实现

### Serial收集器

串行收集器是最古老，最稳定以及效率高的收集器，可能会产生较长的停顿，只使用一个线程去回收。新生代、老年代使用串行回收；新生代复制算法、老年代标记-压缩；垃圾收集的过程中会Stop The World（服务暂停）

参数控制： `-XX:+UseSerialGC` 串行收集器

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjoryZoyPFzBtvO0wBX94sUqX2Y2sDc7TXxMn731ibmvYVLZcOibgvxZRlRQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

ParNew收集器 ParNew收集器其实就是Serial收集器的多线程版本。新生代并行，老年代串行；新生代复制算法、老年代标记-压缩

参数控制：

`-XX:+UseParNewGC` ParNew收集器
`-XX:ParallelGCThreads` 限制线程数量

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjorKNB1pJ8YMVtHFA157S1SlGRjXNibGpz31sPrCtpeb4zL3vWampq8dfQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### Parallel收集器

Parallel Scavenge收集器类似ParNew收集器，Parallel收集器更关注系统的吞吐量。可以通过参数来打开自适应调节策略，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大的吞吐量；也可以通过参数控制GC的时间不大于多少毫秒或者比例；新生代复制算法、老年代标记-压缩

参数控制： `-XX:+UseParallelGC` 使用Parallel收集器+ 老年代串行

### Parallel Old 收集器

Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法。这个收集器是在JDK 1.6中才开始提供

参数控制： `-XX:+UseParallelOldGC` 使用Parallel收集器+ 老年代并行

### CMS收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的Java应用都集中在互联网站或B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。

从名字（包含“Mark Sweep”）上就可以看出CMS收集器是基于“标记-清除”算法实现的，它的运作过程相对于前面几种收集器来说要更复杂一些，整个过程分为4个步骤，包括：

- 初始标记（CMS initial mark）
- 并发标记（CMS concurrent mark）
- 重新标记（CMS remark）
- 并发清除（CMS concurrent sweep）

其中初始标记、重新标记这两个步骤仍然需要“Stop The World”。初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，并发标记阶段就是进行GC Roots Tracing的过程，而重新标记阶段则是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。

由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以总体上来说，CMS收集器的内存回收过程是与用户线程一起并发地执行。老年代收集器（新生代使用ParNew）

**优点**: 并发收集、低停顿
**缺点**: 产生大量空间碎片、并发阶段会降低吞吐量

参数控制：

`-XX:+UseConcMarkSweepGC` 使用CMS收集器
`-XX:+ UseCMSCompactAtFullCollection` Full GC后，进行一次碎片整理；整理过程是独占的，会引起停顿时间变长
`-XX:+CMSFullGCsBeforeCompaction` 设置进行几次Full GC后，进行一次碎片整理
`-XX:ParallelCMSThreads` 设定CMS的线程数量（一般情况约等于可用CPU数量）

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjorfdaee1XLicg5ZLjhNiajxyD8X78TBrHpnfu3cdtu30apSxDF1PhYcRrw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### G1收集器

G1是目前技术发展的最前沿成果之一，HotSpot开发团队赋予它的使命是未来可以替换掉JDK1.5中发布的CMS收集器。与CMS收集器相比G1收集器有以下特点：

1. **空间整合**，G1收集器采用标记整理算法，不会产生内存空间碎片。分配大对象时不会因为无法找到连续空间而提前触发下一次GC。
2. **可预测停顿**，这是G1的另一大优势，降低停顿时间是G1和CMS的共同关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为N毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，这几乎已经是实时Java（RTSJ）的垃圾收集器的特征了。

上面提到的垃圾收集器，收集的范围都是整个新生代或者老年代，而G1不再是这样。使用G1收集器时，Java堆的内存布局与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔阂了，它们都是一部分（可以不连续）Region的集合。

![图片](http://mmbiz.qpic.cn/mmbiz_jpg/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjoru5HeMdP7OnkFlDIpg71gf6utSNzcoH5E3BpzwZ9ytvFKHBxvf929Dw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

G1的新生代收集跟ParNew类似，当新生代占用达到一定比例的时候，开始出发收集。和CMS类似，G1收集器收集老年代对象会有短暂停顿。

收集步骤：

1、标记阶段，首先初始标记(Initial-Mark),这个阶段是停顿的(Stop the World Event)，并且会触发一次普通Mintor GC。对应GC log:GC pause (young) (inital-mark)

2、Root Region Scanning，程序运行过程中会回收survivor区(存活到老年代)，这一过程必须在young GC之前完成。

3、Concurrent Marking，在整个堆中进行并发标记(和应用程序并发执行)，此过程可能被young GC中断。在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那个这个区域会被立即回收(图中打X)。同时，并发标记过程中，会计算每个区域的对象活性(区域中存活对象的比例)。

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjorMmqL4XiaoBlyRy1ziascfFousWcicSvFxOrFN2GIIgyiagPaWljnbtMzibg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

4、Remark, 再标记，会有短暂停顿(STW)。再标记阶段是用来收集 并发标记阶段 产生新的垃圾(并发阶段和应用程序一同运行)；G1中采用了比CMS更快的初始快照算法:snapshot-at-the-beginning (SATB)。

5、Copy/Clean up，多线程清除失活对象，会有STW。G1将回收区域的存活对象拷贝到新区域，清除Remember Sets，并发清空回收区域并把它返回到空闲区域链表中。

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjornTUQo5RqW79icN3rDRuZ4bl8Y2GU8q7CuM73hJwQpia5Ek52DicIpUDTQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

6、复制/清除过程后。回收区域的活性对象已经被集中回收到深蓝色和深绿色区域。

![图片](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnqLYgY6g5DgUKYUPgXXTjor8XeOMiay8P27IQnWawjlyJdvYriba2ae07zxODmvMOgHqVxCiazYnGia1g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 常用的收集器组合

|       | 新生代GC策略      | 老年老代GC策略 | 说明                                                         |
| :---- | :---------------- | :------------- | :----------------------------------------------------------- |
| 组合1 | Serial            | Serial Old     | Serial和Serial Old都是单线程进行GC，特点就是GC时暂停所有应用线程。 |
| 组合2 | Serial            | CMS+Serial Old | CMS（Concurrent Mark Sweep）是并发GC，实现GC线程和应用线程并发工作，不需要暂停所有应用线程。另外，当CMS进行GC失败时，会自动使用Serial Old策略进行GC。 |
| 组合3 | ParNew            | CMS            | 使用 `-XX:+UseParNewGC`选项来开启。ParNew是Serial的并行版本，可以指定GC线程数，默认GC线程数为CPU的数量。可以使用-XX:ParallelGCThreads选项指定GC的线程数。如果指定了选项 `-XX:+UseConcMarkSweepGC`选项，则新生代默认使用ParNew GC策略。 |
| 组合4 | ParNew            | Serial Old     | 使用 `-XX:+UseParNewGC`选项来开启。新生代使用ParNew GC策略，年老代默认使用Serial Old GC策略。 |
| 组合5 | Parallel Scavenge | Serial Old     | Parallel Scavenge策略主要是关注一个可控的吞吐量：应用程序运行时间 / (应用程序运行时间 + GC时间)，可见这会使得CPU的利用率尽可能的高，适用于后台持久运行的应用程序，而不适用于交互较多的应用程序。 |
| 组合6 | Parallel Scavenge | Parallel Old   | Parallel Old是Serial Old的并行版本                           |
| 组合7 | G1GC              | G1GC           | `-XX:+UnlockExperimentalVMOptions` `-XX:+UseG1GC` #开启； `-XX:MaxGCPauseMillis=50` #暂停时间目标； `-XX:GCPauseIntervalMillis=200` #暂停间隔目标； `-XX:+G1YoungGenSize=512m` #年轻代大小； `-XX:SurvivorRatio=6` #幸存区比例 |

 **引用类型**

强引用(StrongReference)：通常通过 new 来创建一个新对象时返回的引用就是一个强引用，若一个对象通过一系列强引用可到达，它就是强可达的(strongly reachable)，那么它就不被回收。

软引用(SoftReference)：软引用可达的对象在内存不充足时才会被回收，因此软引用要比弱引用“强”一些。

弱引用(WeakReference)：弱引用简单来说就是将对象留在内存的能力不是那么强的引用。使用 WeakReference，垃圾回收器决定引用的对象何时回收并且将对象从内存移除。若一个对象是弱引用可达，无论当前内存是否充足它都会被回收

虚引用(PhantomReference)：虚引用是 Java 中最弱的引用，那么它弱到什么程度呢？通过虚引用无法获取到被引用的对象，虚引用存在的唯一作用就是当它指向的对象被回收后，虚引用本身会被加入到引用队列中，用作记录它指向的对象已被回收。

```java
        // 强引用 对象只要有引用，那么垃圾回收期不会回收它
        Object o = new Object();
        System.out.println("o = " + o);

        // 软引用 如果内存空间不足，垃圾回收期就会回收这个对象
        SoftReference<Object> softReference = new SoftReference<>(new Object());
        System.out.println("softReference = " + softReference.get());

        // 弱引用 一旦被垃圾回收器发现，就会被回收
        WeakReference<Object> weakReference = new WeakReference<>(new Object());
        System.out.println("weakReference = " + weakReference.get());

        // 虚引用 对象在被垃圾回收器回收的时候，能够收到一个系统通知
        ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
        PhantomReference<Object> phantomReference = new PhantomReference<>(new Object(),referenceQueue);
        System.out.println("phantomReference = " + phantomReference.get());
```

 

 

 

 

 

 

 

 

 

 

 

 

 

 

## 类加载

虚拟机把描述类的数据从 Class 文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的 Java 类型，这就是虚拟机的类加载机制。

 

其中类加载的过程包括了加载、验证、准备、解析、初始化五个阶段。在这五个阶段中，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定（也成为动态绑定或晚期绑定）。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

对于初始化阶段，虚拟机规范规定了有且只有 5 种情况必须立即对类进行“初始化”（而加载、验证、准备自然需要在此之前开始）：

1遇到new、getstatic 和 putstatic 或 invokestatic 这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。对应场景是：使用 new 实例化对象、读取或设置一个类的静态字段（被 final 修饰、已在编译期把结果放入常量池的静态字段除外）、以及调用一个类的静态方法。

2，对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。

3，当初始化类的父类还没有进行过初始化，则需要先触发其父类的初始化。（而一个接口在初始化时，并不要求其父接口全部都完成了初始化）

4，虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），
虚拟机会先初始化这个主类。

### 加载

查找并加载类的二进制数据加载时类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：

- 通过一个类的全限定名来获取其定义的二进制字节流。
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 在Java堆中生成一个代表这个类的 `java.lang.Class`对象，作为对方法区中这些数据的访问入口。

相对于类加载的其他阶段而言，加载阶段（准确地说，是加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载。

加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在Java堆中也创建一个 `java.lang.Class`类的对象，这样便可以通过该对象访问方法区中的这些数据。

### 连接

**验证：确保被加载的类的正确性**

验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作：

- **文件格式验证**：验证字节流是否符合Class文件格式的规范；例如：是否以 `0xCAFEBABE`开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
- **元数据验证**：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了 `java.lang.Object`之外。
- **字节码验证**：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
- **符号引用验证**：确保解析动作能正确执行。

验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用 `-Xverifynone`参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

**准备：为类的 `静态变量分`配内存，并将其初始化为默认值**

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

- 1、这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。
- 2、这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、null、false等），而不是被在Java代码中被显式地赋予的值。

假设一个类变量的定义为： `publicstaticintvalue=3`；

那么变量value在准备阶段过后的初始值为0，而不是3，因为这时候尚未开始执行任何Java方法，而把value赋值为3的 `publicstatic`指令是在程序编译后，存放于类构造器 `<clinit>（）`方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行。

> 这里还需要注意如下几点：
>
> - 对基本数据类型来说，对于类变量（static）和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值，而对于局部变量来说，在使用前必须显式地为其赋值，否则编译时不通过。
> - 对于同时被static和final修饰的常量，必须在声明的时候就为其显式地赋值，否则编译时不通过；而只被final修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值。
> - 对于引用数据类型reference来说，如数组引用、对象引用等，如果没有对其进行显式地赋值而直接使用，系统都会为其赋予默认的零值，即null。
> - 如果在数组初始化时没有对数组中的各元素赋值，那么其中的元素将根据对应的数据类型而被赋予默认的零值。

- 3、如果类字段的字段属性表中存在 `ConstantValue`属性，即同时被final和static修饰，那么在准备阶段变量value就会被初始化为ConstValue属性所指定的值。

假设上面的类变量value被定义为： `publicstaticfinalintvalue=3`；

编译时Javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据 `ConstantValue`的设置将value赋值为3。我们可以理解为static final常量在编译期就将其结果放入了调用它的类的常量池中

**解析：把类中的符号引用转换为直接引用**

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。符号引用就是一组符号来描述目标，可以是任何字面量。

直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

### **初始化**

初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。在Java中对类变量进行初始值设定有两种方式：

- ①声明类变量是指定初始值
- ②使用静态代码块为类变量指定初始值

JVM初始化步骤

- 1、假如这个类还没有被加载和连接，则程序先加载并连接该类
- 2、假如该类的直接父类还没有被初始化，则先初始化其直接父类
- 3、假如类中有初始化语句，则系统依次执行这些初始化语句

类初始化时机：只有当对类的主动使用的时候才会导致类的初始化，类的主动使用包括以下六种：

- 创建类的实例，也就是new的方式
- 访问某个类或接口的静态变量，或者对该静态变量赋值
- 调用类的静态方法
- 反射（如 `Class.forName(“com.shengsiyuan.Test”)`）
- 初始化某个类的子类，则其父类也会被初始化
- Java虚拟机启动时被标明为启动类的类（ `JavaTest`），直接使用 `java.exe`命令来运行某个主类

**结束生命周期**

在如下几种情况下，Java虚拟机将结束生命周期

- 执行了 `System.exit()`方法
- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而导致Java虚拟机进程终止

 

## 类加载器

把实现类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作的代码模块称为“类加载器”。

![å¾ç](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnokxXiapDdvntH8PGwa0zGXM7qXKib1ibsib1BuyLxjoP1sgorwib78yTD4896N5r1AibdhDXTHZ7z9VyBg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

继承ClassLoader,重写findClass

**JVM类加载机制**

- **全盘负责**，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
- **父类委托**，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
- **缓存机制**，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效

 

## 双亲委派模型

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。

### New 一个对象过程发生了什么

1，确认类元信息是否存在。当 JVM 接收到 new 指令时，首先在 metaspace 内检查需要创建的类元信息是否存在。 若不存在，那么在双亲委派模式下，使用当前类加载器以 ClassLoader + 包名＋类名为 Key 进行查找对应的 class 文件。 如果没有找到文件，则抛出 ClassNotFoundException 异常 ， 如果找到，则进行类加载（加载 - 验证 - 准备 - 解析 - 初始化），并生成对应的 Class 类对象。

2，分配对象内存。 首先计算对象占用空间大小，如果实例成员变量是引用变量，仅分配引用变量空间即可，即 4 个字节大小，接着在堆中划分—块内存给新对象。 在分配内存空间时，需要进行同步操作，比如采用 CAS (Compare And Swap) 失败重试、 区域加锁等方式保证分配操作的原子性。

3，设定默认值。 成员变量值都需要设定为默认值， 即各种不同形式的零值。

4，设置对象头。设置新对象的哈希码、 GC 信息、锁信息、对象所属的类元信息等。这个过程的具体设置方式取决于 JVM 实现。

5，执行 init 方法。 初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量。