1. Spring的事务传播机制

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

   