##java多线程编程核心技术第三章总结
* 等待/通知机制的实现
1.方法wait()是Object的方法，该方法将当前线程置入"预执行等待队列",并停程序的执行直到接到通知或中断。如果调用wait()时没有持适当
的锁，则抛出IllegalMonitorStateException的异常，它是RuntimeException的一个子类，不需要try-catch。
2.方法notify()或notifyAll()也要在同步或同步块中调用。 即在调用之前，线程也必须获得该对象的对象级的锁。
* notify之后不会立即释放锁
    
        package com.chenxi.api.thread;
        
        import java.util.ArrayList;
        import java.util.List;
        
        /**
         * Created by xi.chen on 2016/12/11 17:08.
         */
        public class WaitAndNotify {
            public static List<String> list = new ArrayList<>();
        
            public static void main(String[] args) {
        
                Object lock = new Object();
                Thread threadA = new Thread(new ThreadA(lock));
                Thread threadB = new Thread(new ThreadB(lock));
        
                threadB.start();
        
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                threadA.start();
        
            }
        
        
        }
        
        class ThreadA implements Runnable {
        
            private Object lock;
        
            public ThreadA(Object lock) {
                this.lock = lock;
            }
        
            @Override
            public void run() {
        
                synchronized (lock) {
        
                    for (int i = 0; i < 10; i++) {
                        WaitAndNotify.list.add("1");
                        if (WaitAndNotify.list.size() == 5) {
                            lock.notify();
                        }
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("list size is " + WaitAndNotify.list.size());
                    }
                }
            }
        }
        
        class ThreadB implements Runnable {
            private Object lock;
        
            public ThreadB(Object lock) {
                this.lock = lock;
            }
        
            @Override
            public void run() {
                synchronized (lock) {
        
                    // if(WaitAndNotify.list.size()==5){
                    System.out.println("ThreadB method is start");
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("ThreadB method is end");
                    //}
                }
            }
        }
        
运行结果：

          ThreadB method is start
          list size is 1
          list size is 2
          list size is 3
          list size is 4
          list size is 5
          list size is 6
          list size is 7
          list size is 8
          list size is 9
          list size is 10
          ThreadB method is end
结果如图所示，notify之后B没有立即执行，只是表示B线程被唤醒了，此时没有得到锁。当线程A运行结束后，才释放的锁，这时候B才继续运行。wait（）之后可以立即释放锁。
将main方法代码修改如下：
     
        public static void main(String[] args) {
            Object lock = new Object();
            Thread threadA = new Thread(new ThreadA(lock));
            Thread threadB = new Thread(new ThreadB(lock));
            threadA.start();
            threadB.start();
        }

运行结果：
       
        ThreadB method is start
        list size is 1
        list size is 2
        list size is 3
        list size is 4
        list size is 5
        list size is 6
        list size is 7
        list size is 8
        list size is 9
        list size is 10
        ThreadB method is end
结合2次运行的结果可以说明，wait和notify使用的是同一个锁，不能同时运行，即A线程和B线程不能同时运行，也可以看出wait之后锁是立即释放的。

* 线程的各个状态
新建
就绪(可运行):线程进入就绪状态的5种方式
运行
阻塞:线程出现阻塞的5中情况
死亡
* 当interrupt()方法遇到wait()
当线程呈wait()状态时,调用线程会出现InterruptedException异常。
* 方法wait(long)的使用
方法wait(long)的功能在等待一段时间后对线程唤醒。当然在指定时间之内也可以通过notify或notifyAll唤醒。wait(long)与sleep(long)的区别在于wait会释放锁，sleep不释放锁。
* 过早noyify或notifyAll会打乱程序的正常逻辑，如线程永远wait
* 等待wait()的条件发生变化
在使用wait/notify模式时，还需要注意另一种情况，wait条件发生变化时，容易造成程序逻辑混乱。


         private List<Object> list = new ArrayList<>();
        
        
            public void add(){
                synchronized (lock){
                    list.add("");
                    lock.notifyAll();
                }
            }
        
            public  void remove(){
                synchronized (lock){
                    if(list.size() == 0){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
                list.remove(0);
            }
如果开2个线程去调remove（）方法，此时这2个线程都会进入等待中；开一个线程去掉add（）方法，此时list加1了，并唤醒了remove中的2个线程，单由于只加了1一次
list的大小为1，所以会抛出IndexOutOfBoundsException异常。这显然不是我们期望的结果。

                    public  void remove(){
                        synchronized (lock){
                            while(list.size() == 0){
                                try {
                                    lock.wait();
                                } catch (InterruptedException e) {
                                    e.printStackTrace();
                                }
                            }
                        }
                        list.remove(0);
                    }
将if判断语句改为while，这样就避免了上面的错误。
* 生产者-消费者模式

        由于在后面会有专项的练习，这里代码略。

* 通过管道进行线程间通信
 在JAVA语言中提供了各种各样的输入输出流，使我们能够很方便地对数据进行操作，其中管道流（pipeStream）是一种特殊的流，用于在不同线程间
 传送数据。一个线程发送数据到输出管道，另一个线程从输入管道读取数据。通过管道实现不同线程间的通信，无需借用临时文件之类的东西。主要借助
 PipedOutputStream ,PipedInputStream,PipedWriter,PipedReader;inputStream.connect(outputStream)看个例子。
 
        package com.chenxi.api.thread.pipestream;
        
        import java.io.IOException;
        import java.io.PipedInputStream;
        import java.io.PipedOutputStream;
        
        /**
         * Created by xi.chen on 2016/12/17 11:55.
         */
        public class PipeStreamManager {
        
            public void waiteData(PipedOutputStream outputStream) {
        
                String data = "fsfsdfdgfdggd gd gfd fh dgh ";
                byte[] dataByte = data.getBytes();
                try {
                    outputStream.write(dataByte, 0, data.length());
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        outputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
        
            }
        
            public void readData(PipedInputStream inputStream) {
        
                byte[] bytes = new byte[1024];
                try {
                    int length = 0;
                    while ( (length =  inputStream.read(bytes, 0, bytes.length)) != -1) {
                        //String dd = new String(,);
                        System.out.println(new String(bytes));
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }finally {
                    try {
                        inputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }

开2个线程

        package com.chenxi.api.thread.pipestream;
        
        import java.io.PipedInputStream;
        
        /**
         * Created by xi.chen on 2016/12/17 16:42.
         */
        public class ReadThread implements Runnable {
        
        
        
            private PipedInputStream pipedInputStream;
            private PipeStreamManager pipeStreamManager;
        
            public ReadThread(PipedInputStream pipedInputStream, PipeStreamManager pipeStreamManager) {
                this.pipedInputStream = pipedInputStream;
                this.pipeStreamManager = pipeStreamManager;
            }
        
            @Override
            public void run() {
        
                pipeStreamManager.readData(pipedInputStream);
            }
        }
        
        
        
        package com.chenxi.api.thread.pipestream;
        import java.io.PipedOutputStream;
        
        /**
         * Created by xi.chen on 2016/12/17 16:42.
         */
        public class WriteThread implements Runnable{
        
            private PipedOutputStream pipedOutputStream;
            private PipeStreamManager pipeStreamManager;
        
            public WriteThread(PipedOutputStream pipedOutputStream, PipeStreamManager pipeStreamManager) {
                this.pipedOutputStream = pipedOutputStream;
                this.pipeStreamManager = pipeStreamManager;
            }
        
            @Override
            public void run() {
                pipeStreamManager.waiteData(pipedOutputStream);
            }
        }

测试类：
        package com.chenxi.api.thread.pipestream;
        
        import java.io.*;
        
        /**
         * 线程间的通信
         * Created by xi.chen on 2016/12/17 11:48.
         */
        public class MainClass {
            public static void main(String[] args) {
                PipedOutputStream outputStream = new PipedOutputStream();
                PipedInputStream inputStream = new PipedInputStream();
                try {
                    inputStream.connect(outputStream); //这是线程间通信的关键代码
                } catch (IOException e) {
                    e.printStackTrace();
                }
        
                PipeStreamManager pipeStreamManager = new PipeStreamManager();
                WriteThread writeThread  = new WriteThread(outputStream,pipeStreamManager);
                ReadThread readThread = new ReadThread(inputStream,pipeStreamManager);
                Thread thread1 = new Thread(writeThread);
                Thread thread2 = new Thread(readThread);
        
                thread1.start();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                thread2.start();
        
            }
        }

运行结果:
        
        fsfsdfdgfdggd gd gfd fh dgh                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     
        
        Process finished with exit code 0

* 方法join的使用
很多情况下，主线程创建并启动子线程，如果子线程要进行大量的耗时运算，主线程往往将早于子线程结束。这时，如果主线程要用到子线程的
值，显然达不到我们预期的结果，就要用到join()方法了。

        package com.chenxi.api.thread.JoinThread;
        
        /**
         * Created by xi.chen on 2016/12/17 18:49.
         */
        public class MainClass {
        
            public static void main(String[] args) {
                final Thread thread = new Thread(new Runnable() {
                    @Override
                    public void run() {
        
                        try {
                            Thread.sleep(2000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("子线程。。。");
                    }
                });
                thread.start();
        
                try {
                    thread.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("主线程。。。");
            }
        }

运行结果：

        子线程。。。
        主线程。。。
        
        Process finished with exit code 0
如果将下面这段代码去掉：

         try {
                thread.join();
         } catch (InterruptedException e) {
                e.printStackTrace();
         }
        
运行结果为:

        主线程。。。
        子线程。。。
        
        Process finished with exit code 0

这例子中可以很明显看出join的作用，这里需要注意一点的是,join()方法一定要写在start()方法之后，不然不起作用。

* 方法join的异常
在join过程中，如果当前线程被中断，则当前线程发生异常。如现在有2线程A,B，B是A的子线程，如果A线程中断了，则会在join()的地方
抛出InterruptdException的异常。比如在另外的线程中调用interrupt(),按照正常情况下会在A线程执行完之后中断，但是有子线程之后
就会抛出异常。
* 方法join(long)
方法join(long)设置等待时间，指定的是主线程等待子线程的时间，如果在指定的时间类子线程还未执行完，则主线程在指定时间后自动运行。效果
与sleep(long)相似。
* 方法join(long)与sleep(long)的区别
方法join(long)的内部是用wait(long)来实现的，所以具备释放锁的功能。注意，join(long)释放的是子线程的锁。如a.join()释放的是a对象的锁。
下面是jdk join(long)的源码

               public final synchronized void join(long millis)
                    throws InterruptedException {
                        long base = System.currentTimeMillis();
                        long now = 0;
                
                        if (millis < 0) {
                            throw new IllegalArgumentException("timeout value is negative");
                        }
                
                        if (millis == 0) {
                            while (isAlive()) {
                                wait(0);
                            }
                        } else {
                            while (isAlive()) {
                                long delay = millis - now;
                                if (delay <= 0) {
                                    break;
                                }
                                wait(delay);
                                now = System.currentTimeMillis() - base;
                            }
                        }
                    }
* 方法join()的陷阱
A线程类：

        package com.chenxi.api.thread.JoinThread;
        
        /**
         * Created by xi.chen on 2016/12/17 20:45.
         */
        public class ThreadA extends Thread {
        
            private ThreadB threadB;
        
            public ThreadA(ThreadB threadB) {
                this.threadB = threadB;
            }
        
            @Override
            public void run() {
        
                synchronized (threadB){
                    System.out.println("ThreadA is begin");
                    try {
                        Thread.sleep(5000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("ThreadA is end");
                }
            }
        }

B线程类：

        package com.chenxi.api.thread.JoinThread;
        
        /**
         * Created by xi.chen on 2016/12/17 20:45.
         */
        public class ThreadB extends Thread {
        
        
            @Override
            public void run() {
                synchronized (this){
                    System.out.println("ThreadB is begin");
                    try {
                        Thread.sleep(5000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("ThreadB is end");
                }
            }
        }
测试类：

        package com.chenxi.api.thread.JoinThread;
        
        /**
         * Created by xi.chen on 2016/12/17 18:49.
         */
        public class MainClass {
        
            public static void main(String[] args) {
                ThreadB b = new ThreadB();
                ThreadA a = new ThreadA(b);
                a.start();
                b.start();
                try {
                    b.join(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("main is end");
            }
        }

运行结果1：

        ThreadA is begin
        ThreadA is end
        main is end
        ThreadB is begin
        ThreadB is end
        
        Process finished with exit code 0

运行结果2：  
    
        ThreadB is begin
        ThreadB is end
        main is end
        ThreadA is begin
        ThreadA is end
        
        Process finished with exit code 0
        
运行结果3：

        ThreadA is begin
        ThreadA is end
        ThreadB is begin
        ThreadB is end
        main is end
        
        Process finished with exit code 0
从结果来看具有不确定性，所以将join与同步想结合使用一定要谨慎。由于join内部是使用wait实现，在这里b.join()会竞争b对象的锁。由于线程一异步的，在
这里 b.start()，a.start()，a.start()谁获得会竞争b对象的锁对象的锁都不一定。所以，切记切记在join与同步相结合使用，一定要仔细的考虑逻辑。

* 类ThreadLocal的使用
变量值的共享可以使用public static变量的形式，所有的线程都使用同一个public static变量。如果想实现线程自己的一个共享变量该如何解决呢？JDK提
供ThreadLocal来解决这样的问题。
* 线程变量的隔离性
A线程：
       
        package com.chenxi.api.thread.ThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 21:35.
         */
        public class ThreadA extends Thread {
        
            @Override
            public void run() {
        
                MainClass.t1.set("ThreadA");
        
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("ThreadA value is:"+MainClass.t1.get());
            }
        }
        
B线程：
       
        package com.chenxi.api.thread.ThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 21:35.
         */
        public class ThreadB extends Thread {
        
            @Override
            public void run() {
                MainClass.t1.set("ThreadB");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("ThreadB value is:"+MainClass.t1.get());
            }
        }
        
测试类：

        package com.chenxi.api.thread.ThreadLocal;
        
        /**
         * 线程的变量共享问题
         * Created by xi.chen on 2016/12/17 21:32.
         */
        public class MainClass {
        
            public static MyThreadLoacl t1 = new MyThreadLoacl();
            public static void main(String[] args) {
                ThreadA threadA  = new ThreadA();
                ThreadB threadB  = new ThreadB();
                ThreadB threadB1  = new ThreadB();
                threadA.start();
                threadB.start();
                threadB1.start();
            }
        }

运行结果：

        ThreadB value is:ThreadB
        ThreadB value is:ThreadB
        ThreadA value is:我是默认值
        
        Process finished with exit code 0

从结果上看，各自的线程拥有自己的值，是互相隔离的，对于ThreadB，虽然创建了2个线程，但是这2个线程是一个类型，所以线程变量是一样的。但是如果是普通的
变量，B线程中打印的值肯定不同。

* 线程变量的初始值

        package com.chenxi.api.thread.ThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 21:45.
         */
        public class MyThreadLoacl extends ThreadLocal {
        
            @Override
            protected Object initialValue() {
                return "我是默认值";
            }
        }

将A线程代码修改如下：

        package com.chenxi.api.thread.ThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 21:35.
         */
        public class ThreadA extends Thread {
        
            @Override
            public void run() {
        
        
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("ThreadA value is:"+MainClass.t1.get());
            }
        }
        
测试类修改如下：

        package com.chenxi.api.thread.ThreadLocal;
        
        /**
         * 线程的变量共享问题
         * Created by xi.chen on 2016/12/17 21:32.
         */
        public class MainClass {
        
            public static MyThreadLoacl t1 = new MyThreadLoacl();
            public static void main(String[] args) {
                ThreadA threadA  = new ThreadA();
                ThreadB threadB  = new ThreadB();
                threadA.start();
                threadB.start();
            }
        }

运行结果:

        ThreadB value is:ThreadB
        ThreadA value is:我是默认值
        
        Process finished with exit code 0

从结果看来，A线程中并没有set值确认值，是因为initialValue()会初始化一个值，初始化的值是所有线程公用的。

* 类InheritableThreadLocal的使用
从类的名字就可以看出是线程变量值继承，是用类InheritableThreadLocal的使用可以让子线程从父线程中获取值。
看下面代码：

        package com.chenxi.api.thread.InheritableThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 22:00.
         */
        public class MyInheritableThreadLocal extends InheritableThreadLocal {
        
            @Override
            protected Object initialValue() {
                return "初始值";
            }
        }

A线程：

        package com.chenxi.api.thread.InheritableThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 22:00.
         */
        public class ThreadA extends Thread {
            @Override
            public void run() {
                System.out.println("子线程A的值：" +MainClass.threadLocal.get());
            }
        }

主线程main：

        package com.chenxi.api.thread.InheritableThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 22:00.
         */
        public class MainClass {
        
            public  static  InheritableThreadLocal threadLocal = new MyInheritableThreadLocal();
        
            public static void main(String[] args) {
                System.out.println("主线程main的值为：" + MainClass.threadLocal.get());
                ThreadA threadA  = new ThreadA();
                threadA.start();
            }
        }
运行结果：

        主线程main的值为：初始值
        子线程A的值：初始值
        
        Process finished with exit code 0

从运行结果可以看出子线程A的线程变量的值与主线程main的线程变量发的值是一样的。

值继承再修改；
将上面MyInheritableThreadLocal类的代码修改如下 ：

        package com.chenxi.api.thread.InheritableThreadLocal;
        
        /**
         * Created by xi.chen on 2016/12/17 22:00.
         */
        public class MyInheritableThreadLocal extends InheritableThreadLocal {
        
            @Override
            protected Object initialValue() {
                return "初始值";
            }
        
            @Override
            protected Object childValue(Object parentValue) {
                return  parentValue+"子线程的值";
            }
        }

运行结果为：

        主线程main的值为：初始值
        子线程A的值：初始值子线程的值
        
        Process finished with exit code 0
可以看出使用InheritableThreadLocal可以让线程变量的值继承和修改。