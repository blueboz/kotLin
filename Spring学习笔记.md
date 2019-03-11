# Spring的自动配置

## Configuration注解

表示该类的@Bean注解开头的方法，其返回值都会注入Spring容器中。类似于将该java文件声明为一个Spring配置文件

```java
 @Configuration
 public class AppConfig {

     @Bean
     public MyBean myBean() {
         // instantiate, configure and return bean ...
     }
 }
```
### bootstrap 方式
1.
手工BootStrap@Configuration文件，可以通过AnnotationConfigApplicationContext 或者 AnnotationConfigWebApplicationContext. 
如下。

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.register(AppConfig.class);
ctx.refresh();
MyBean myBean = ctx.getBean(MyBean.class);
```
2.还可以通过如下的方式进行
```
<beans>
    <context:annotation-config/>
    <bean class="com.acme.AppConfig"/>
</beans>
 ```
 其中，上面的 <context:annotation-config/>是必要的，这个是为了启用ConfigurationClassPostProcessor
 
 ### 原理
 其实都是基于BeanFactoryPostProcessor 来进行的，我们知道，BeanFactory 在初始完毕之后，会调用这个类的实现类。并且送入BeanFactory供对需要的功能进行
 改造的途径。ConfigurationClassPostProcessor也是一样，主要处理类方法上的@Bean注解
 
 Spring Enviroment的应用
 https://blog.csdn.net/tales522/article/details/82319001
 
 
 Spring自动配置原理
 https://www.cnblogs.com/lfjjava/p/6096884.html
