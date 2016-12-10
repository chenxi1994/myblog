##java多线程编程核心技术第一章总结
*  线程实现的2种方法。
1.exthends Thread

        package com.msxf.psp.api.controller.threadtest;
        
        /**
         * Created by xi.chen on 2016/12/6 10:23.
         */
        public class CountOperate extends Thread{
        
        
            public CountOperate(){
        
                System.out.println("CountOperate start");
                System.out.println("Thread.currentThread.getName="+Thread.currentThread().getName());
                System.out.println("CountOperate end");
            }
        
            @Override
            public void run() {
                System.out.println("run start");
                System.out.println("Thread.currentThread.getName="+Thread.currentThread().getName());
                System.out.println("run end");
            }
        
        
        
            public static void main(String[] args) {
        
                CountOperate countOperate = new CountOperate();
                countOperate.setName("A");
                countOperate.start();
            }
        }
        

    
2.implements Runnable

        package com.msxf.psp.api.controller.threadtest;
        
        /**
         * Created by xi.chen on 2016/12/6 10:23.
         */
        public class CountOperate implements Runnable{
        
        
            public CountOperate(){
        
                System.out.println("CountOperate start");
                System.out.println("Thread.currentThread.getName="+Thread.currentThread().getName());
                System.out.println("CountOperate end");
            }
        
            @Override
            public void run() {
                System.out.println("run start");
                System.out.println("Thread.currentThread.getName="+Thread.currentThread().getName());
                System.out.println("run end");
            }
        
        
        
            public static void main(String[] args) {
        
                CountOperate countOperate = new CountOperate();
                Thread thread = new Thread(countOperate);
                thread.start();
            }
        }
* isAlive()方法
判断当前线程是否存活
* sleep()方法
指定线程睡眠一段时间后自动运行
* getId()方法
getID方法的作用是获取线程的唯一id
* 停止线程
java有3种停止线程的方式，如下：
1.正常退出
2.使用stop()，强行终止线程，不推荐
3.interrupt()，中断线程(不会像for break那样立即停止,仅仅是打了一个停止的标记)
* 暂停线程
suspend(暂停),resume(运行)，过时的方法不推荐使用，它有以下几个缺陷。
1.在使用suspend与resume方法时，如果使用不恰当，极易造成公共同步对象独占，导致其它线程无法使用其资源。（示例参考1.8.2）
2.在使用suspend与resume方法时，容易出现线程暂停导致数据部同步的情况。（示例参考1.8.3）
* yield()方法
yield()表示放弃当前cpu的资源，将它让给其它任务线程去使用。注意：此方法放弃资源的时间不确定，有可能刚放弃，马上又获取资源。（示例参考1.9）
* 线程优先级,setPriority();
1.优先级的继承特性（子线程会继承主线程的优先级）
2.优先级具有规则性（高优先级总是先执行）
3.优先级具有随机性（不一定优先级高的线程就先执行）
* 用户线程，守护线程
setDeamon()在start()之前。
        package com.msxf.psp.batch.controller.batch;
        
        /**
         * 守护线程
         * Created by xi.chen on 2016/12/6 16:11.
         */
        public class DaemonThread {
        
            public static void main(String[] args) {
                Thread t = new Thread(new Runnable() {
                    private int i= 0;
                    @Override
                    public void run() {
                        while (true){
                            System.out.println(i++);
                            try {
                                Thread.sleep(1000);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                });
        
                t.setDaemon(true);  //守护线程，在主线程执行完之后自动死亡
                t.start();
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("主线程执行结束");
        
            }
        }

    
