##java多线程编程核心技术第六章总结
* 懒汉模式的线程非完全问题
单例类

        package com.chenxi.api.thread.SignletonPattern;
        
        /**
         * Created by xi.chen on 2016/12/28 17:50.
         */
        public class SignleObject {
            private static  SignleObject signleObject;
        
        
            private SignleObject() {
            }
        
            public static SignleObject getInstance(){
                if(signleObject == null){
                    signleObject = new SignleObject();
                }
        
                return  signleObject;
            }
        }

测试类：

        package com.chenxi.api.thread.SignletonPattern;
        
        /**
         * Created by xi.chen on 2016/12/28 17:53.
         */
        public class MainClass {
        
            public static void main(String[] args) {
        
        
                for(int i=0;i<10;i++){
                    new Thread(new Runnable() {
                        @Override
                        public void run() {
        
                            System.out.println(SignleObject.getInstance().hashCode());
                        }
                    }).start();
        
                }
            }
        }

运行结果：

            1212229433
            762721342
            762721342
            762721342
            762721342
            762721342
            1212229433
            762721342
            762721342
            762721342
            
            Process finished with exit code 0

从结果可以看出，有些实例的hashcode是不一样的，由此可见，在多线程并发的情况下，懒汉模式是非线程安全的，并不能按照得到预期的结果。

* 懒汉模式非线程安全问题解决一

        package com.chenxi.api.thread.SignletonPattern;
        
        /**
         * Created by xi.chen on 2016/12/28 17:50.
         */
        public class SignleObject {
            private static  SignleObject signleObject;
        
        
            private SignleObject() {
            }
        
            public synchronized static SignleObject getInstance(){
                if(signleObject == null){
                    signleObject = new SignleObject();
                }
        
                return  signleObject;
            }
        }

在构建实例的方法上加个关键字就可以解决了，但是这种方案效率非常低下，如果一个线程在获取实例中，另外的线程就必须等待。

* 懒汉模式非线程安全问题解决二

        
           package com.chenxi.api.thread.SignletonPattern;
           /**
            * Created by xi.chen on 2016/12/28 17:50.
            */
           public class SignleObject {
               private  static SignleObject signleObject;
               private SignleObject() {
               }
               public static SignleObject getInstance() {
           
                   if (signleObject == null) {
                       synchronized (SignleObject.class) {
                           if(signleObject == null){
                               signleObject = new SignleObject();
                           }
           
                       }
           
                   }
                   return signleObject;
               }
           }

2次if的判断保证在下一次并发的时候可以不用重新获取锁了，而且synchronized方法块只对关键代码加锁，大大的提高的效率。网上说这样做有个缺点，大致
是指令重排序的问题。我们知道java早创建一个对象如，new SignleObject();分为3步。1.创建对象引用，2.在堆内存中创建对象实例，3.将对象实例赋给引
用。但是在多线程的环境了，可能先执行了第三步，这样对象还没创建，会报空指针异常。

* 懒汉模式非线程安全问题解决三

        package com.chenxi.api.thread.SignletonPattern;
        
        /**
         * Created by xi.chen on 2016/12/28 17:50.
         */
        public class SignleObject {
            private volatile static SignleObject signleObject;
        
        
            private SignleObject() {
            }
        
            public static SignleObject getInstance() {
        
        
                if (signleObject == null) {
                    synchronized (SignleObject.class) {
                        if(signleObject == null){
                            signleObject = new SignleObject();
                        }
        
                    }
        
                }
                return signleObject;
            }
        }
加上volatiel关键字是禁止指令重排序，对于这点我都是看别人博客得知，但是在深入理解jvm一书中对volatile的解释有另一种说法，具体是什么原因还有
待自己查阅质料核实。

* 静态内部类实现单利模式

        package com.chenxi.api.thread.SignletonPattern;
        
        /**
         * Created by xi.chen on 2016/12/28 20:58.
         */
        public class InnerSigntonObject {
        
            private static class InnerClass{
        
                private static InnerSigntonObject innerSigntonObject = new InnerSigntonObject();
            }
        
            private InnerSigntonObject(){
        
            }
            public static InnerSigntonObject getInstance(){
        
                return InnerClass.innerSigntonObject;
            }
        }

* 序列化与反序列化的单利模式
静态内部类可以实现线程安全的单例模式，但如果遇到序列化对象的时候还是非线程安全的。


            package com.chenxi.api.thread.SignletonPattern;
            
            import java.io.Serializable;
            
            /**
             * Created by xi.chen on 2016/12/28 21:02.
             */
            public class SerializableSigntonObject implements Serializable {
            
                private static final long  serialVersionId = 888L;
            
                private static class InnerClass{
            
                    private static SerializableSigntonObject serializableSigntonObject = new SerializableSigntonObject();
                }
            
                private SerializableSigntonObject(){
            
                }
                public static SerializableSigntonObject getInstance(){
            
                    return InnerClass.serializableSigntonObject;
                }
            }

测试类：

        package com.chenxi.api.thread.SignletonPattern;
        
        import java.io.*;
        
        /**
         * Created by xi.chen on 2016/12/28 17:53.
         */
        public class MainClass {
        
            public static void main(String[] args) {
        
                SerializableSigntonObject serializableSigntonObject = SerializableSigntonObject.getInstance();
                try {
                    FileOutputStream fileOutputStream = new FileOutputStream(new File("E:/a.txt"));
        
                    ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
                    objectOutputStream.writeObject(serializableSigntonObject);
                    objectOutputStream.close();
                    fileOutputStream.close();
                    System.out.println(serializableSigntonObject.hashCode());
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
        
        
                try {
                    FileInputStream fileInputStream = new FileInputStream(new File("E:/a.txt"));
                    ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
                    SerializableSigntonObject serializableSigntonObject1= (SerializableSigntonObject) objectInputStream.readObject();
                    objectInputStream.close();
                    fileInputStream.close();
                    System.out.println(serializableSigntonObject1.hashCode());
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (ClassNotFoundException e) {
                    e.printStackTrace();
                }
        
            }
        }
运行结果：

        58716751
        2098409254
        
        Process finished with exit code 0
从运行结果可以看出，对于序列化的对象，2次的对象的hashcode不同。
解决办法如下：

        package com.chenxi.api.thread.SignletonPattern;
        
        import java.io.ObjectStreamException;
        import java.io.Serializable;
        
        /**
         * Created by xi.chen on 2016/12/28 21:02.
         */
        public class SerializableSigntonObject implements Serializable {
        
            private static final long  serialVersionId = 888L;
        
            private static class InnerClass{
        
                private static SerializableSigntonObject serializableSigntonObject = new SerializableSigntonObject();
            }
        
            private SerializableSigntonObject(){
        
            }
            public static SerializableSigntonObject getInstance(){
        
                return InnerClass.serializableSigntonObject;
            }
        
        
            protected  Object readResolve() throws ObjectStreamException {
                return  InnerClass.serializableSigntonObject;
            }
        }

增加一个 protected  Object readResolve() throws ObjectStreamException()方法，jvm会自己调用这个方法？

* 枚举实现单利模式


* 对枚举单例模式的改造