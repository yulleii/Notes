随着需求的变化，代码改动过大的原因是什么？

学习新的**设计模式**来重构我们的代码。前人根据自己的经验总结出来的，代表了代码设计的最佳实践。

从优秀的开源框架中，学习如何去使用。

![](/image/Snipaste_2019-07-23_21-30-45.png)

**观察者模式**

合理利用Spring事件机制来完成自己的需求。订单通知情景

**模板方法模式**

例如数据库的操作时，有很多重复的步骤，只修改特定的步骤来实现。

**策略模式**

电商"杀熟"怎么实现？对于不同买家实现不同的价格

总共有23种设计模式

# IOC

## 理解ApplicationContext

![](image/FileSystemXmlApplicationContext.png)

通过FileSystemXmlApplicationContext来理解ApplicationContext

### 如何给注册一个类？

xml配置文件

注解的方法

### 自动扫描组件和扫描规则

*ANNOTATION*

定制过滤规则

### 设置组件作用域

默认获取都是单实例

PROTOTYPE：多实例，IOC启动不会创建对象，只有在获取时才会创建对象到IOC容器中

SINGLETON：单实例（默认），IOC容器启用会调用方法创建对象到IOC容器中。

懒加载`@Lazy`：容器启动时，不会创建对象。只有在第一次使用获取，并初始化。

### 按照一定的条件给容器中注册Bean

### 导入组件到IoC容器

- import
  - @import(xxx.class,...)
  - **ImportSelector**
  - ImportBeanDefinitionRegistry

- 默认类：@Component @Controller @Service

- 自定义类：@Bean

- 使用Spring提供的FactoryBean（工厂Bean）

  默认获取getObject返回的对象。如果加上&将获得FactoryBean本身

### 生命周期

bean创建-创建-销毁的过程，使用容器来管理bean生命周期。可以自定义初始化和销毁方法

指定初始化和销毁方法

**1. @Bean**

使用`@Bean(initMethod="",destroyMethod="")`

初始化：对象创建完成，并赋值好。调用初始方法

销毁：单实例：容器关闭的时候销毁。而对于多实例，容器 销毁bean。

**2. 让Bean实现IntializingBean和DisposableBean**

重写对应的方法

**3. 使用@PostContruct和@PreDestroy**

JSR250，使用这两个注解来修饰对应的初始化方法和销毁方法

**4. BeanPostProcessor——bean的后置处理器**

初始化前后调用

### 赋值

使用@Value赋值

1. 基本数值
2. 可以写SpEL，#{}
3. ${}配置文件中的值（在运行环境变量里面的值）

### 自动装配

Spring利用依赖注入（DI），完成对IoC容器中各个组件依赖关系赋值。后置处理器的实际应用

**1. Autowired**

spring带有的Autowired。可以标注属性、方法、构造器、参数

- 默认优先按照类型去容器中找对应的组件：applicationContext.getBean(XXX.class)

- 如果容器中有多个相同类型的组件，再将字段名作为组件的id去容器中查找
- 使用@Qualifier("xxxx")，指定需要装配的组件的id

- 自动装配默认一定要将属性赋值好，没有就会报错。可以使用@Autowired(required=false)可以作为非必须
- @Primary让Spring自动装配默认使用首选的Bean。也可以继续使用@Qualifier。

当标注方法时，Spring容器会创建当前对象，就会调用方法，完成赋值。其中方法使用的参数，自定义类型的值从IoC容器中获取。@Bean标注的方法创建对象的时候，方法的参数的值从容器中获取

当标注构造器时，默认加在IoC容器中的组件，容器启动会调用无惨构造器创建对象，再进行初始化赋值等操作。如果组件只有一个有参构造器，@Autowired可以省略。

组件如果想要使用Spring容器底层的一些组件（ApplicationContext、BeanFactory等），自定义组件可以实现XXXAware：在创建对象的时候，会调用接口规定的方法注入到相关组件。原理也能使用XXXprocessor处理的

**2.Resource和Inject**

支持使用java规范的注解@Resource和@Inject。

**@Resource**：以和Autowired实现自动装配，但是这个注解默认是按照组件名称来装配。可以使用@Resource(name="xxxx")来指定，但不能支持@Primary和@Autowired(required=false)。

**@Inject**：需要导入javax.inject，和Autowired支持@Primary，但不支持required。

### Profile

可以根据当前环境动态的激活和切换一系列组件的功能。

例如开发环境、测试环境、生产环境

- 使用命令行参数：在虚拟机参数位置加载 -Dspring.profiles.active=test
- 使用代码

~~~java
1.创建一个applicationContext
AnnotationConfigApplicationContext applicationContext=new AnnotationConfigApplicationContext();
2.设置需要激活的环境
applicationContext.getEnvironment().setActiveProfiles("test","dev");
3.注册主配置类
applicationContext.register(MainConfigProfile.class);
4.启动刷新容器
applicationContext.refresh();
~~~

- 写在配置类上，没有指定环境表示的Bean都会被加载

# AOP

## 如何使用AOP

动态代理：指在程序运行期间将一段代码切入到指定方法指定位置运行的编程方式

**aop准备工作**

- 导入aop模块：Spring AOP（spring-aspects）

- 创建一个业务逻辑类（MathCalculator），在业务逻辑运行的时候将日志进行打印（方法之前、结束或者出异常等情况）

- 切面需要能感知到方法运行到哪里了。

> 通知方法有
>
> 前置通知(@Before)：目标方法运行之前运行
>
> 后置通知(@After)：方法运行之后通知
>
> 返回通知(@AfterReturning)：返回通知
>
> 异常通知(@AfterThrowing)：方法出现异常后运行
>
> 环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）

**创建和使用切面类：**

1. 给切面类的目标方法标注
2. 将切面类和业务逻辑类都加入到容器中
3. 告诉容器哪个类是切面类，也就是给切面类上加上@Aspect标注
4. 给配置类中加上@EnableAspectJAtutoProxy（开启基于注解的aop模式）

## AOP原理

看给容器中注册了什么组件,这个组件什么时候工作,以及组件的作用是什么?

### 核心注解@EnableAspectJAutoProxy

```
@Import({AspectJAutoProxyRegistrar.class})
```

给容器中导入AspectJAutoProxyRegistrar。利用AspectJAutoProxyRegistrar自定义容器中注册Bean

AnnotationAwareAspectJAutoProxyCreator=internalAutoProxyCreator

给容器中注册一个AnnotationAwareAspectJAutoProxyCreator

### AnnotationAwareAspectJAutoProxyCreator

![](/image/diagram.png)

注意其中**SmartInstantiationAwareBeanPostProcessor**后置处理器和**BeanFactorAware**

在和处理器以及BeanFactory相关的方法处打上断点，进行调试。

```java
AbstractAutoProxyCreator
public void setBeanFactory(BeanFactory beanFactory)
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName)
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName)

AbstractAdvisorAutoProxyCreator
public void setBeanFactory(BeanFactory beanFactory)->initBeanFactory()

AspectJAwareAdvisorAutoProxyCreator

AnnotationAwareAspectJAutoProxyCreator
protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory)
```

### 流程

创建IOC容器，传入配置类

注册配置类，调用refresh()刷新容器。

registerBeanPostProcessors(beanFactory);

1. 先获取IoC容器已经定义了的需要创建对象的所有BeanPostProcessor

2. 给容器中添加其他的BeanPostProcessor

3. 优先注册实现了PriorityOrdered接口的BeanPostProcessor

4. 再给容器中注册实现了Ordered接口的BeanPostProcessor

5. 注册没有实现优先级接口的BeanPostProcessor

6. 注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存到容器中。

   1. 创建Bean的实例
   2. populateBean：给Bean的各种属性赋值
   3. initializeBean：初始化bean
      1. invokeAwareMethods（），处理Aware接口的方法回调
      2. applyBeanPostProcessorsBeforeInitialization（）：执行后置处理器的postProcessorBeforeInitialization（）
      3. invokeInitMethods（）：执行自定义的初始化方法
      4. applyBeanPostProcessorsAfterInitialization（）：执行后置处理器的postProcessorAfterInitialization（）
   4. BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功

7. 把BeanPostProcessor注册到BeanFactory中

   beanFactory.addBeanPostProcessor(postProcessor)

以上是创建和注册的过程

AnnotationAwareAspectJAutoProxyCreator=>InstantiationAwareBeanPostProcessor（后置处理器不同）

完成beanFactory初始化工作：创建剩下的单实例Bean

1. 遍历容器并获取中所有的Bean，依次创建对象getBean（beanName）

   getBean（）-> doGetBean ->getSingleton()

2. 创建Bean，先从缓存获取当前bean，如果能获取到，说明bean已经被创建过；直接使用，否则再创建。只要创建好的bean都会被缓存起来。

   *AnnotationAwareAspectJAutoProxyCreator***在所有bean创建之前会有一个拦截**，*InstantiationAwareBeanPostProcessor*，会调用postProcessBeforeInstantiation()

   - createBean（）：创建bean，AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例

     **BeanPostProcessor**是在Bean对象创建完成初始化前后调用的

     **InstantiationAwareBeanPostProcessor**是在创建任何Bean实例之前先尝试用后置处理器返回对象的

     - resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation。希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续

       - 后置处理器尝试返回对象

         bean = applyBeanPostProcessorsBeforeInstantiation（）：

         拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;就执行postProcessBeforeInstantiation

         if (bean != null) {
         							bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
         						}

     - doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；



AnnotationAwareAspectJAutoProxyCreator【**InstantiationAwareBeanPostProcessor**】的作用：

1. 每一个bean创建之前，调用postProcessBeforeInstantiation()；

   关心MathCalculator和LogAspect的创建

   1. 判断当前bean是否在advisedBeans中（保存了所有增强bean）

   2. 判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，或者是否是切面（@Aspect）

   3. 判断是否需要跳过：获取候选的增强器（切面里面的通知方法）List<Advisor> candidateAdvisors每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor

      判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true

2. 创建对象

   postProcessAfterInitialization；

   ​	return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下

   1. 获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
      1. 找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
      2. 取到能在bean使用的增强器。
      3. 给增强器排序
      
   2. 保存当前bean在advisedBeans中；
   
   3. 如果当前bean需要增强，创建当前bean的代理对象；
      - 获取所有增强器（通知方法）
      - 保存到proxyFactory
      - 创建代理对象：Spring自动决定
        - JdkDynamicAopProxy(config);jdk动态代理；
        - ObjenesisCglibAopProxy(config);cglib的动态代理；
      
   4. 给容器中返回当前组件使用cglib增强了的代理对象；
   
   5. 以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；
   

目标方法执行

容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；

1. CglibAopProxy.intercept();拦截目标方法的执行

2. 根据ProxyFactory对象获取将要执行的目标方法拦截器链

   List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

   > 如何获取一个拦截器链呢？
   >
   > 拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
   >
   > - List<Object> interceptorList保存所有拦截器 ：一个默认的ExposeInvocationInterceptor 和 4个增强器；
   >
   > - 遍历所有的增强器，将其转为Interceptor：registry.getInterceptors(advisor);
   >
   > - 将增强器转为List<MethodInterceptor>；
   >
   >   如果是MethodInterceptor，直接加入到集合中
   >
   >   如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
   >
   > - 转换完成返回MethodInterceptor数组；

3. 如果没有拦截器链，直接执行目标方法

4. 如果有拦截器链，把需要执行的目标对象，目标方法，拦截器链的信息传入创建CglibMethodInvocation对象，并调用Object retval=mi.proceed();

5. 拦截器链的触发过程

   - 如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
   - 链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；拦截器链的机制，保证通知方法与目标方法的执行顺序；

## 总结

利用@EnableAspectJAutoProxy 开启AOP功能，该注解会给容器中注册一个组件AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；

容器的流程：

1. registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象

2. finishBeanFactoryInitialization（）初始化剩下的单实例bean

   1. 创建业务逻辑组件和切面组件

   2. AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程

   3. 组件创建完之后，判断组件是否需要增强

      是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）


执行目标方法：由代理对象执行目标方法CglibAopProxy.intercept()；

1. 得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）

2. 利用拦截器的链式机制，依次进入每一个拦截器进行执行；

3. 效果：

   正常执行：前置通知-》目标方法-》后置通知-》返回通知

   出现异常：前置通知-》目标方法-》后置通知-》异常通知

# 声明式事务

## 使用事务

1. 配置数据源dataSource

2. 给方法上标注**@Transactional**，表明当前方法是一个事务方法。

3. 还需要在配置文件上**@EnableTransactionManagement**。

4. 配置事务管理器来控制事务

~~~java
@Transactional
public void insertUser(){

}
//在配置文件中
@Bean
public PlatformTransactionManger transactionManager(){
    return new DataSourceTransactionManager(dataSource());
}
~~~

## 事务原理

`@Import({TransactionManagementConfigurationSelector.class})`

默认导入两个组件**AutoProxyRegistrar**和**ProxyTransactionManagementConfiguration**

### AutoProxyRegistrarProxy

给容器中注册一个*InfrastructureAdvisorAutoProxyCreator*组件，利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器进行调用。

### ProxyTransactionManagementConfiguration

给容器中事务增强器。事务增强器要用事务注解的信息，AnnotationTransactionAttributeSource解析事务

事务拦截器：transactionInterceptor保存了事务的属性以及事务管理器；它是个MethodInterceptor，在目标方法执行的时候执行拦截器链。首先获取事务相关的属性，获取PlatformTransactionManager，如果事先没有添加指定任何Platform，则会在容器查找标注为Platform的Bean。执行目标方法，如果异常，获取到事务管理器，利用事务管理器回滚操作。如果正常，利用事务管理器提交事务。



# 扩展组件原理

## BeanFactoryPostProcessor

# IoC

Spring容器的refresh()

prepareRefresh()刷新前的预处理

​	