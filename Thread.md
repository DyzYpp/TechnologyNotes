# Thread

### 1. 什么是进程,什么是线程

**进程**

**一个程序运行起来就是一个进程**

**线程**

**一个程序运行起来后最小的一个执行单元就是一个线程**

### 2.创建线程的方式

#### 2.1  继承Thread类

```java
public class TestThread1 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println("子线程"+i);
        }
    }

    public static void main(String[] args) {

        TestThread1 testThread1 = new TestThread1();
        testThread1.start();

        for (int i = 0; i < 1000; i++) {
            System.out.println("主线程"+i);
        }
    }
}
```

**创建线程类:  继承Thread类,重写Run方法**

**启动该线程: 创建线程类对象,调用对象的Stop方法**

------

#### 2.2  实现Runnable接口

```java
public class TestThread3 implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println("子线程"+i);
        }
        System.out.println("实现Runable接口创建线程");
    }

    public static void main(String[] args) {
        TestThread3 testThread3 = new TestThread3();
        Thread thread = new Thread(testThread3,"线程1");
        System.out.println(thread.getName());
        thread.start();

        for (int i = 0; i < 1000; i++) {
            System.out.println("主线程"+i);
        }
    }
}
```

**创建线程类: 实现Runnable接口,重写Run方法**

**启动该线程: 首先创建类对象,在创建Thread类对象,调用Thread类对象的start()方法,将类对象作为参数传递进去**

------

#### 2.3  实现Callable<泛型> 接口 （可获取返回值,返回值类型支持自定义）

```java
public class TestCallable1 implements Callable<Boolean> {
    private String name;

    public TestCallable1(String name) {
        this.name = name;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        TestCallable1 testThread1 = new TestCallable1("张三");
        TestCallable1 testThread2 = new TestCallable1("李四");
        TestCallable1 testThread3 = new TestCallable1("王五");

        // 创建执行服务
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        Future<Boolean> submit = executorService.submit(testThread1);
        Future<Boolean> submit1 = executorService.submit(testThread2);
        Future<Boolean> submit2 = executorService.submit(testThread3);

        // 三个线程的执行顺序由cpu调度安排,人为无法干预
        Boolean aBoolean = submit.get();
        Boolean aBoolean1 = submit1.get();
        Boolean aBoolean2 = submit2.get();

        System.out.println(aBoolean);
        System.out.println(aBoolean1);
        System.out.println(aBoolean2);

        executorService.shutdown();
    }

    @Override
    public Boolean call() throws Exception {
        System.out.println(name);
        return true;
    }
}
```

**实现Callable<T>并指定泛型,重写Call()方法后,可获得返回值,返回值类型与Callable<T> 指定的泛型一致.**

### 3. 停止线程

```java
// 建议线程正常停止,不建议死循环
    // 建议使用标志位,设置一个标志位
    // 不要使用stop或destroy等过时方法
public class TestSTop implements Runnable{

    // 设置一个标志位
    private boolean flag = true;

    @Override
    public void run() {
        int i = 0;
        while (flag){
            System.out.println("---"+i++);
        }
    }

    public void stop(){
        this.flag = false;
        System.out.println("线程停止了");
    }

    public static void main(String[] args) {
        TestSTop testSTop = new TestSTop();
        new Thread(testSTop).start();

        for (int i = 0; i < 100; i++) {
            System.out.println("+++"+ i);
            if (i == 88){
                testSTop.stop();
                break;
            }
        }
    }
}
```

**停止线程建议使用标志位或程序正常执行完毕结束线程**

**不建议使用死循环,或stop,destroy等已经过时的方法**

### 4.  线程常用的方法

#### 4.1  线程休眠 sleep()

```java
// 模拟倒计时
public class TestSleep2{
    public static void main(String[] args) {
//        tenDown();

        Date startTime = new Date(System.currentTimeMillis());
        while (true){
            try {
                Thread.sleep(1000);
                System.out.println(new SimpleDateFormat("HH:mm:ss").format(startTime));
                startTime = new Date(System.currentTimeMillis());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void tenDown(){
        int num = 10;
        while (true){
            try {
                Thread.sleep(1000);
                System.out.println(num--);
                if (num <= 0){
                    break;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
// 模拟网络延时,放大问题的发生性
public class TestSleep implements Runnable {
    private int nums = 10;

    @Override
    public void run() {
        while (true){
            if (nums <= 0){
                break;
            }
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"----"+nums--+"票");
        }
    }

    public static void main(String[] args) {
        TestThread4 thread4 = new TestThread4();

        new Thread(thread4,"张三").start();
        new Thread(thread4,"李四").start();
        new Thread(thread4,"王五").start();
    }
}
```

**sleep()方法主要用于网络延时,倒计时等等**

#### 4.2    线程插队 join()

```java
// 测试join方法  插队
public class TestJoin implements Runnable{


    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("---"+i);
        }
    }

    public static void main(String[] args) {
        TestJoin testJoin = new TestJoin();
        Thread thread = new Thread(testJoin);
        thread.start();

        for (int i = 0; i < 100; i++) {
            if (i == 50){
                try {
                    thread.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("+++"+i);
        }
    }
}
```

#### 4.3  礼让线程  yield()

```java
// 礼让线程
public class TestYield {
    public static void main(String[] args) {
        MyYield myYield = new MyYield();
        new Thread(myYield,"a").start();
        new Thread(myYield,"b").start();
    }
}

class MyYield implements Runnable{
    @Override
    public void run() {
        System.out.println("开始执行"+Thread.currentThread().getName());
        Thread.yield();
        System.out.println("执行结束"+Thread.currentThread().getName());
    }
}
```

**如代码所示,调用线程的yield()方法后，a线程和b线程在都输出启动后才会结束.不会出现a启动->a技术,b启动->b结束;**

------

### 5. 什么是脏读

```java
public class DutyRead {

    public static void main(String[] args) {
    Account account = new Account();
        try {
            new Thread(()->{
                account.set("zhangsan",100.0);
            }).start();
            Thread.sleep(1000);
            System.out.println(account.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            Thread.sleep(2000);
            System.out.println(account.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Account{

    private double balance;

    private String name;

    public synchronized  void set(String name,double balance){
        this.name = name;
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.balance = balance;
    }

    public synchronized  double get(){
        return this.balance;
    }
}
```

**get方法若不加synchronized关键字,则在线程休眠时,就会调用get方法读出balance数据.**

------



































# synchronized底层实现

#### 1.简介

**在JDK1.5中,synchronized的性能是比较差的,低效. 影响性能的主要原因是阻塞的实现,挂起线程和恢复线程的操作都需要转入内核态(通过操作系统)来完成,这些操作会给操作系统的并发性带来很大的压力.相比之下使用Java 提供的Lock对象，性能更高一些。**

```java
class MyThread extends Thread{
    private Lock lock = new ReentrantLock();
    private int ticket = 100;

    @Override
    public void run() {
        for(int i = 0;i < 100;i++){
            lock.lock();
            if(this.ticket > 0){
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"还剩下"+this.ticket--+"票");
            }
            lock.unlock();
        }
    }
}

public class TestThread {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        new Thread(myThread,"黄牛1").start();
        new Thread(myThread,"黄牛2").start();
        new Thread(myThread,"黄牛3").start();
    }
}

```

**到JDK1.6之后,对于synchronized进行了优化,有了<span style="color:red">锁消除，锁粗化，偏向锁，自旋、轻量级锁，最后到重量级锁</span>.性能方面已经与Lock无很大差距.synchronized最大的特征就是在同一时刻只有一个线程能够获得对象监视器(monitor),从而进入到同步代码块或同步方法之中,即表现为(互斥性),这种方式保证了线程的安全.但是每次只能通过一个线程,所以在保证了安全的前提下,如何在提升程序的执行效率呢?** 

**打个比方，去收银台付款，之前的方式是，大家都去排队，然后取纸币付款收银员找零，有的时候付款的时候在包里拿出钱包再去拿出钱，这个过程是比较耗时的，然后，支付宝解放了大家去钱包找钱的过程，现在只需要扫描下就可以完成付款了，也省去了收银员跟你找零的时间的了。同样是需要排队，但整个付款的时间大大缩短，是不是整体的效率变高速率变快了？这种优化方式同样可以引申到锁优化上，缩短获取锁的时间。**

------

#### 2. 锁的四种状态

![synchronized锁升级过程](Thread.assets/synchronized%E9%94%81%E5%8D%87%E7%BA%A7%E8%BF%87%E7%A8%8B-1595479905921.png)





![未命名文件](Thread.assets/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)



##### 2.1 <span style="color:brown">无锁</span>

**无锁是指没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。**

**无锁的特点是修改操作会在循环内进行，线程会不断的尝试修改共享资源。如果没有冲突就修改成功并退出，否则就会继续循环尝试。如果有多个线程修改		同一个值，必定会有一个线程能修改成功，而其他修改失败的线程会不断重试直到修改成功。**

##### 2.2<span style="color:brown"> 偏向锁</span>

**偏向锁是指当一段同步代码一直被同一个线程所访问时，即不存在多个线程的竞争时，那么该线程在后续访问时便会自动获得锁，从而降低获取锁带来的消		耗，即提高性能。**

**当一个线程访问同步代码块并获取锁时，会在 Mark Word 里存储锁偏向的线程 ID。在线程进入和退出同步块时不再通过 CAS 操作来加锁和解锁，而是检		测 Mark Word 里是否存储着指向当前线程的偏向锁。轻量级锁的获取及释放依赖多次 CAS 原子指令，而偏向锁只需要在置换 ThreadID 的时候依赖一次 		CAS 原子指令即可。**

**偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程是不会主动释放偏向锁的。**

**关于偏向锁的撤销，需要等待全局安全点，即在某个时间点上没有字节码正在执行时，它会先暂停拥有偏向锁的线程，然后判断锁对象是否处于被锁定状态   		如果线程不处于活动状态，则将对象头设置成无锁状态，并撤销偏向锁，恢复到无锁（标志位为01）或轻量级锁（标志位为00）的状态。**

**偏向锁在 JDK 6 及之后版本的 JVM 里是默认启用的。可以通过 JVM 参数关闭偏向锁：-XX:-UseBiasedLocking=false，关闭之后程序默认会进入轻量级		锁状态。**

##### 2.3 <span style="color:brown">轻量级锁</span>

**轻量级锁的获取主要由两种情况：① 当关闭偏向锁功能时；② 由于多个线程竞争偏向锁导致偏向锁升级为轻量级锁。**

**在代码进入同步块的时候，如果同步对象锁状态为无锁状态，虚拟机将首先在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁		对象目前的 Mark Word 的拷贝，然后将对象头中的 Mark Word 复制到锁记录中。**

**拷贝成功后，虚拟机将使用 CAS 操作尝试将对象的 Mark Word 更新为指向 Lock Record 的指针，并将 Lock Record 里的 owner 指针指向对象Mark 		Work**

**如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象 Mark Word 的锁标志位设置为“00”，表示此对象处于轻量级锁定状态。**

**如果轻量级锁的更新操作失败了，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就		可以直接进入同步块继续执行，否则说明多个线程竞争锁。**

**若当前只有一个等待线程，则该线程将通过自旋进行等待。但是当自旋超过一定的次数时，轻量级锁便会升级为重量级锁（锁膨胀）。**

**自旋10次或CPU超过50%(锁膨胀)**

##### 2.4 <span style="color:brown">重量级锁</span>

**重量级锁是指当有一个线程获取锁之后，其余所有等待获取该锁的线程都会处于阻塞状态。**

**重量级锁通过对象内部的监视器（monitor）实现，而其中 monitor 的本质是依赖于底层操作系统的 Mutex Lock 实现，操作系统实现线程之间的切换需		要从用户态切换到内核态，切换成本非常高。**

**简言之，就是所有的控制权都交给了操作系统，由操作系统来负责线程间的调度和线程的状态变更。而这样会出现频繁地对线程运行状态的切换，线程的挂		起和唤醒，从而消耗大量的系统资源，导致性能低下。**

##### 2.5 <span style="color:brown">总结</span>

**偏向锁通过对比 Mark Word 解决加锁问题，避免执行CAS操作。**

**轻量级锁是通过用 CAS 操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。**

**重量级锁是将除了拥有锁的线程以外的线程都阻塞。**

------

#### 3. **CAS（Compare and Swap）**

- **<span style="color:red">什么是CAS</span>**

  **使用锁时,线程获取锁是一种悲观锁(jdk1.6之前叫内建锁)策略,假设每一次访问共享资源都会产生冲突,所以当线程获取锁时也会阻塞其他未获取到锁的线程.**

  **而CAS操作(又称为无锁操作),它是一种乐观锁策略,它假设所有线程访问共享资源的时候不会出现冲突,由于不会出现冲突就不会阻塞其他线程,因此线程也就不会出现阻塞停顿状态, 当然这种情况在实际应用中并不合理,所以当出现冲突时CAS是如何做的呢？**

  **当冲突出现时,CAS（比较并替换）是这样操作的, 首先当执行CAS操作时，说明偏向锁已经被撤销,在多个线程竞争访问同一资源时,进行CAS操作，进行CAS操作时，锁状态是轻量级锁,假设线程A和线程B同时竞争访问资源X，假设A先抢到了这把锁,访问到了资源A,如何标注资源X正在被线程A访问呢？每个线程创建时都有自己的线程栈,栈帧中有一个Lock Record的,<span style="color:green">Lock Record是线程以轻量级锁的方式进入同步代码快的时候,先在当前的活动记录就是我们所说的栈帧中创建Lock Record,然后读取锁对象的markword替换为自己的Lock Record的指针(地址)，替换之前要比较当前锁对象的markword与第一次读取的markword是否相同,若相同则替换,若不相同,则重试(也就是自旋操作)</span>**
  
- **<span style="color:red">ABA问题</span>**

  **CAS操作会检查旧的值有没有发生变化，存在这个一样问题，旧值为A，要替换为C，但是在写入之前,A先变为B后又变为A，其实A已经发生了变化，**

  **解决方案:**

  1. **添加版本号**
  2. **或者在JDK1.5后使用atomic包的 `AtomicStampedReference类`解决问题。**

- **<span style="color:red">自旋会浪费大量的CPU资源问题</span>**

  **例如, 等待红灯时,假如只有5秒了,你可能就是踩下刹车停几秒钟,就又起步出发了, 要是还有50秒钟,你可能就会挂N或P档,拉起手刹.对于我们的线程来说,他不知道他还有多久可以获得这把锁.所以他就会持续自旋**
  **解决方案:**

  - **自适应自旋（重量级锁的优化）：根据以往自旋等待时能否获取锁，来动态调整自旋的时间（循环次数）。如果在自旋时获取到锁，则会稍微增加下一次自旋的时长；否则就稍微减少下一次自旋时长。**

- <span style="color:red">**公平性问题**</span>

  **自旋状态还带来另外一个副作用，不公平的锁机制。处于阻塞状态的线程，无法立刻竞争被释放的锁。然而，处于自旋状态的线程，则很有可能优先获得这把锁。**

  **内建锁无法实现公平机制，而`lock体系`可以实现公平锁**。

#### 4. Java 对象头和 Monitor

