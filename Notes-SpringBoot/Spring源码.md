- 看过Spring的源码吗？
- IOC应用及原理
- AOP应用及原理
- 事务
- Spring涉及的设计模式
- 谈谈Spring Bean的生命周期和作用域
- Spring MVC的工作原理
- Spring vs Spring MVC
- Spring vs Spring Boot
- 了解微服务，Spring Cloud

# IoC vs DI

Inverse of Control 反转控制的概念，就是将原本在程序中手动创建UserService对象的控制权，交由Spring框架管理，简单说，就是创建UserService对象控制权被反转到了Spring框架。

依赖注入（Dependency Inject）：在Spring框架负责创建Bean对象时，动态的将依赖对象注入到Bean组件

> IoC 和 DI的区别？
>
> IoC 控制反转，指将对象的创建权，反转到Spring容器 ， DI 依赖注入，指Spring创建对象的过程中，将对象依赖属性通过配置进行注入

# IOC应用及原理

# Spring Bean的生命周期和作用域

## Bean的生命周期

**1.bean定义**：在配置文件里面用<bean></bean>来进行定义

**2.bean初始化**

- 在配置文件中通过init-method属性来完成
- 实现org.springframwork.beans.factory.InitializingBean接口

**3.bean调用**

有三种方式可以得到bean实例，并调用

- 使用类构造器实例化 `<bean id="bean1" class="cn.itcast.spring.b_instance.Bean1"></bean>`

- 使用静态工厂方法实例化(简单工厂模式)：`<bean id="bean2" class="cn.itcast.spring.b_instance.Bean2Factory" factory-method="getBean2"></bean>`

- 使用实例工厂方法实例化(工厂方法模式)

  ```
  <bean id="bean3Factory" class="cn.itcast.spring.b_instance.Bean3Factory"></bean>
  <bean id="bean3" factory-bean="bean3Factory" factory-method="getBean3"></bean>
  ```

**4.bean销毁**

- 使用配置属性指定的destroy-method属性
- 实现org.springframwork.bean.factory.DisposeableBean接口

## 作用域

**singleton**：Spring IoC容器中只能存在一个共享的bean实例，并且所有对bean的请求，只要与该bean相匹配。则只会返回bean的同一实例

**prototype**：Prototype作用域的bean会导致在每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。根据经验，对于所有有状态的bean应该使用prototype，而没有状态的则使用singleton。

**request**：在一次HTTP请求中，一个bean定义对应一个实例，即每次http请求将会有各自的bean实例，它们根据某个bean定义创建而成。该作用域仅在基于web的SpringApplicationContext情形下有效。

**seesion**：在一个HttpSession中，一个bean定义对应一个实例。和上述request相同，只能适用于基于Web的SpringApplicationContext

**global session**：在一个全局的HttpSession中，一个bean定义对应一个实例。只能适用于基于Web的SpringApplicationContext

# Spring事务

事务就是对一系列的数据库操作进行统一的提交或者回滚操作。如果插入成功，那么一起成功。如果中间有一条数据出现异常，那么回滚之前的所有操作。防止出现脏数据，防止数据库出现问题。  

为了避免这种情况，Spring也有自己的事务管理机制，一般使用TransactionMananger进行管理，可以通过Spring注入来完成此功能。提供了几个关于事务处理的类：

- 