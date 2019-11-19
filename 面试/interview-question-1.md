1. Spring的事务机制

   1. 事务的四大特性
      - 原子性：一个事务把多个细粒度操作在逻辑上包装为一个操作
      - 一致性：多个操作要么全部成功要么全部失败
      - 永久性：事务中的操作产生的结果是永久性的，不被意外情况所改变
      - 隔离性：每个独立的事务互不影响
   2. Spring中的事务
      1. 事务的实现方式
         1. 编程式
            - 通过在代码中进行事务的
         2. 声明式

2. Spring的代理机制，两种方式的区别

3. Spring aop实现原理

4. feign的启用，调用方式及底层实现原理

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
   	private volatile static Single single = null;
   	private Single(){}
   	public static Single getInstance(){
   		//为什么要判断两次
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