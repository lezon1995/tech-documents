# Spring整理

## Spring中依赖注入的三种方式

- 构造器注入
- setter方法注入
- 接口注入

## Spring框架中的IoC

IoC容器由`beans`包和`context`包构成

`BeanFactory`是Spring IoC容器的核心接口，用来包装和管理前面提到的各种bean

`BeanFactory`接口提供了先进的配置机制，使得任何类型的对象的配置成为了可能

## BeanFactory和ApplicationContext有什么区别

`BeanFactory`可以理解为含有bean集合的工厂类

`BeanFactory `包含了种bean的定义，以便在接收到客户端请求时将对应的bean实例化

`ApplicationContext`继承自`BeanFactory`，扩展了其他的功能

- 提供了支持国际化的文本消息
- 统一的资源文件读取方式
- 已在监听器中注册的bean的事件

我们常用的`ClassPathXmlApplicationContext`就间接地实现了`ApplicationContext`接口

下面是三种常用`ApplicationContext` 实现方式

1. ClassPathXmlApplicationContext

   > 从`classpath`的XML配置文件中读取上下文

2. FileSystemXmlApplicationContext

   > 由`文件系统`中的XML配置文件读取上下文

3. XmlWebApplicationContext

   > 由`Web应用`的XML文件读取上下文

## Spring有几种配置方式

1. 基于`XML`的配置

   > applicationContext.xml中配置<bean id="xxx">

2. 基于`注解`的配置

   > 使用`@Bean`、`@Configuration`等注解
   >
   > 开启注解配置`<context:annotation-config/>`

3. 基于`Java`的配置

   > 配合`注解`使用

## Spring Bean的生命周期

`BeanFactory` 负责管理在spring容器中被创建的bean的生命周期

Bean的生命周期由`2组回调（CallBack）方法`组成

- 初始化之后
- 销毁之前

Spring框架提供 `4` 种方法来管理Bean的生命周期事件

- `InitializingBean`和`DisposableBean`回调接口

- 针对特殊行为的其他`Aware`接口

- Bean配置文件中的`init()`方法和`destroy()`方法

  ```xml
  <beans>
  	<bean id="demoBean" class="com.howtodoinjava.task.DemoBean" init-method="customInit" destroy-method="customDestroy"></bean>
  </beans>
  ```

- `@PostConstruct`和@`PreDestroy`注解方式

## Spring Bean的作用域

Spring容器中的bean可以分为 `5` 个范围

- `singleton`

  > 整个程序中只有一份

- `prototype`

  > 为每个bean的请求创建一份

- `request`

  > 为自客户端的网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收

- `session`

  > 每个session中有一个bean的实例，在session过期后，bean会随之失效

- `global-session`

  > 如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中

## Spring中单例Bean是线程安全的么

**不是**

Spring中没有对SIngleton的bean做任何多线程的封装处理

如果需要实现线程安全的话，需要开发者自行处理

实际上，大部分单例的bean一般都是`service`、`dao`之类的bean

这类bean并没有多种状态可变，所以就算多线程调用的也不会有什么安全问题

但是如果是某种状态较多的bean的话，则需要自行保证线程安全，比如将该bean的作用域设为`prototype`

## Spring中注入一个Collection

- List

  ```xml
  <!-- java.util.List -->
  <property name="customList">
      <list>
          <value>INDIA</value>
          <value>Pakistan</value>
          <value>USA</value>
          <value>UK</value>
      </list>
  </property>
  ```

- Set

  ```xml
  <!-- java.util.Set -->
  <property name="customSet">
      <set>
          <value>INDIA</value>
          <value>Pakistan</value>
          <value>USA</value>
          <value>UK</value>
      </set>
  </property>
  ```

- map

  ```xml
  <!-- java.util.Map -->
  <property name="customMap">
      <map>
          <entry key="1" value="INDIA"/>
          <entry key="2" value="Pakistan"/>
          <entry key="3" value="USA"/>
          <entry key="4" value="UK"/>
      </map>
  </property>
  ```

- properties

  ```xml
  <!-- java.util.Properties -->
  <property name="customProperies">
      <props>
          <prop key="admin">admin@nospam.com</prop>
          <prop key="support">support@nospam.com</prop>
      </props>
  </property>
  ```

## Spring Bean的自动装配

- XML配置装配

  ```xml
  <bean id="employeeDAO" class="com.xxx.EmployeeDAOImpl" autowire="byName" />
  ```

- 注解装配

  > 使用`@Autowired`注解来自动装配指定的bean
  >
  > 前提是需要开启注解配置`<context:annotation-config />`
  >
  > 或者配置`AutowiredAnnotationBeanPostProcessor`，也能达到相同的效果

  ```xml
  <bean class ="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
  ```

## 5种自动装配模式

- no

  > 这是Spring框架的**默认设置**，在该设置下自动装配是关闭的，开发者需要自行在bean定义中用标签明确的设置依赖关系。

- byName

  > 该选项可以根据bean名称设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的名称自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

- byType

  > 该选项可以根据bean类型设置依赖关系。当向一个bean中自动装配一个属性时，容器将根据bean的类型自动在在配置文件中查询一个匹配的bean。如果找到的话，就装配这个属性，如果没找到的话就报错。

- constructor

  > 造器的自动装配和`byType模式`类似，但是仅仅适用于与有构造器相同参数的bean，如果在容器中没有找到与构造器参数类型一致的bean，那么将会抛出异常。

- autoDetect

  > 该模式自动探测使用构造器自动装配或者byType自动装配。首先，首先会尝试找合适的带参数的`构造器`，如果找到的话就是用构造器自动装配，如果在bean内部没有找到相应的构造器或者是无参构造器，容器就会自动选择`byTpe`的自动装配方式。

## @Required注解

`@Required`适用于属性的`setter`方法

表示某个属性必须赋值

如果没有赋值的话则会抛出`BeanInitializationException`异常

前提是开启了`注解配置`

或者配置了`RequiredAnnotationBeanPostProcesso`

## @Qualifier注解

配合`@Autowired`注解一起使用

当IoC容器中存在多个相同类型的Bean

此时需要使用`@Qualifie("beanName")`来指定具体装配那个类

## Spring 框架中都用到了哪些设计模式

- 工厂模式

  > Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象

- 代理模式

  > Spring AOP 功能的实现
  >
  > 比如`JdkDynamicAopProxy`和`Cglib2AopProxy`

- 单例模式

  > 在spring配置文件中定义的bean默认为单例模式。

- 模板方法模式

  > 用来解决代码重复的问题。比如RestTemplate, JmsTemplate, JpaTemplate。

- 装饰者模式

  > 一种类名中含有`Wrapper`，一种类名中含有`Decorator`，基本上都是动态地给一个对象添加一些额外的职责。 

- 观察者模式

  > Spring中Observer模式常用的地方是listener的实现。如`ApplicationListener`。

- 适配器模式

  > 在Spring的Aop中，使用的`Advice`（通知）来增强被代理类的功能。

- 策略模式

  > spring中在实例化对象的时候用到Strategy模式
  > 在SimpleInstantiationStrategy中有如下代码说明了策略模式的使用情况