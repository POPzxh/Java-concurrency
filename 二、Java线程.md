# 二、Java线程

### 一、创建线程

Java中创建线程**Thread**类：`ThTread thread = new Thread();`创建一个线程。再调用其**start()**方法执行该线程`thread.start();`。

但是，我们并没有为线程编写运行代码。在Java中编写线程运行时执行的代码有两种方式：

* 创建**Thread**子类的实例，并重写**run**方法
* 在创建Thread实例的时候实现**Runnable**接口

### 二、创建Thread子类

创建**Thread子类**的一个实例并**重写run**方法，**run**方法会在调用**start()**方法之后被执行。例子如下：

```java
public class MyThread extends Thread{
	@Override
    public void run() {
        for(int i = 0 ; i<10000 ; i++)
            System.out.println("This is a Thread");
    }
}
```

也可以创建一个Thread的匿名子类：

```java
Thread thread = new Thread(){
	@Override
    public void run() {
        for(int i = 0 ; i<10000 ; i++)
            System.out.println("This is a Thread");
    }
};
thread.start();
```

### 三、实现Runnable接口

第二种编写线程执行代码的方式是新建一个实现了java.lang.Runnable接口的类的实例，实例中的方法可以被线程调用。下面给出例子：

```java
public class MyThread implements Runnable{
	@Override
    public void run() {
        for(int i = 0 ; i<10000 ; i++)
            System.out.println("This is a Thread");
    }
}
```

为了使线程能够执行run()方法，需要在Thread类的构造函数中传入 MyRunnable的实例对象。示例如下：

```java
Thread thread = new Thread(new MyRunnable());
thread.start();
```

同样也可以创建**Runnable**匿名类（使用lambda表达式）

```java
Runnable r = ()->{
	 for(int i = 0 ; i<10000 ; i++)
            System.out.println("This is a Thread");
};
new Thread(r).start();
```

### 四、常见错误：**调用run()**方法而非**start()**方法

创建并运行一个线程所犯的常见错误是调用线程的run()方法而非start()方法，如下所示：

```java
Thread newThread = new Thread(()->System.out.print("New Thread"));
newThread.run();//should be start()
```

但是，实际上执行了线程中的任务（run函数部分），而启动了一个新的线程。调用`Thread.start()`将会创建一个执行``run``方法的新线程。

### 五、线程名

当创建线程时，可以通过构造器给线程起一个名字：

```java
Thread thread = new Thread(new myThread() , "Thread 1");
```

通过**Thread**类的静态方法**currentThread()**可以的到当前运行的线程对象的引用。在通过**Thread.getName()**既可得到当前执行线程的名称。

* 这里是一个小小的例子。首先输出执行main()方法线程名字。这个线程JVM分配的。然后开启10个线程，命名为1~10。每个线程输出自己的名字后就退出。

```java
System.out.println(Thread.currentThread().getName());
        for(int i = 0 ; i < 10 ; i ++)
            new Thread(()->
                    System.out.println("ThreadName："+Thread.currentThread().getName()+" is running!")
                    ,"Thread " + String.valueOf(i)).start();
```

* 需要注意的是，尽管启动线程的顺序是有序的，**但是执行的顺序并非是有序的**。也就是说，1号线程并不一定是第一个将自己名字输出到控制台的线程。这是因为线程是并行执行而非顺序的。

### 六、中断线程

当线程执行到最后return，或者执行时出现了异常，线程将会终止。

* 没有可以**强制终止**线程的方法。
* 可以通过**interrupt()**方法请求终止线程。

#### `void interrupt()`：

向线程发送中断请求。线程的**中断状态**将被设置为**true**。此外，如果线程处于阻塞状态，调用`interruput()`方法可以使线程抛出`InterruptedException`异常。所以，可以使用`interrupt()`方法来**中断一个处于阻塞状态的线程**。

当一个线程的**中断状态**被设置为**true**后。线程也**不会立刻中断**。除非线程主动检测，通过**interrupted()**或者**isInterrupted()**方法，来检测状态为，并处理\退出。

#### `static boolean interrupted()`:

Thread类的静态方法，返回当前执行这一条命令的线程的**中断状态**。但是，调用这一方法后，该线程的状态为会**重置为false**。

#### `boolean is Interrupted()`:

检测线程是否被终止。**不会重置状态位**。

### 七、线程状态

**一个线程可以有如下6种状态**：

* new（新创建）
* Runnable（可运行）
* Blocked（被阻塞）
* Waiting（等待）
* Timed Waiting（计时等待）
* Terminated（被终止）

#### 1.新创建线程

当**new**创建一个新线程使，该线程还没有开始运行。这意味着该线程的当前状态是**New**。

#### 2.可运行线程

当一个线程调用里**start**方法后，即进入了runnable（可运行状态）。但是，这不代表该线程**一定在运行**。而且，一旦一个线程开始运行之后，它**不必始终保持运行**。所以，一个可运行线程**既有可能在运行也有可能不在运行**。（这取决于操作系统）

#### 3.被阻塞（blocked）线程 和 等待（waiting，timed-waiting）线程

当线程处于**阻塞或者等待**时：它暂时不活动，不运行任何代码且消耗最少的资源。直到线程调度器再次激活它，细节取决于是如何进入到**非活动状态的**

* 阻塞（Blocked）：线程企图取得的**对象的锁**被其它线程持有，从而进入**阻塞状态**。所以，当其它线程释放该锁，并且线程调度器允许持有时，将进入非阻塞状态。
* 等待（Waiting）：线程持有锁后，但需要等待其它线程执行某些操作时进入。（wait，join，park）
* 计时等待（Timed Waiting）：调用超时参数的方法：Thread.sleep

#### 4.被终止的线程 Terminated

* run方法正常退出而自然死亡
* 没有捕获的异常终止了run方法

![img](https://images0.cnblogs.com/blog/288799/201409/061045374695226.jpg)

### 八、线程属性

#### 8.1 线程优先级

每一个线程可以拥有优先级，通过`setPriority()`方法设置。默认情况下，一个线程继承其父线程的优先级。（主线程默认是5）。

* 优先级从**MIN_PRIORITY（1）**到**MAX_PRIORITY（10）**之间。优先级越高，线程调度器有机会选择线程是会**选择具有高优先级的线程**。
* 线程优先级**高度依赖于系统**：会映射到系统的优先级。
* `static void yield()`：静态方法，让当前执行线程处于让步状态。如果有其它可运行线程的优先级>=此线程，那么将会被调度。

#### 8.2 守护线程

守护线程：可以看作一种特殊的用户线程，为其他线程提供服务。在一个线程中new一个新的线程并且使用`setDaemon(true)`方法将其设置位守护线程。

* 当被守护的线程结束后，守护线程也会被杀死。

#### 8.3 未捕获异常处理器

在线程的run方法中是不能抛出任何**受查异常**的。所以，需要使用**try/catch**来捕获异常。对于非受查异常，会导致线程的终止。除了同样使用**catch**捕获以外，可以在线程死亡之前将异常传递到一个**未捕获异常处理器**

* **为线程添加*未捕获异常处理器***

  1. 为所用线程添加默认的异常捕获器

     `static void setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler handler)`

  2. 为一个线程添加一个异常捕获器` void setUncaughtExceptionHandler(UncaughtExceptionHandler eh)`

参数是一个实现`UncaughtExceptionHandler`接口的对象。这个接口只有一个方法：`uncaughtException(Thread t, Throwable e);`处理传入的异常。

### 九、同步

当一个操作的正确性取决于线程间的时序时$\to$*竞争条件（race condition）*

#### 9.1 竞争条件的例子

100个银行账户（100个线程）一直相互转账，随机的转账给另一个账户若干金额。

```java
Runnable r = () -> {
   try
   {
      while (true)
      {
         int toAccount = (int) (bank.size() * Math.random());
         double amount = MAX_AMOUNT * Math.random();
         bank.transfer(fromAccount, toAccount, amount);
         Thread.sleep((int) (DELAY * Math.random()));
      }
   }
   catch (InterruptedException e)
   {
   }            
};
Thread t = new Thread(r);
t.start();
```

```java
public void transfer(int from, int to, double amount)
{
   if (accounts[from] < amount) return;
   System.out.print(Thread.currentThread());
   accounts[from] -= amount;
   System.out.printf(" %10.2f from %d to %d", amount, from, to);
   accounts[to] += amount;
   System.out.printf(" Total Balance: %10.2f%n", getTotalBalance());
}
```

我们发现，在不断的转账过程中，会出现错误：总金额或多或少。

```java
hread[Thread-21,5,main]     621.11 from 21 to 45 Total Balance:  100000.00
Thread[Thread-7,5,main]Thread[Thread-45,5,main]     416.93 from 45 to 98 Total Balance:   99418.00
Thread[Thread-6,5,main]     151.25 from 6 to 30 Total Balance:   99418.00
Thread[Thread-19,5,main]     181.31 from 19 to 44 Total Balance:   99418.00
Thread[Thread-65,5,main]     110.21 from 65 to 6 Total Balance:   99418.00
Thread[Thread-55,5,main]     643.36 from 55 to 67 Total Balance:   99418.00
```

那么这是为什么呢？

#### 9.2 竞争条件详解

如果有两个线程同时执行了`accounts[to] += amount`操作，指令被如下处理：

1. 将account[to]加载到寄存器
2. 增加account
3. 将结果写回到accounts[to]

如果，现在线程1执行完1、2步，但是被剥夺了运行全。线程2此时被调度器唤醒完成所有3步修改了account[to]的值。然后，线程1完成了第三步。显然，线程2的写入操作被覆盖了。

显然，这个操作并非是**线程安全的**：当一个类，不断被多个线程调用，仍能表现出正确的行为时，那它就是线程安全的。

* 原子操作：一个操作时不分割的。显然，如上操作并不是**原子的**，当一个操作是非原子时，会导致线程的不安全。

  >假定有两个操作A和B，如果从执行A的线程来看，当线程B执行时，要么将B全部执行完，要么就完全不执行B，那么对A和B来说彼此的操作是**原子的**

* 复合操作

  相对于**原子操作**，可以被分割的操作被称为**复合操作**。例如，“读取-修改-写入”和“先检查后执行”。所以，如果操作为**复合操作**，显然会产生竞争条件，即不能确保线程安全。所以，为了使如上的**复合操作**变为线程安全，我们必须以**原子的方式执行他们**。

* 原子方式执行：

  * 1）加锁机制
  * 2）使用线程安全类（Java提供的原子类）

一下章，我们将重点先看看Java提供的原子类：`java.util.concurrent.atomic`