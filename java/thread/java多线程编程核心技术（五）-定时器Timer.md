##java多线程编程核心技术第五章总结
Timer类主要用来设置计划任务，但封装的任务的类却是TimeTask类。执行计划任务的时候要将代码逻辑放入TimeTask的子类，因为TimeTask是一个抽象类。
* Timer的简单示例
定义一个TimeTask类

        package com.chenxi.api.thread.timer;
        
        import java.util.TimerTask;
        
        /**
         * 任务
         * Created by xi.chen on 2016/12/27 17:10.
         */
        public class MyTask extends TimerTask {
            @Override
            public void run() {
        
                System.out.println("改任务执行了！,执行时间为："+System.currentTimeMillis());
            }
        }

测试类：

        package com.chenxi.api.thread.timer;
        
        import java.util.Timer;
        
        /**
         * Created by xi.chen on 2016/12/22 11:16.
         */
        public class MainClass {
            public static void main(String[] args) {
                System.out.println("main start time is "+System.currentTimeMillis());
                Timer timer = new Timer();
                timer.schedule(new MyTask(),2000); //异步
                System.out.println("main end time is "+System.currentTimeMillis());
            }
        }

运行结果：

        main start time is 1482843949004
        main end time is 1482843949005
        改任务执行了！,执行时间为：1482843951006

从结果可以看出，任务MyTask是在启动main函数之后2s执行的。而且此任务是异步的。看下timer的构造方法会发现，timer执行任务是新开了一个线程。并且这
个线程创建了之后可以复用，因为在task任务执行完之后，此线程还一直存在。如果我们将此线程设置为守护线程呢？

          public Timer(String name) {
                thread.setName(name);
                thread.start();
            }

将timer创建的线程设置为守护线程，只需timer timer = new Timer(true);就可以了。现将测试类的代码修改如下：

        package com.chenxi.api.thread.timer;
        
        import java.util.Timer;
        
        /**
         * Created by xi.chen on 2016/12/22 11:16.
         */
        public class MainClass {
            public static void main(String[] args) {
                System.out.println("main start time is "+System.currentTimeMillis());
                Timer timer = new Timer(true);
                timer.schedule(new MyTask(),2000); //异步
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("main end time is "+System.currentTimeMillis());
            }
        }

运行结果如下：

        main start time is 1482845590813
        改任务执行了！,执行时间为：1482845592814
        main end time is 1482845595814
        
        Process finished with exit code 0
可以看出此任务也是在2s后执行的，但是在主线程执行完之后，tiemr创建的线程会自动消失。所以将timer设置为true的时候，一定要注意diamante逻辑，如果
主线程的时间太短，导致task任务还没执行完就结束了，线程就消亡了。看下面一段代码；

        package com.chenxi.api.thread.timer;
        
                import java.util.Timer;
        
        /**
         * Created by xi.chen on 2016/12/22 11:16.
         */
        public class MainClass {
            public static void main(String[] args) {
                System.out.println("main start time is "+System.currentTimeMillis());
                Timer timer = new Timer(true);
                timer.schedule(new MyTask(),2000); //异步
              /*  try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }*/
                System.out.println("main end time is "+System.currentTimeMillis());
            }
        }

运行结果：

        main start time is 1482847255986
        main end time is 1482847255987
        
        Process finished with exit code 0
从结果可以看出，由于主线程执行时间太短，task任务的线程就没有执行。
* 方法 scheduleAtFixedRate(TimerTask task, Date firstTime,long period)的使用
scheduleAtFixedRate(TimerTask task, Date firstTime,long period) 与schedule（TimerTask task, Date firstTime,long period）
的区别在于如果任务执行没有延迟，scheduleAtFixedRate下次执行任务的时间是上次任务执行的结束时间起，schedule下次执行任务的时间是指定的定时时间
执行。如果任务有延迟，scheduleAtFixedRate和schedule没有区别，下次任务执行的时间都是上次任务结束的时间。