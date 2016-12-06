##java多线程编程核心技术第二章总结
* 方法内变量线程安全
"非线程安全"问题存在于"实例变量"中，如果是方法内部的私有变量，则不会存在"非线程安全"的问题。
* 多个对象多个锁
        
        package com.msxf.psp.api.controller.threadtest;
        
        /**
         * 多个对象多个锁
         * Created by xi.chen on 2016/12/6 18:48.
         */
        public class MultiObjectAndLock {
        
            private int num =0;
            public static void main(String[] args) {
               // final MultiObjectAndLock multiObjectAndLock = new MultiObjectAndLock();
        
        
                Thread thread = new Thread(new Runnable() {
                    MultiObjectAndLock multiObjectAndLock1=new MultiObjectAndLock();
                    @Override
                    public void run() {
                        multiObjectAndLock1.mehtodA("a");
                    }
                });
        
                Thread thread2 = new Thread(new Runnable() {
                    MultiObjectAndLock multiObjectAndLock1 = new MultiObjectAndLock();
                    @Override
                    public void run() {
                        multiObjectAndLock1.mehtodA("b");
                    }
                });
                thread.start();
                thread2.start();
            }
        
            synchronized public void mehtodA(String username) {
                if(username.equals("a")){
                    num  = 100;
                    System.out.println("a is run");
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }else {
                    num = 200;
                    System.out.println("b is run ");
                }
        
                System.out.println("username num" + num);
            }
        }

运行结果：

        a is run
        b is run 
        username num200
        username num100
        
        Process finished with exit code 0
如果是同一个对象multiObjectAndLock1 =multiObjectAndLock，则运行结果

    a is run
    username num100
    b is run 
    username num200
    
    Process finished with exit code 0

* synchronized方法锁的级别是对象。
如果一个人类中有2个synchronized同步方法，只有当其中一个方法的锁释放掉另一个方法才能运行（获取锁）。

* synchronized锁重入
当一个线程获取一个对象的锁时，可以再次请求调用次对象的其它方法重新得到此对象的锁。这也证明在一个synchronized方法/块的内部调用此类其它的synchronized方法/块时，是可以永远得到锁的。
* 锁重入支持父子继承环境中

        package com.msxf.psp.api.controller.threadtest;
        
        /**
         * 锁重入支持父子继承环境中
         * Created by xi.chen on 2016/12/6 20:51.
         */
        public class LockReusingExtends {
        
        
            public static void main(String[] args) {
                MyThread1 myThread = new MyThread1();
                myThread.start();
        
            }
        
        }
        /******************************分割线***********************************/
        class FatherClass {
        
            public int i = 10;
        
            synchronized public void testMethod() {
                try {
                    i--;
                    System.out.println("father method" + i);
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        
        }
        
        class ChildClass extends FatherClass {
        
            synchronized public void testMethod() {
        
                try {
                    while (i>0){
                        i--;
                        System.out.println("child method" + i);
                        Thread.sleep(300);
                        super.testMethod();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        
        }
        
        class  MyThread1 extends  Thread{
            @Override
            public void run() {
                ChildClass child = new ChildClass();
                child.testMethod();
                super.run();
            }
        }

运行结果:

    child method9
    father method8
    child method7
    father method6
    child method5
    father method4
    child method3
    father method2
    child method1
    father method0
     
    Process finished with exit code 0

* 出现异常自动释放锁
很好理解...
* synchronized方法不可被继承
也就是说自己继承父类的synchronized方法是无效的，子类想要达到同步，必须加上synchronized，关键字。