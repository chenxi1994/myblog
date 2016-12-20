##java多线程编程核心技术第四章总结
* ReentantLock的初次使用
一个简单的service类

        package com.chenxi.api.thread.ReentantLock;
        
        import java.util.concurrent.locks.Lock;
        import java.util.concurrent.locks.ReentrantLock;
        
        /**
         * Created by xi.chen on 2016/12/18 17:10.
         */
        public class SeriveA {
            private Lock lock = new ReentrantLock();
        
            public void testMethod() {
                try {
                    lock.lock();
                    System.out.println(Thread.currentThread().getName()+"start");
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName()+"end");
        
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
        
                }
            }
        }

ThreadA

        package com.chenxi.api.thread.ReentantLock;
        
        /**
         * Created by xi.chen on 2016/12/18 17:10.
         */
        public class ThreadA extends Thread{
            private SeriveA seriveA ;
        
            public ThreadA(SeriveA seriveA) {
                this.seriveA = seriveA;
            }
        
            @Override
            public void run() {
                seriveA.testMethod();
            }
        }
测试类：

        package com.chenxi.api.thread.ReentantLock;
        
        /**
         * Created by xi.chen on 2016/12/18 17:09.
         */
        public class MainClass {
        
            public static void main(String[] args) {
                SeriveA seriveA =  new SeriveA();
                ThreadA threadA1 = new ThreadA(seriveA);
                ThreadA threadA2= new ThreadA(seriveA);
                ThreadA threadA3 = new ThreadA(seriveA);
                ThreadA threadA4 = new ThreadA(seriveA);
                threadA1.start();
                threadA2.start();
                threadA3.start();
                threadA4.start();
        
            }
        }
运行结果；

            Thread-0start
            Thread-0end
            Thread-1start
            Thread-1end
            Thread-2start
            Thread-2end
            Thread-3start
            Thread-3end
            
            Process finished with exit code 0
从运行结果看来，lock.lock(); lock.unlock();实现了同步的效果，

* 使用Condition实现等待/通知
关键字synchronized与wait/notify方法结合使用可以是等待/通知的模式，类ReentrantLock也可以实现等待/通知的模式，但需要借助Condition对象。在
使用wait/notify进行通知时，被通知的线程是jvm随机选择的。但使用ReentrantLock结合Condition可以实现选择性通知。而synchroized就相当于整个
Lock中只有一个单一的Condition对象。线程在notifyAll()时，需要唤醒所有waiting中的线程，没有选择权，这是极大的消耗cpu的资源。不管怎么说要实现
等待/通知就必须有一个共享的资源或在同步中。下面是使用Condition实现等待/通知的示例。
service类：

        package com.chenxi.api.thread.Condition;
        
        import java.util.concurrent.locks.Condition;
        import java.util.concurrent.locks.Lock;
        import java.util.concurrent.locks.ReentrantLock;
        
        /**
         * Created by xi.chen on 2016/12/18 17:40.
         */
        public class MyService {
        
            private Lock lock = new ReentrantLock();
            private Condition condition = lock.newCondition();
        
        
            public void await() {
        
                try {
                    lock.lock();
                    System.out.println("myService is start");
                    condition.await();
                    System.out.println("myService is end");
        
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        
            public void signal(){
                try{
                    lock.lock();
                    System.out.println("唤醒线程");
        
                    condition.signal();
                }finally {
                    lock.unlock();
                }
        
            }
        }

线程A类，用于线程等待：

        package com.chenxi.api.thread.Condition;
        
        /**
         * Created by xi.chen on 2016/12/18 17:40.
         */
        public class MyThreadA extends Thread {
        
            private  MyService myService;
        
            public MyThreadA(MyService myService) {
                this.myService = myService;
            }
        
            @Override
            public void run() {
                myService.await();
            }
        }
线程B类，用于唤醒线程:

        package com.chenxi.api.thread.Condition;
        
        /**
         * Created by xi.chen on 2016/12/18 17:40.
         */
        public class MyThreadB extends Thread {
        
            private  MyService myService;
        
            public MyThreadB(MyService myService) {
                this.myService = myService;
            }
        
            @Override
            public void run() {
                myService.signal();
            }
        }

测试类：

        package com.chenxi.api.thread.Condition;
        
        /**
         * Created by xi.chen on 2016/12/18 17:39.
         */
        public class MainClass {
            public static void main(String[] args) {
                MyService myService = new MyService();
                MyThreadA myThreadA = new MyThreadA(myService);
                MyThreadB myThreadB = new MyThreadB(myService);
                myThreadA.start();
        
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                myThreadB.start();
        
            }
        }
运行结果：

        myService is start
        唤醒线程
        myService is end
        
        Process finished with exit code 0
从运行结果很明显可以看出MyService中的await方法打印了myService is start，进入了等待中，然后线程B唤醒了线程A，执行了 myService is end。由此
实现了一对一的等待/通知模式。下面使用多个condition实现等待/通知模式。
service类：

        package com.chenxi.api.thread.Condition;
        
        import java.util.concurrent.locks.Condition;
        import java.util.concurrent.locks.Lock;
        import java.util.concurrent.locks.ReentrantLock;
        
        /**
         * Created by xi.chen on 2016/12/18 17:40.
         */
        public class MyService {
        
            private Lock lock = new ReentrantLock();
            private Condition conditionA = lock.newCondition();
            private Condition conditionB = lock.newCondition();
        
            public void awaitA() {
        
                try {
                    lock.lock();
                    System.out.println("A is start");
                    conditionA.await();
                    System.out.println("A is end");
        
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        
            public void awaitB() {
        
                try {
                    lock.lock();
                    System.out.println("B is start");
                    conditionB.await();
                    System.out.println("B is end");
        
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        
            public void signalA(){
                try{
                    lock.lock();
                    System.out.println("唤醒A");
                    conditionA.signal();
                }finally {
                    lock.unlock();
                }
        
            }
        
            public void signalB(){
                try{
                    lock.lock();
                    System.out.println("唤醒B");
                    conditionB.signal();
                }finally {
                    lock.unlock();
                }
        
            }
        }
A线程：

        package com.chenxi.api.thread.Condition;
        
        /**
         * Created by xi.chen on 2016/12/18 17:40.
         */
        public class MyThreadA extends Thread {
        
            private  MyService myService;
        
            public MyThreadA(MyService myService) {
                this.myService = myService;
            }
        
            @Override
            public void run() {
                myService.awaitA();
            }
        }
B线程：

        package com.chenxi.api.thread.Condition;
        
        /**
         * Created by xi.chen on 2016/12/18 17:40.
         */
        public class MyThreadB extends Thread {
        
            private  MyService myService;
        
            public MyThreadB(MyService myService) {
                this.myService = myService;
            }
        
            @Override
            public void run() {
                myService.awaitB();
            }
        }

测试类：

        package com.chenxi.api.thread.Condition;
        
        /**
         * Created by xi.chen on 2016/12/18 17:39.
         */
        public class MainClass {
            public static void main(String[] args) {
                MyService myService = new MyService();
                MyThreadA myThreadA = new MyThreadA(myService);
                MyThreadB myThreadB = new MyThreadB(myService);
                myThreadA.start();
                myThreadB.start();
        
        
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                myService.signalA();
            }
        }
运行结果：

        A is start
        B is start
        唤醒A
        A is end
在service类中，4个方法都是使用的同一个lock，也就是同一共享资源。但是通过使用不同的condition可以选着性的唤醒正在等待的线程。而synchroized只能
全部唤醒或者随机唤醒一个。
* 使用ReentrantLock和condition实现消费者-生产者模式。


        代码略，后面文章中会有。
* 公平锁与非公平锁
锁Lock分为公平锁与非公平锁，公平锁表示线程索取锁的顺序是按照线程加锁的顺序来获取的，即先来先得的FIFO先进先出的顺序。而非公平锁就是一种获取锁的抢
占机制，是随机得到的。

        private Lock lock = new ReentrantLock(true);  //表示公平锁
        private Lock lock = new ReentrantLock(false);  //表示非公平锁

* ReentrantLock的常见方法。

                /**
                 *表示condition这个对象监视器有多少个线程在等待condition唤醒。
                 * 比如：调用了condition.await()10次，就return 10;
                 */
                lock.getWaitQueueLength(condition);
                /**
                 *获取有多少个线程在等待此锁。也就是说相当于这个lock阻塞了多少个线程。比如5个线程去执行一个同步的方法，
                 * 加入这个方法执行时间会很长，那么就会有4个线程在等待此锁。return 4；
                 */
                lock.getQueueLength();
                /**
                 *查询当前线程保持此锁定的个数，也就是调用lock()方法的次数。
                 */
                lock.getHoldCount();
                 /**
                 * 查询指定线程是否正在等待此锁。也就是说线程线程是否阻塞
                 * @return boolean
                 */
                lock.hasQueuedThread(new Thread());
                /**
                 * 不执行某一个线程，表示有没有线程正在等待此锁。也就是说有没有线程被这个锁阻塞了。
                 */
                lock.hasQueuedThreads();
                /**
                 * 有没有线程正在等待当前监视器。
                 * @return boolean
                 */
                lock.hasWaiters(condition);
                 /**
                 * 判断是不是公平锁
                 */
                lock.isFair();
                /**
                 * 查询当前线程是否持有此锁
                 */
                lock.isHeldByCurrentThread();
                /**
                 *查询有没有现车持有此锁
                 */
                lock.isLocked();
                try {
                    /**
                     * 如果当前线程未中断，测获取此锁。如果已经中断则抛出异常。
                     */
                    lock.lockInterruptibly();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        
                /**
                 * 尝试着获取锁，也就是说仅在调用时锁未被另一个线程保持的时候，才能获取该锁。
                 */
                lock.tryLock();
* 类Condition的常见方法
           
                /**
                * 与condition.await()效果相似，但是awaitUninterruptibly()方法线程等地的时候中断不会
                * 抛异常，而await()在等待的时候线程中断了会抛出异常。
                */
               condition.awaitUninterruptibly();
               
               /**********************************分割线****************************************/
               try {
                   Calendar calendar = Calendar.getInstance();
                   calendar.add(Calendar.SECOND,10);//时间向后推10秒
                   /**
                    * 在指定的时间自动唤醒，比如calendar.add(Calendar.SECOND,10)， condition.awaitUntil(calendar.getTime());
                    * 表示所有持有次监视器condition的线程 10秒后自动唤醒。
                    */
                   condition.awaitUntil(calendar.getTime());
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
* 使用Condition实现顺序执行               
* 类ReentrantReadWriteLock的使用
类ReentrantLock具有完全排它的效果，即同一时间只有一个线程在执行ReentrantLock.lock()后面的任务。这样虽然保证了线程安全的效果，但效率非常的地下
。但JDK提供了类eentrantReadWriteLock类，使它可以加快运行效率。在没有不需要操作实例变量中，完全可以使用读写锁ReentrantReadWriteLock来提升
代码的运行效率。读写锁有2个锁，一个读锁，就就是共享锁；一个写锁，也就是排它锁。也就是多个读锁之间不互斥，读锁与写锁互斥，写锁与写锁互斥。
1.读读共享
2.写写互斥
3.读写互斥
4.写读互斥

* 读读共享例子：
service类
        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        import java.util.concurrent.locks.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:01.
         */
        public class MyService {
            /**
             * 读写锁
             */
            private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
        
            public void read() {
        
                try {
                    lock.readLock().lock();
                    System.out.println(Thread.currentThread().getName()+"获得锁"+System.currentTimeMillis());
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.readLock().unlock();
                }
            }
        }
线程A

        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:05.
         */
        public class MyThreadA extends Thread {
        
            private MyService service;
        
            public MyThreadA(MyService service) {
                this.service = service;
            }
        
            @Override
            public void run() {
                service.read();
            }
        }
线程B

        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:05.
         */
        public class MyThreadB extends Thread {
            private MyService service;
        
            public MyThreadB(MyService service) {
                this.service = service;
            }
        
            @Override
            public void run() {
                service.read();
            }
        }
测试类：

        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:01.
         */
        public class MainClass {
        
            public static void main(String[] args) {
                MyService myService = new MyService();
                MyThreadA  myThreadA = new MyThreadA(myService);
                MyThreadB myThreadB =  new MyThreadB(myService);
                myThreadA.setName("A");
                myThreadB.setName("B");
                myThreadA.start();
                myThreadB.start();
            }
        }
运行结果：

        A获得锁1482248824786
        B获得锁1482248824786
        
        Process finished with exit code 0

由运行结果可以看出，A，B线程获取所得时间基本是一样，所以读锁是共享的，是并行的。

* 读写互斥的例子:
service类

        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        import java.util.concurrent.locks.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:01.
         */
        public class MyService {
            /**
             * 读写锁
             */
            private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
        
            public void read() {
        
                try {
                    lock.readLock().lock();
                    System.out.println(Thread.currentThread().getName()+"获得锁"+System.currentTimeMillis());
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.readLock().unlock();
                }
            }
            public void write() {
        
                try {
                    lock.writeLock().lock();
                    System.out.println(Thread.currentThread().getName()+"获得锁"+System.currentTimeMillis());
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.writeLock().unlock();
                }
            }
        
        }

A线程read

        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:05.
         */
        public class MyThreadA extends Thread {
        
            private MyService service;
        
            public MyThreadA(MyService service) {
                this.service = service;
            }
        
            @Override
            public void run() {
                service.read();
            }
        }
B线程write

        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:05.
         */
        public class MyThreadB extends Thread {
            private MyService service;
        
            public MyThreadB(MyService service) {
                this.service = service;
            }
        
            @Override
            public void run() {
                service.write();
            }
        }

测试类:

        package com.chenxi.api.thread.ReentrantReadWriteLock;
        
        /**
         * Created by xi.chen on 2016/12/20 23:01.
         */
        public class MainClass {
        
            public static void main(String[] args) {
                MyService myService = new MyService();
                MyThreadA  myThreadA = new MyThreadA(myService);
                MyThreadB myThreadB =  new MyThreadB(myService);
                myThreadA.setName("A");
                myThreadB.setName("B");
                myThreadA.start();
                myThreadB.start();
            }
        }
运行结果：

        A获得锁1482248971406
        B获得锁1482248972407
从运行结果上看，A,B获得锁的时间相差了1000毫秒，也就是说A,B是互斥的。线程安全的。