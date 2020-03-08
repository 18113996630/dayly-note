## IOC容器的初始化

![image-20200308123624264](.\pic\image-20200308123624265.png)



### 核心类

- BeanDefinition

  > 包含了对Bean实例的描述：属性值、构造参数等信息

- BeanDefinitionReader

  > 读取Bean的配置描述信息

- Resource

  > 封装了对资源文件、IO的操作

- BeanDefinitionRegistry

  > 将BeanDefinition向IOC容器进行注册

------

![image-20200308123556961](.\pic\image-20200308123556961.png)



### 初始化流程

1. 定位resource文件

   ![image-20200308123624264](.\pic\image-20200308123624264.png)

2. 通过不同的方式读取bean的配置信息，将信息封装为**BeanDefinition**

   - XmlBeanDefinitionReader
   - AnnotatedBeanDefinitionReader

3. 通过**BeanDefinitionRegistry**注册BeanDefinition信息（HashMap）

   > org.springframework.beans.factory.support.DefaultListableBeanFactory#registerBeanDefinition



## IOC容器的依赖注入

### 依赖注入的入口

```java
org.springframework.beans.factory.BeanFactory#getBean(java.lang.String, java.lang.Class<T>)
```

![image-20200308123624264](.\pic\image-20200308123624231.png)











