### 1. 线程安全性

对于多个线程访问同一个变量所出现的问题，可以通过如下三种思路解决：

- 不在线程之间共享该变量
- 将该变量设置为不可变的变量
- 在访问变量时进行同步操作

#### 内置锁

- 保证操作的原子性
- 保证可见性

#### volatile

- 保证变量的内存可见性
- 通常用于修饰标志变量（某个操作初始化成功、结束）

