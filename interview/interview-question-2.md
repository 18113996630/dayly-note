1. ##### 一个是线程同步的，基本就是如何确保临界区，最多只有三个线程进入
	

通过**semaphore**可以控制并发的数量
	Semaphore semaphore = new semaphore(int permits);
	semaphore.acquire();
	deal();
	semaphore.release();


​	**permits**代表同时允许进入同步区的线程数量，通过**accquire()**方法获取信号量，**release()**方法释放信号量
​	

2. ##### 四五个类的继承链问重写方法的调用


如果子类重写了父类中的方法，则优先调用自身的方法，否则调用父类的方法
   	
当某个类A同时继承了父类、实现了接口，父类与接口中存在相同的方法定义，创建A对象时并向上转型后调用该方法时，优先调用A中的方法逻辑，其次是父类中的方法逻辑
   	
3. ##### 对静态内部类.class调用toString方法
   

外部类：**Entry**
内部类：**Inner**

在外部类中调用println方法进行输出：
`System.out.println(Inner.class.toString())`
class org.hr.testDemo.extend.multi.Entry$Inner
   	

4. ##### Top K问题

	1. 如果是KV类型的数据，先根据K进行分组
	2. 在分组后的数据中将结果进行reduce
	3. 根据K进行降序排序，并取前K的数据

	------
	
	1. 如果是K类型的数据，则先调用map方法将数据转化为KV类型的数据
	2. 根据K进行分组
   3. 在分组后的数据中将结果进行reduce
   4. 根据K进行降序排序，并取前K的数据
   
5. ##### 把数组中的重复元素去掉然后按升序排序

   1. 通过List进行去重，再排序
   2. 创建目标数组，对原数组进行双重循环判断重复元素，再排序

6. ##### 一致性哈希算法

  - 目的：将数据进行更加均匀的映射，主要用于缓存
  - 实现步骤
    - 取出数据的hashCode，并将其对2^32取余获得值：A
    - 将0到2^32-1区间想象为一个闭合的环，将缓存节点均匀分布在这个环上
    - 将A值放在该环上，顺时针遇到的第一个节点即为数据缓存的目标节点
    - 当缓存节点比较少的时候，可以虚拟出多个虚拟节点，提高数据的均匀分布性

7. ##### CountDownLatch

   > CountDownLatch为一个线程等待工具，主要的目的是控制线程执行的次数，并让主线程等待其他线程结束后再继续
   >
   > 使用创建CountDownLatch对象时可以传入一个int值，在线程进行具体的业务操作时，可以调用CountDownLatch.count()方法降低值，在线程外部调用await()方法阻断主线程的继续，

8. ##### 阻塞队列

9. ##### 线程池 

   > 线程池是一个池化技术，线程是一个创建代价昂贵的资源，所以需要将线程进行统一管理，提高线程资源的利用效率、系统的反应速度(获取线程的速度)
   >
   > 核心参数：
   >
   > 1. 核心线程数量
   > 2. 最大线程数量
   > 3. 线程活跃时间
   > 4. 等待队列
   > 5. 等待队列满了并且线程数量达到最大线程数量时对于任务的拒绝策略

10. ##### sychronized和lock区别

    | 类别     |                         synchronized                         |                             Lock                             |
    | -------- | :----------------------------------------------------------: | :----------------------------------------------------------: |
    | 存在层次 |                  Java的关键字，在jvm层面上                   |                           是一个类                           |
    | 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁 |         在finally中必须释放锁，不然容易造成线程死锁          |
    | 锁的获取 |  假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待  | 分情况而定，Lock有多个锁获取的方式，具体下面会说道，大致就是可以尝试获得锁，线程可以不用一直等待 |
    | 锁状态   |                           无法判断                           |                           可以判断                           |
    | 锁类型   |                    可重入 不可中断 非公平                    |               可重入 可判断 可公平（两者皆可）               |
    | 性能     |                           少量同步                           |                           大量同步                           |

11. ##### 分布式锁

    > 原理：使用redis的setnx命令进行实现

    ```java
    package com.hrong.transaction.util;
    
    import lombok.extern.slf4j.Slf4j;
    import org.apache.commons.lang3.StringUtils;
    import org.springframework.data.redis.core.StringRedisTemplate;
    import org.springframework.stereotype.Component;
    
    import javax.annotation.Resource;
    
    /**
        * @author huangrong
        */
    @Slf4j
    @Component
    public class DistributedLockHandler {
    
        @Resource
        private StringRedisTemplate stringRedisTemplate;
    
        /**
         * 加锁
         * @param key productId - 商品的唯一标志
         * @param value  当前时间+超时时间 也就是时间戳
         * @return
         */
        public boolean lock(String key,String value){
            //对应setnx命令
            if(stringRedisTemplate.opsForValue().setIfAbsent(key,value)){
                //可以成功设置,也就是key不存在
                return true;
            }
    
            //判断锁超时 - 防止原来的操作异常，没有运行解锁操作  防止死锁
            String currentValue = stringRedisTemplate.opsForValue().get(key);
            //如果锁过期
            //currentValue不为空且小于当前时间
            if(!StringUtils.isEmpty(currentValue) && Long.parseLong(currentValue) < System.currentTimeMillis()){
                //获取上一个锁的时间value
                //对应getset，如果key存在
                String oldValue =stringRedisTemplate.opsForValue().getAndSet(key,value);
    
                //假设两个线程同时进来这里，因为key被占用了，而且锁过期了。获取的值currentValue=A(get取的旧的值肯定是一样的),两个线程的value都是B,key都是K.锁时间已经过期了。
                //而这里面的getAndSet一次只会一个执行，也就是一个执行之后，上一个的value已经变成了B。只有一个线程获取的上一个值会是A，另一个线程拿到的值是B。
                if(!StringUtils.isEmpty(oldValue) && oldValue.equals(currentValue) ){
                    //oldValue不为空且oldValue等于currentValue，也就是校验是不是上个对应的商品时间戳，也是防止并发
                    return true;
                }
            }
            return false;
        }
        /**
         * 解锁
         */
        public void unlock(String key,String value){
            try {
                String currentValue = stringRedisTemplate.opsForValue().get(key);
                if(!StringUtils.isEmpty(currentValue) && currentValue.equals(value) ){
                    //删除key
                    stringRedisTemplate.opsForValue().getOperations().delete(key);
                }
            } catch (Exception e) {
                log.error("[Redis分布式锁] 解锁出现异常了",e);
            }
        }
    }
    ```

12. ##### 手写算法题统计连续子数组和等于指定值的次数

13. ##### java List，HashMap的操作

    1. List

       1. ArrayList

          > ArrayList善于查询，因为数组在堆中是一块连续的内存空间，可以直接通过下标获取到值 。创建的时候如果清楚元素数量最好初始化ArrayList的时候就指定容量。
          >
          > 
          >
          > 使用object[]数组来存储数据，创建的时候如果没有指定容量，则默认为10（懒加载，并不会直接创建对象）
          >
          > 新增元素的时候，会进行容量是否溢出的判断，如果需要扩容的话，扩容1.5倍，使用`elementData = Arrays.copyOf(elementData, newCapacity);`方法进行扩容

       2. LinkedList

          >LinkedList善于增删，因为内存不连续，list中的元素持有上个元素和下一个元素的引用，可以直接修改引用指向即可完成增删操作，不善于查询，只能从头遍历
          >
          >
          >
          >使用Node链表来保存数据，会持有first和last元素的引用，新增删除操作都是针对链表来进行操作

       

14. ##### B树遍历比二叉树快还是慢

15. ##### 为什么ORACLE默认使用BTREE的存储结构

   BTREE适用于早起计算机内存不大的时候，可以当做整个文件读进内存进行读写操作。

16. ##### 堆排序