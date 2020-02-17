1. Spring的事务机制

   >  https://blog.csdn.net/trigl/article/details/50968079 

   1. 事务的四大特性
      - 原子性：一个事务把多个细粒度操作在逻辑上包装为一个操作
      - 一致性：多个操作要么全部成功要么全部失败
      - 永久性：事务中的操作产生的结果是永久性的，不被意外情况所改变
      - 隔离性：每个独立的事务互不影响
   2. Spring中的事务
      1. 事务的实现方式
         1. 编程式
            - 通过在代码中进行事务的声明与控制
         2. 声明式
      2. 事务的传播
         - PROPAGATION_REQUIRED：支持当前事务，如果没有事务则新建一个事务，这是默认值。
         - PROPAGATION_REQUIRES_NEW：无论何种情况都会新建一个事务。
         - PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
         - PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
         - PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
         - PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
         - PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于PROPAGATION_REQUIRED。
      3. 事务的回滚
         - 当处于被事务控制的方法内抛出了RunTimeException及其子类异常，会在抛出异常后进行事务的回滚，如果在方法内部将异常进行了catch并且不抛出的话异常是不会回滚的，在方法上的@Transaction注解中可以指定异常种类进行针对性的回滚或者是不回滚。

2. Spring的代理，两种方式的区别

   1. jdk动态代理

      - 如果目标类是某个接口的实现，则通过jdk动态代理会动态生成该接口的实现，动态生成的代理类是Proxy的子类

        ```java
        核心方法：
        newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
        传入的参数：
            1.指定的类加载器
            2.指定的需要代理的接口
            3.invocationHandler：对该类进行实现，并将第二个参数中的接口的实现类作为成员变量，在invoke方法中编写自己的处理逻辑，最后可以通过反射进行目标对象方法的调用
        ```

      - 代理对象的生成过程

        1. `Class<?> cl = getProxyClass0(loader, intfs);`尝试从缓存中获取代理类class

           1. 如果缓存中无相关信息，则调用`ProxyClassFactory.apply()方法`生成class信息

              ```
              生成字节码方法：
              byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
              可以将字节码写入到磁盘中，反编译看生成的文件是什么
              反编译方法：使用jad.exe
              命令：jad -s java class文件名字
              ```


        2. 根据class获取构造器，生成实例

      - 代理对象的调用逻辑

        >  代理对象实现了被代理对象的接口，并继承了Proxy类，在调用代理对象的方法时，会直接调用父类Proxy的invoke方法，从而实现逻辑增强并进行实际方法的调用

   2. cglib代理

      - 如果目标类没有实现某个接口，则会使用cglib动态生成代理类(继承被代理类)

        ```java
        核心方法：
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(AAA.class);
        enhancer.setCallback(new MyMethodInterceptor());
        AAA aaa = (AAA)enhancer.create();
        aaa.hello();
        
        MyMethodInterceptor implements MethodInterceptor{
        ...
        }
        ```

      - 主要调用逻辑
        - 代理对象的方法后，会调用**MethodInterceptor**的**intercept()**方法，进而调用目标方法

3. Spring aop实现原理

   > 通过jdk动态代理或者cglib动态代理对切点进行逻辑增强，织入增加代码。

4. feign的启用，调用方式及底层实现原理

   > feign封装了http请求，在启动类中使用@EnableFeignClients注解开启fegin功能，一般使用interface定义feign client，在interface上使用@feignClient(name="xxx")，表明该interface为一个feign client。
   >
   > 系统在启动的时候会基于interface生成代理类，在代理类中使用http请求调用目标对象的接口

5. 单例模式

   ```java
   public class Single{
   	private Single(){}
   	private class Inner{
   		private static Single single = new Single();
   	}
   	public static Single getInstance(){
   		return Inner.single;
   	}
   
   }
   
   public class Single{
   	//为什么要使用volatile关键字
       //使得线程在读取变量时强制性从主存中读取，保证变量的线程可见性，防止以下这种情况：
       //single = new Single();这句代码其实分为三步：
       //1.开辟内存分配给这个对象
       //2.初始化对象
       //3.将内存地址赋给虚拟机栈内存中的doubleLock变量
       //注意上面这三步，第2步和第3步的顺序是随机的，这是计算机指令重排序的问题
       //假设有两个线程，其中一个线程执行下面这行代码，如果第三步先执行了，就会把没有初始化的内存赋值给doubleLock
       //然后恰好这时候有另一个线程执行了第一个判断if(single == null)，然后就会发现doubleLock指向了一个内存地址
       //这另一个线程就直接返回了这个没有初始化的内存，所以要防止第2步和第3步重排序
   	private volatile static Single single = null;
   	private Single(){}
   	public static Single getInstance(){
   		//为什么要判断两次，第一次判断是为了让大多数线程能直接获取已经创建好的对象；第二次判断是为了防止以下这种情况：同时两个线程进入第一个if，这时其中一个线程获取到类锁进入同步块，将线程创建好，另外一个线程也进入同步代码块，如果不进行第二次判断则会再次进行对象的创建。
   		if(single == null){
   			synchronized (Single.class){
   				if(single == null){
   					single = new Single();
   				}
   			}
   		}
   		return single;
   	}
   }
   ```

6. Mysql索引分类，unique索引和主键索引的区别

   ```
   索引分类：
   1. 一般索引
   2. 唯一索引
   3. 空间索引
   4. full text索引
   
   主键索引只能作用于主键，实际上是一个特殊的唯一索引，主键索引可以被其他表引用作为外键
   唯一索引可以有多个，而主键索引只能有一个
   ```
7. MySQL的默认事务隔离级别
> 可重复读，当开启了某个事务A对数据库进行查询时，另一个事务B对数据进行了修改并进行事务的提交，在A事务中无法读取到B事务对数据库的修改
> 而Oracle的事务隔离级别是读已提交
   
