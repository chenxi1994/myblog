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



