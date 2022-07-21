# 并发编成

### 1，多线程基础

####              1.1，进程与线程

- 进程：一个运行的程序并分配内存空间，

- 线程：由进程创建，一个进程可以创建多个线程

- 并发：同一个时刻，多个任务交替执行，时间分片

- 并行：同一个时刻，多个任务同时进行


#### 		1.2，创建线程的方式

- 继承Thread

  ```java
  class Thread1 extends Thread {
      @Override
      public void run() {
          System.out.println("one -> 线程开始运行 ");
  
          int count = 0;
          while (count <= 80){
              try {
                  System.out.println("one -> run " + count);
                  sleep(10);
                  count++;
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```

- 实现Runable

  ```java
  class Thread2 implements Runnable{
      @Override
      public void run() {
          System.out.println("2 -> 线程开始运行 ");
  
          int count = 0;
          while (count <= 80){
              try {
                  System.out.println("2 -> run " + count);
                  Thread.sleep(10);
                  count++;
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```

- 实现Callable

  ```java
  class Thread3 implements Callable<String>{
  
      // 有返回值，可以抛出异常
      @Override
      public String call() throws Exception {
          System.out.println("one -> 线程开始运行 ");
  
          int count = 0;
          while (count <= 80){
              try {
                  System.out.println("one -> run " + count);
                  Thread.sleep(1000);
                  count++;
              } catch (InterruptedException e) {
                  throw e;
              }
          }
          return "0";
      }
  }
  
  Thread3 call = new Thread3();
  FutureTask<String> futureTask = new FutureTask<>(call);
  new Thread(futureTask,"call").start();
  String s = futureTask.get();
  System.out.println("s = " + s);
  ```

- 线程池

  ```java
  ExecutorService pool = Executors.newSingleThreadExecutor();
  pool.submit(new Thread2());
  ```

  > java无法操作硬件，无法开启线程，是通过调用native本地方法开启的
	
	```java
	public synchronized void start() {
	    if (threadStatus != 0)
	        throw new IllegalThreadStateException();
	    group.add(this);
	
	    boolean started = false;
	    try {
	        start0();
	        started = true;
	    } finally {
	        try {
	            if (!started) {
	                group.threadStartFailed(this);
	            }
	        } catch (Throwable ignore) {
	        }
	    }
	}
	
	private native void start0();
	```
#### 		1.3，线程的状态



![image-20220422010909524](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220422010909524.png)

- NEW:  尚未启动的线程处于此状态
- RUNABLE： 在Java虚拟机中执行的线程处于此状态,又分为 readly和runing状态
- BLOCKED： 被阻塞等待监视器锁定的线程处于此状态。
- WAITING： 正在等待另一个线程执行特定动作的线程处于此状态
- TIMED_WAITING： 正在等待另一个线程执行动作达到指定等待时间的线程处于此状态
- TERMINATED： 已退出的线程处于此状态

#### 1.4，常用方法

​		![image-20220422011307637](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220422011307637.png)

### 2，java内存模型（JMM）

#### 2.1 基础理解

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X2pwZy91Q2htZWVYMUZweUI2V2tNVEwySVVhcGZUdEdINkZGT2lhbGNvRDlJUlhINWpVc2tNaWFIZlBkOEFLbVdaWUdBZXJMSDNicHVsRjcySVlocksya3VpYTlQZy82NDA?x-oss-process=image/format,png)

通常情况下，当一个CPU需要读取主存时，它会将主存的部分读到CPU缓存中。它甚至可能将缓存中的部分内容读到它的内部寄存器中，然后在寄存器中执行操作。当CPU需要将结果写回到主存中去时，它会将内部寄存器的值刷新到缓存中，然后在某个时间点将值刷新回主存。

**可见性，原子性，有序性**

#### 2.2，内存交互协议

- lock:  主内存变量，把一个变量标识为某个线程独占状态。
- unlock：主内存变量，把一个处于锁定状态变量释放出来，被释放后的变量才可以被其他线程锁定。
- read：主内存变量，把一个变量的值从主内存传输到线程的工作内存中，以便锁喉的load动作使用
- load：工作内存变量，把read读取到的主内存中的变量放入工作内存的变量副本中。
- use：工作内存变量，把工作内存中变量的值传递给java虚拟机执行引擎，每当虚拟机遇到一个需要使用到变量值的字节码指令时，将会执行该操作
- assign：工作内存变量，把从执行引擎收到的变量的值复制给工作变量，每当虚拟机遇到一个给变量复制的字节码是，将会执行该操作
- store：工作内存变量，把工作内存中要给变量的值传送到主内存中，一遍随后的write使用。
- write：主内存变量，把store操作从工作内存中得到的变量值放入主内存的变量中。

#### 2.3，指令优化

​	![image-20220422223848074](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220422223848074.png)

**as-if-serial原则**

​	不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器，runtime 和处理器都必须遵守as-if-serial语义

**happens-before原则**

- 程序顺序规则：如果程序中操作A在操作B之前，那么线程中A操作将在B操作之前执行
- 监视器锁规则：在监视器锁上的解锁操作必须在同一监视器锁上的加锁操作之前执行
- volatile变量规则：对volatile变量的写入操作必须对该变量的读操作之前执行
- 线程启动规则：Thread.Start的调用必须在该线程中执行任何操作之前执行
- 线程结束规则：线程中任何操作都必须在其他线程检测到该线程已经结束之前执行，如从Thread.join中成功返回，或者在调用Thread.isAlive时返回false。
- 中断规则：当一个线程在另一个线程上调用interrupt时，必须在被中断线程检测到interrupt调用之前执行（通过抛出InterruptedException，或者调用isInterrupted和interrupted）。
- 终结器规则：对象的构造函数必须在启动该对象的终结器之前执行完成
- 传递性：如果操作A在操作B之前完成，并且操作B在操作C之前执行，那么操作A必须在操作C之前执行



### 3，锁

#### 3.1，基础解释

```java
//锁定方法
public synchronized void sale(){
    number -= 1;
}

public int getNumber() {
    //锁定的是对象引用 
    synchronized (this){
        number++;
    }
    return number;
}

public void setNumber(int number) {
    // 这里锁定的class类 jvm中只存在一个  效果等同于static方法
    synchronized (Ticket.class){
        number--;
    }
    this.number = number;
}

```

> 锁的升级过程

![img](https://img-blog.csdnimg.cn/img_convert/0894ed62c588d7ad6dbd4640d407e356.png)

#### 3.2，AQS

AQS 的全称为（AbstractQueuedSynchronizer）

![image-20220422230346875](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220422230346875.png)

![preview](https://pic1.zhimg.com/v2-dca2f36e89cdbbe56b896a107c66cd8c_r.jpg)

```java
private transient volatile Node head;

/**
 * Tail of the wait queue, lazily initialized.  Modified only via
 * method enq to add new wait node.
 */
private transient volatile Node tail;

/**
 * 原子状态
 */
private volatile int state;
```

#### 3.3，CAS

全称compareAndSwap 比较交换

```java
public final int getAndSetInt(Object var1, long var2, int var4) {
    int var5;
    //自旋
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var4));

    return var5;
}
```

引发ABA问题，可以使用添加版本号处理

#### 3.4，原子类

![image-20220422233207548](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220422233207548.png)	

![image-20220422233543285](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220422233543285.png)

### 4，locks

#### 4.1，ReentrantLock

可重入锁，基于AQS 实现手动锁，

```java
/**
 * 默认是非公平锁
 * 公平锁：按照AQS的CHL队列顺序执行的
 * 非公平锁：不按照AQS的CHL队列顺序执行，每次加入都会去竞争
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}

// 非公平锁 首先通过CAS去竞争
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

//锁定方法
public void sale(){
    ReentrantLock lock = new ReentrantLock();
    lock.lock();
    try {
        number -= 1;
    } finally {
        lock.unlock();
    }
}
```

#### 4.2，Condition

![image-20220422235540292](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220422235540292.png)

可以实现精准线程通知

```java
public void sale(){
    ReentrantLock lock = new ReentrantLock();
    Condition condition  = lock.newCondition();
    lock.lock();
    try {
        number -= 1;
        condition.await();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
        condition.signal();
    }
}
```

4.3，ReadWriteLock 读写锁

4.4，CountDownLatch  减法锁

![](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220423000348633.png)

4.5，CyclicBarrier  加法锁

![image-20220423000742539](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220423000742539.png)

#### 4.6，Semaphore 

一个计数信号量。 在概念上，信号量维持一组许可证。 如果有必要，每个acquire()都会阻塞，直到许可证可用，然后才能使用它。 每个release()添加许可证，潜在地释放阻塞获取方。 但是，没有使用实际的许可证对象; Semaphore只保留可用数量的计数，并相应地执行。

信号量通常用于限制线程数，而不是访问某些（物理或逻辑）资源。

此类的构造函数可选择接受公平参数。 当设置为false时，此类不会保证线程获取许可的顺序。 特别是， 闯入是允许的，也就是说，一个线程调用acquire()可以提前已经等待线程分配的许可证-在等待线程队列的头部逻辑新的线程将自己。 当公平设置为真时，信号量保证调用acquire方法的线程被选择以按照它们调用这些方法的顺序获得许可（先进先出; FIFO）。 请注意，FIFO排序必须适用于这些方法中的特定内部执行点。 因此，一个线程可以在另一个线程之前调用acquire ，但是在另一个线程之后到达排序点，并且类似地从方法返回。 另请注意， 未定义的tryAcquire方法不符合公平性设置，但将采取任何可用的许可证。

 

通常，用于控制资源访问的信号量应该被公平地初始化，以确保线程没有被访问资源。 当使用信号量进行其他类型的同步控制时，非正常排序的吞吐量优势往往超过公平性。

### 5，并发集合

5.1，ConcurrentHashMap

5.2，CopyOnWriteArrayList

5.3，CopyOnWriteArraySet

5.4，阻塞队列 BlockingQueue

​		ArrayBlockingQueue 基于数组的有限队列  单向的

​		LinkedBlockingQueue 基于链表的无界队列 单向

### 6，线程池

#### 6.1，原理

![img](https://img2018.cnblogs.com/blog/632381/201907/632381-20190708111039970-1535445943.png)

1. 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
2. 如果此时线程池中的数量等于corePoolSize，但是缓冲队列workQueue未满，那么任务被放入缓冲队列。
3. 如果此时线程池中的数量大于等于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。
4. 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。
5. 当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。

#### 6.2，参数详解

| 参数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| corePoolSize    | 核心线程数量，线程池维护线程的最少数量                       |
| maximumPoolSize | 线程池维护线程的最大数量                                     |
| keepAliveTime   | 线程池除核心线程外的其他线程的最长空闲时间，超过该时间的空闲线程会被销毁 |
| unit            | keepAliveTime的单位，TimeUnit中的几个静态属性：NANOSECONDS、MICROSECONDS、MILLISECONDS、SECONDS |
| workQueue       | 线程池所使用的任务缓冲队列                                   |
| threadFactory   | 线程工厂，用于创建线程，一般用默认的即可                     |
| handler         | 线程池对拒绝任务的处理策略                                   |

当线程池任务处理不过来的时候（什么时候认为处理不过来后面描述），可以通过handler指定的策略进行处理，ThreadPoolExecutor提供了四种策略：

1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常；也是默认的处理方式。
2. ThreadPoolExecutor.DiscardPolicy：丢弃任务，但是不抛出异常。
3. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
4. ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

可以通过实现RejectedExecutionHandler接口自定义处理方式。

#### 6.3，常用队列介绍

1. ArrayBlockingQueue： 这是一个由数组实现的容量固定的有界阻塞队列.
2. SynchronousQueue： 没有容量，不能缓存数据；每个put必须等待一个take; offer()的时候如果没有另一个线程在poll()或者take()的话返回false。
3. LinkedBlockingQueue： 这是一个由单链表实现的默认无界的阻塞队列。LinkedBlockingQueue提供了一个可选有界的构造函数，而在未指明容量时，容量默认为Integer.MAX_VALUE。

　　队列操作:

| 方法    | 说明                                                   |
| ------- | ------------------------------------------------------ |
| add     | 增加一个元索; 如果队列已满，则抛出一个异常             |
| remove  | 移除并返回队列头部的元素; 如果队列为空，则抛出一个异常 |
| offer   | 添加一个元素并返回true; 如果队列已满，则返回false      |
| poll    | 移除并返回队列头部的元素; 如果队列为空，则返回null     |
| put     | 添加一个元素; 如果队列满，则阻塞                       |
| take    | 移除并返回队列头部的元素; 如果队列为空，则阻塞         |
| element | 返回队列头部的元素; 如果队列为空，则抛出一个异常       |
| peek    | 返回队列头部的元素; 如果队列为空，则返回null           |
