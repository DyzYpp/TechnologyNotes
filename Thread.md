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

#### 2. **CAS（Compare and Swap）**

- **<span style="color:red">什么是CAS</span>**

  **使用锁时,线程获取锁是一种悲观锁(jdk1.6之前叫内建锁)策略,假设每一次访问共享资源都会产生冲突,所以当线程获取锁时也会阻塞其他未获取到锁的线程.**

  而CAS操作(又称为无锁操作),它是一种乐观锁策略,它假设所有线程访问共享资源的时候不会出现冲突,由于不会出现冲突就不会阻塞其他线程,因此线程也就不会出现阻塞停顿状态, 当然这种情况在实际应用中并不合理,所以当出现冲突时CAS是如何做的呢？

  当冲突出现时,CAS（比较并替换）是这样操作的, 首先获取到访问资源的对应的锁,当没有竞争存在时,当前线程将它线程

