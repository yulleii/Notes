随着需求的变化，代码改动过大的原因是什么？

学习新的**设计模式**来重构我们的代码。前人根据自己的经验总结出来的，代表了代码设计的最佳实践。

从优秀的开源框架中，学习如何去使用。

![](Snipaste_2019-07-23_21-30-45.png)

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

