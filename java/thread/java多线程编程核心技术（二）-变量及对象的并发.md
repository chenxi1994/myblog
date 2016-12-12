##java多线程编程核心技术第二章总结
这篇主要写了关于synchronized的使用，以及对象锁的概念。
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
* synchronized方法的弊端
使用synchronized方法块会比synchronized（）更灵活，效率更高

synchronized（）:

           synchronized public void A() {
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("打印一句话");
            }
synchronized方法块

         public void A() {
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (this) {
                    System.out.println("打印一句话");
        
                }
            }
            
虽然是个简单的实例，但是很明显可以看出synchronized方法块的优势。

* 静态同步synchronized方法与synchronized(class)代码块
关键字synchronized可以加载static方法上，表示对当前class文件类持锁，synchronized（class）也是表示对class类持锁。
        
          synchronized public static void A() {
        
            }
            public  void B() {
        
                synchronized (SynchroizedTest.class){
        
                }
            }
如上面一段代码，方法A和B方法中的synchronized代码块持有的是一个锁。

* 引用数据类型String的常量池特性
以String类型来作为synchronized方法块的锁，需要注意String常量池的特性。

          public  void A() {
                synchronized ("C") { //同一个字符C作为锁
                    while (true){
                        System.out.println("A method is run");
        
                    }
                }
            }
            public void B() {
                synchronized ("C") { //同一个字符C作为锁
                    while (true){
                        System.out.println("B method is run");
        
                    }
                }
            }
输出结果:
        
          A method is run
          A method is run
          A method is run
          A method is run
          A method is run
          A method is run
          A method is run
          A method is run
          A method is run
          A method is run
只有线程A执行了，线程B没执行。

* 线程互相等待造成死锁
可以用JDK自带的工具来检测是否是死锁，进入CMD，进入JDK安装的bin目录，执行jps命令。得到一个run id，如3324，在执行jstack -1 3324就可观察是否死锁。
* 锁对象的改变

        package com.chenxi.api.thread;
        
        /**
         * Created by xi.chen on 2016/12/10 20:01.
         */
        public class SynchroizedTest {
        
        
            private String lock = "1";
        
            public void test() {
                synchronized (lock) {
                    System.out.println(Thread.currentThread().getName()+ " method is start");
                    lock = "2";
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+ " method is end");
                }
            }
        
            public static void main(String[] args) {
                final SynchroizedTest synchroizedTest = new SynchroizedTest();
                Thread thread = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        synchroizedTest.test();
                    }
                });
                thread.setName("A");
                thread.start();
        
                Thread thread2 = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        synchroizedTest.test();
                    }
                });
                thread2.setName("B");
                thread2.start();
            }
        
        }

运行结果：

        A method is start
        B method is start
        A method is end
        B method is end
        
很明显，在A线程执行的过程中，锁对象发生了改变，然后B获得了锁，所以结果如图。

* 关键字volatile与死循环
同步死循环问题，看西面一段代码。

        package com.chenxi.api.thread;
        
        /**
         * Created by xi.chen on 2016/12/10 20:51.
         */
        public class VolaticeTest {
        
        
            private boolean isRun = true;
        
            public void volatileTest() {
                    while (isRun) {
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("打印一句话");
                    }
            }
        
            public static void main(String[] args) {
                 VolaticeTest volaticeTest = new VolaticeTest();
        
                volaticeTest.volatileTest();
                volaticeTest.isRun = false;
                System.out.println("停止");
            }
        }

运行结果：

        "C:\Program Files\Java\jdk1.7.0_80\bin\java" -Didea.launcher.port=7532 "-Didea.launcher.bin.path=D:\Program Files\IntelliJ IDEA 14.1.5\bin" -Dfile.encoding=UTF-8 -classpath "C:\Program Files\Java\jdk1.7.0_80\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jce.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jfxrt.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\resources.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\rt.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\zipfs.jar;E:\github\test-project\spring-boot-test\api\build\classes\main;E:\github\test-project\spring-boot-test\api\build\resources\main;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-data-jpa\1.3.0.RELEASE\spring-boot-starter-data-jpa-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-web\1.3.0.RELEASE\spring-boot-starter-web-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\redis\clients\jedis\2.8.0\jedis-2.8.0.jar;C:\Users\xi.chen@msxf.com\.gradle\caches\modules-2\files-2.1\com.alibaba\druid\1.0.18\9e4da3cbebad34c9eb3900ef9dc30cb4a60cffe0\druid-1.0.18.jar;C:\Users\xi.chen@msxf.com\.m2\repository\mysql\mysql-connector-java\5.1.37\mysql-connector-java-5.1.37.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter\1.3.0.RELEASE\spring-boot-starter-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-aop\1.3.0.RELEASE\spring-boot-starter-aop-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-jdbc\1.3.0.RELEASE\spring-boot-starter-jdbc-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\hibernate-entitymanager\4.3.11.Final\hibernate-entitymanager-4.3.11.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\javax\transaction\javax.transaction-api\1.2\javax.transaction-api-1.2.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\data\spring-data-jpa\1.9.1.RELEASE\spring-data-jpa-1.9.1.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-aspects\4.2.3.RELEASE\spring-aspects-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-tomcat\1.3.0.RELEASE\spring-boot-starter-tomcat-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-validation\1.3.0.RELEASE\spring-boot-starter-validation-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.6.3\jackson-databind-2.6.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-web\4.2.3.RELEASE\spring-web-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-webmvc\4.2.3.RELEASE\spring-webmvc-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\commons\commons-pool2\2.4.2\commons-pool2-2.4.2.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot\1.3.0.RELEASE\spring-boot-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-autoconfigure\1.3.0.RELEASE\spring-boot-autoconfigure-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-logging\1.3.0.RELEASE\spring-boot-starter-logging-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-core\4.2.3.RELEASE\spring-core-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\yaml\snakeyaml\1.16\snakeyaml-1.16.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-aop\4.2.3.RELEASE\spring-aop-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\aspectj\aspectjweaver\1.8.7\aspectjweaver-1.8.7.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\tomcat-jdbc\8.0.28\tomcat-jdbc-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-jdbc\4.2.3.RELEASE\spring-jdbc-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\jboss\logging\jboss-logging\3.3.0.Final\jboss-logging-3.3.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\jboss\logging\jboss-logging-annotations\1.2.0.Beta1\jboss-logging-annotations-1.2.0.Beta1.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\hibernate-core\4.3.11.Final\hibernate-core-4.3.11.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\dom4j\dom4j\1.6.1\dom4j-1.6.1.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\common\hibernate-commons-annotations\4.0.5.Final\hibernate-commons-annotations-4.0.5.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\javax\persistence\hibernate-jpa-2.1-api\1.0.0.Final\hibernate-jpa-2.1-api-1.0.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\javassist\javassist\3.18.1-GA\javassist-3.18.1-GA.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\data\spring-data-commons\1.11.1.RELEASE\spring-data-commons-1.11.1.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-orm\4.2.3.RELEASE\spring-orm-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-context\4.2.3.RELEASE\spring-context-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-tx\4.2.3.RELEASE\spring-tx-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-beans\4.2.3.RELEASE\spring-beans-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\slf4j-api\1.7.13\slf4j-api-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\jcl-over-slf4j\1.7.13\jcl-over-slf4j-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-core\8.0.28\tomcat-embed-core-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-el\8.0.28\tomcat-embed-el-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-logging-juli\8.0.28\tomcat-embed-logging-juli-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-websocket\8.0.28\tomcat-embed-websocket-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\hibernate-validator\5.2.2.Final\hibernate-validator-5.2.2.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.6.3\jackson-annotations-2.6.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.6.3\jackson-core-2.6.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-expression\4.2.3.RELEASE\spring-expression-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\ch\qos\logback\logback-classic\1.1.3\logback-classic-1.1.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\jul-to-slf4j\1.7.13\jul-to-slf4j-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\log4j-over-slf4j\1.7.13\log4j-over-slf4j-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\aopalliance\aopalliance\1.0\aopalliance-1.0.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\tomcat-juli\8.0.28\tomcat-juli-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\antlr\antlr\2.7.7\antlr-2.7.7.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\jboss\jandex\1.1.0.Final\jandex-1.1.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\xml-apis\xml-apis\1.0.b2\xml-apis-1.0.b2.jar;C:\Users\xi.chen@msxf.com\.m2\repository\javax\validation\validation-api\1.1.0.Final\validation-api-1.1.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\classmate\1.1.0\classmate-1.1.0.jar;C:\Users\xi.chen@msxf.com\.m2\repository\ch\qos\logback\logback-core\1.1.3\logback-core-1.1.3.jar;E:\github\test-project\spring-boot-test\common\build\resources\main;D:\Program Files\IntelliJ IDEA 14.1.5\lib\idea_rt.jar" com.intellij.rt.execution.application.AppMain com.chenxi.api.thread.VolaticeTest
        打印一句话
        打印一句话
        打印一句话
        打印一句话
        打印一句话
        
        Process finished with exit code -1
程序运行不会停止下了，那我们有什么好的方法解决吗？
异步解决死循环。

        package com.chenxi.api.thread;
        
        /**
         * Created by xi.chen on 2016/12/10 20:51.
         */
        public class VolaticeTest {
        
        
            private boolean isRun = true;
        
            public void volatileTest() {
                    while (isRun) {
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("打印一句话");
                    }
            }
        
            public static void main(String[] args) {
                final VolaticeTest volaticeTest = new VolaticeTest();
        
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        volaticeTest.volatileTest();
                    }
                }).start();
        
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                volaticeTest.isRun = false;
                System.out.println("停止");
            }
        }

运行结果：

        "C:\Program Files\Java\jdk1.7.0_80\bin\java" -Didea.launcher.port=7533 "-Didea.launcher.bin.path=D:\Program Files\IntelliJ IDEA 14.1.5\bin" -Dfile.encoding=UTF-8 -classpath "C:\Program Files\Java\jdk1.7.0_80\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jce.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jfxrt.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\resources.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\rt.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.7.0_80\jre\lib\ext\zipfs.jar;E:\github\test-project\spring-boot-test\api\build\classes\main;E:\github\test-project\spring-boot-test\api\build\resources\main;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-data-jpa\1.3.0.RELEASE\spring-boot-starter-data-jpa-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-web\1.3.0.RELEASE\spring-boot-starter-web-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\redis\clients\jedis\2.8.0\jedis-2.8.0.jar;C:\Users\xi.chen@msxf.com\.gradle\caches\modules-2\files-2.1\com.alibaba\druid\1.0.18\9e4da3cbebad34c9eb3900ef9dc30cb4a60cffe0\druid-1.0.18.jar;C:\Users\xi.chen@msxf.com\.m2\repository\mysql\mysql-connector-java\5.1.37\mysql-connector-java-5.1.37.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter\1.3.0.RELEASE\spring-boot-starter-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-aop\1.3.0.RELEASE\spring-boot-starter-aop-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-jdbc\1.3.0.RELEASE\spring-boot-starter-jdbc-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\hibernate-entitymanager\4.3.11.Final\hibernate-entitymanager-4.3.11.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\javax\transaction\javax.transaction-api\1.2\javax.transaction-api-1.2.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\data\spring-data-jpa\1.9.1.RELEASE\spring-data-jpa-1.9.1.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-aspects\4.2.3.RELEASE\spring-aspects-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-tomcat\1.3.0.RELEASE\spring-boot-starter-tomcat-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-validation\1.3.0.RELEASE\spring-boot-starter-validation-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.6.3\jackson-databind-2.6.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-web\4.2.3.RELEASE\spring-web-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-webmvc\4.2.3.RELEASE\spring-webmvc-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\commons\commons-pool2\2.4.2\commons-pool2-2.4.2.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot\1.3.0.RELEASE\spring-boot-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-autoconfigure\1.3.0.RELEASE\spring-boot-autoconfigure-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\boot\spring-boot-starter-logging\1.3.0.RELEASE\spring-boot-starter-logging-1.3.0.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-core\4.2.3.RELEASE\spring-core-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\yaml\snakeyaml\1.16\snakeyaml-1.16.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-aop\4.2.3.RELEASE\spring-aop-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\aspectj\aspectjweaver\1.8.7\aspectjweaver-1.8.7.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\tomcat-jdbc\8.0.28\tomcat-jdbc-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-jdbc\4.2.3.RELEASE\spring-jdbc-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\jboss\logging\jboss-logging\3.3.0.Final\jboss-logging-3.3.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\jboss\logging\jboss-logging-annotations\1.2.0.Beta1\jboss-logging-annotations-1.2.0.Beta1.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\hibernate-core\4.3.11.Final\hibernate-core-4.3.11.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\dom4j\dom4j\1.6.1\dom4j-1.6.1.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\common\hibernate-commons-annotations\4.0.5.Final\hibernate-commons-annotations-4.0.5.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\javax\persistence\hibernate-jpa-2.1-api\1.0.0.Final\hibernate-jpa-2.1-api-1.0.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\javassist\javassist\3.18.1-GA\javassist-3.18.1-GA.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\data\spring-data-commons\1.11.1.RELEASE\spring-data-commons-1.11.1.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-orm\4.2.3.RELEASE\spring-orm-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-context\4.2.3.RELEASE\spring-context-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-tx\4.2.3.RELEASE\spring-tx-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-beans\4.2.3.RELEASE\spring-beans-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\slf4j-api\1.7.13\slf4j-api-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\jcl-over-slf4j\1.7.13\jcl-over-slf4j-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-core\8.0.28\tomcat-embed-core-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-el\8.0.28\tomcat-embed-el-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-logging-juli\8.0.28\tomcat-embed-logging-juli-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\embed\tomcat-embed-websocket\8.0.28\tomcat-embed-websocket-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\hibernate\hibernate-validator\5.2.2.Final\hibernate-validator-5.2.2.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.6.3\jackson-annotations-2.6.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.6.3\jackson-core-2.6.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\springframework\spring-expression\4.2.3.RELEASE\spring-expression-4.2.3.RELEASE.jar;C:\Users\xi.chen@msxf.com\.m2\repository\ch\qos\logback\logback-classic\1.1.3\logback-classic-1.1.3.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\jul-to-slf4j\1.7.13\jul-to-slf4j-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\slf4j\log4j-over-slf4j\1.7.13\log4j-over-slf4j-1.7.13.jar;C:\Users\xi.chen@msxf.com\.m2\repository\aopalliance\aopalliance\1.0\aopalliance-1.0.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\apache\tomcat\tomcat-juli\8.0.28\tomcat-juli-8.0.28.jar;C:\Users\xi.chen@msxf.com\.m2\repository\antlr\antlr\2.7.7\antlr-2.7.7.jar;C:\Users\xi.chen@msxf.com\.m2\repository\org\jboss\jandex\1.1.0.Final\jandex-1.1.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\xml-apis\xml-apis\1.0.b2\xml-apis-1.0.b2.jar;C:\Users\xi.chen@msxf.com\.m2\repository\javax\validation\validation-api\1.1.0.Final\validation-api-1.1.0.Final.jar;C:\Users\xi.chen@msxf.com\.m2\repository\com\fasterxml\classmate\1.1.0\classmate-1.1.0.jar;C:\Users\xi.chen@msxf.com\.m2\repository\ch\qos\logback\logback-core\1.1.3\logback-core-1.1.3.jar;E:\github\test-project\spring-boot-test\common\build\resources\main;D:\Program Files\IntelliJ IDEA 14.1.5\lib\idea_rt.jar" com.intellij.rt.execution.application.AppMain com.chenxi.api.thread.VolaticeTest
        打印一句话
        打印一句话
        停止
        
        Process finished with exit code 0

如果将JVM的运行参数设为-server，会是什么样呢？上面的程序运行结果一直是死循环，这是因为设置了-server时，JVM为了提高线程的运行效率，线程一直在私有
堆栈中取private boolean isRun = true的值，虽有代码后面volaticeTest.isRun = false，只是表示公共堆栈的值变为false，所以一直是死循环。那要
怎么解决这种异步死循环的问题呢？答案：用volatile关键字，强制线程每次取值从公共堆栈中取。

        volatile private boolean isRun = true;
所以将JVM运行参数设为-server一定要谨慎，虽然能提高线程运行效率，单也包含极大风险。
