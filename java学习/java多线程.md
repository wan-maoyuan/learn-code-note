## java多线程
### 核心
> ==高内聚低耦合的线程操纵资源类==

### 串行
> 每个任务依次执行

### 并发
> 多个任务同时执行

### 线程的生命周期
> 从对象创建到销毁，这个过程我们称为线程的生命周期。

#### 线程有哪些状态
1. 新生态(new)：一个线程被实例化完成，还没有任何操作。
2. 就绪态(ready)：开始争抢CPU时间片。
3. 运行态(run)：抢到了CPU时间片，开始执行。
4. 阻塞态(interrupt)：运行过程中，收到某些操作的影响，放弃CPU的时间片，且不再参与。
5. 死亡态(dead)：一个线程对象需要被销毁。

### 线程的创建方法
1. 继承Thread类

   ```java
   //创建线程方法一：继承Thread类，重写run方法，调用start开启线程
   public class Test01 extends Thread{
       @Override
       public void run() {
   //        run方法体
           for (int i = 0; i < 1000; i++) {
               System.out.println(i);
           }
       }
   
       public static void main(String[] args) {
           Test01 t1 = new Test01();
           t1.start();
       }
   }
   ```

2. 继承Runnable接口

   ```java
   //创建线程方法二：继承Runnable接口，实现run方法，创建一个线程对象，调用start开启线程。
   public class Test01 implements Runnable {
   
       public void run() {
           for (int i = 0; i < 10; i++) {
               System.out.println(i);
           }
       }
   
       public static void main(String[] args) {
           Test01 t1 = new Test01();
           Thread thread = new Thread(t1);
           thread.start();
       }
   }
   ```

   

3. 实现Callable接口

### 线程的命名
```java
Thread t = new Thread();
t.setName("custom");
```

```java
Thread t = new Thread(()->{},"custom");
```

```java
class MyThread extends Thread {
    public MyThread() {}
    public MyThread(String name) {
        super(name);
    }
}
```

### 线程的休眠
```java
class MyThread extends Thread {
    public void run(){
        for (int i=0; i<10; i++){
            System.out.println(i);
            try{
                Thread.sleep(1000);
            }catch(InterruptedException e){
                <!---->
            }
        }
    }
}
```

### 线程的优先级
1. 设置优先级只是修改这个线程抢到CPU时间片的概率
2. 并不是优先级高，一定可以抢到。
3. 优先级的设置是一个0到10的整数，默认是5
4. 设置线程的优先级一定要在线程开启之前
```java
Thread t = new Thread(()->{},"custom");
t.setPriority(10);
t.start();
```

### 线程的礼让
1. 线程礼让，让当前线程释放CPU资源，由运行状态回到就绪状态。
```java
Runnable r = new Runnable(){
  public void run(){
    for (int i=0; i<10; i++){
        if (i == 3){
            Thread.yield();
        }
    }
  }
};
Thread t1 = new Thread(r, "thread-1");
Thread t2 = new Thread(r, "thread-2");
t1.start();
t2.start();
```

### 临界资源问题
1. 一个对象里面加入有多个synchronized方法，某一时刻内，只要有一个线程去调用其中的某个synchronized方法，其他的线程就只能等待。某一时间内，只能有唯一一个线程去调用synchronized方法。

```java
class Phone{
    public synchronized void sendEmail() throws Exception {
        TimeUnit.SECONDS.sleep(3);
        System.out.println("send Email");
    }
    public synchronized void sendDuanXin(){
        System.out.println("send duanxin");
    }
}

public class Demo03 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "A").start();
        new Thread(() -> phone.sendDuanXin(), "A").start();
    }
}
```

2. 当在类的方法中加上static synchronized时，这就形成了一个类所，他会将这个类的所有对象全部锁住，同一时间内只有一个实现对象能够调用static synchronized方法。
> static synchronized是全局锁
```java
class Phone{
    public static synchronized void sendEmail() throws Exception {
        TimeUnit.SECONDS.sleep(3);
        System.out.println("send Email");
    }
    public static synchronized void sendDuanXin(){
        System.out.println("send duanxin");
    }
}

public class Demo03 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "A").start();
        new Thread(() -> phone.sendDuanXin(), "A").start();
    }
}
```

- 同步方法块
```java
public class Main {
    public static void main(String[] args) {
	    Runnable r = new Runnable() {
            @Override
            public void run() {
                while (TicktetCenter.restCount > 0){
                    synchronized (""){
                        if (TicktetCenter.restCount < 1){
                            break;
                        }
                        System.out.println(Thread.currentThread() + "      " + --TicktetCenter.restCount);
                    }

                }
            }
        };
	    Thread t1 = new Thread(r, "thread-1");
	    Thread t2 = new Thread(r, "thread-2");
	    Thread t3 = new Thread(r, "thread-3");
	    Thread t4 = new Thread(r, "thread-4");
	    t1.start();
	    t2.start();
	    t3.start();
	    t4.start();
    }
    static class TicktetCenter{
        public static int restCount = 100;
    }
}
```

- 同步方法
```java
public class Main {

    public static void main(String[] args) {
	    Runnable r = new Runnable() {
            @Override
            public void run() {
                while (TicktetCenter.restCount > 0){
                    synchronized (""){
                        soldTicket();
                    }

                }
            }
        };
	    Thread t1 = new Thread(r, "thread-1");
	    Thread t2 = new Thread(r, "thread-2");
	    Thread t3 = new Thread(r, "thread-3");
	    Thread t4 = new Thread(r, "thread-4");
	    t1.start();
	    t2.start();
	    t3.start();
	    t4.start();
    }
    public synchronized static void soldTicket(){
        if (TicktetCenter.restCount < 1){
            return;
        }
        System.out.println(Thread.currentThread() + "      " + --TicktetCenter.restCount);
    }
    static class TicktetCenter{
        public static int restCount = 100;
    }
}
```

> 静态同步方式的锁:就是当前类。

> 非静态方法的锁:是this。

### 显示锁ReenTrantLock
```java
public class Main {
    public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock();
	    Runnable r = new Runnable() {
            @Override
            public void run() {
                while (TicktetCenter.restCount > 0){
                    lock.lock();
                    if (TicktetCenter.restCount <= 0){
                        return;
                    }
                    System.out.println(Thread.currentThread() + "      " + --TicktetCenter.restCount);
                    lock.unlock();
                }
            }
        };
	    Thread t1 = new Thread(r, "thread-1");
	    Thread t2 = new Thread(r, "thread-2");
	    Thread t3 = new Thread(r, "thread-3");
	    Thread t4 = new Thread(r, "thread-4");
	    t1.start();
	    t2.start();
	    t3.start();
	    t4.start();
    }
    static class TicktetCenter{
        public static int restCount = 100;
    }
}
```

### 死锁
> 多个线程彼此持有对方所需要的锁对象，而不是放自己的锁。
```java
public static void main(String[] args) {
    ReentrantLock lock = new ReentrantLock();
    Runnable r1 = new Runnable() {
        @Override
        public void run() {
            synchronized ("A"){
                System.out.println(Thread.currentThread() + "get A");
                synchronized ("B"){
                    System.out.println(Thread.currentThread() + "get B");
                }
            }
        }
    };
    Runnable r2 = new Runnable() {
        @Override
        public void run() {
            synchronized ("B"){
                System.out.println(Thread.currentThread() + "get B");
                synchronized ("A"){
                    System.out.println(Thread.currentThread() + "get A");
                }
            }
        }
    };
    Thread t1 = new Thread(r1, "thread-1");
    Thread t2 = new Thread(r2, "thread-2");
    t1.start();
    t2.start();
}
```

### wait/notify/notifyAll
- wait
> 等待，object类中的方法，释放当前线程的锁，让出CPU资源，当前线程进入等待队列。

- notify
> 通知，object类种方法，唤醒等待队列中的一个线程，进入锁池。

- notifyAll
> 通知，object类种方法，唤醒等待队列中的所有的线程，并是这些线程进入锁池。


## 两个线程操作一个初始值为0的变量，一个线程对变量加1，一个线程对变量减1.交替10轮，变量值为0.
```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Number{
    int num = 0;

    public synchronized void add() throws Exception{
        if (num != 0){
            this.wait();
        }
        num += 1;
        System.out.println("正在加1，值为" + num);
        this.notifyAll();
    }

    public synchronized void minus() throws Exception{
        if (num == 0){
            this.wait();
        }
        num -= 1;
        System.out.println("正在减1，值为" + num);
        this.notifyAll();
    }
}

public class Deme04 {
    public static void main(String[] args) throws Exception {
        Number number = new Number();
        Lock lock = new ReentrantLock();
        new Thread(() -> {
            for (int i=0; i<10; i++){
                try {
                    number.add();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i=0; i<10; i++){
                try {
                    number.minus();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

```