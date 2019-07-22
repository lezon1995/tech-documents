# MyBatis

## MyBatis的缓存机制

MyBatis的缓存分为`一级缓存`和`二级缓存`

一级缓存放在`session`里面，默认就有

二级缓存放在它的`命名空间`里，默认是不打开的

使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态)，可在它的映射文件中配置

## MyBatis的动态标签

- trim
- where
- set
- foreach
- if
- choose
- when
- otherwise
- bind

## #{}和${}的区别

- `#{}`是预编译处理，`${}`是字符串替换。
- Mybatis在处理`#{}`时，会将sql中的`#{}`替换为`?号`，调用`PreparedStatement`的`set方法`来赋值；
- Mybatis在处理`${}`时，就是把`${}`替换成`变量的值`。
- 使用`#{}`可以有效的防止SQL注入，提高系统安全性。

## 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？

- `Hibernate`属于全自动ORM映射工具，使用`Hibernate`查询关联对象时，可以根据对象关系模型直接获取，所以它是全自动的。
- Mybatis在查询关联对象时，需要手动编写sql来完成，所以称之为半自动ORM映射工具。

## Mybatis延迟加载原理

- Mybatis仅支持`association（1对1）`关联对象和`collection（1对多）`关联集合对象的延迟加载。在Mybatis配置文件中，可以配置是否启用延迟加载`lazyLoadingEnabled = true|false`

原理是通过`CGLIB`创建目标对象的代理对象，当调用目标方法时，进入拦截器方法

比如调用`a.getB().getName()`，拦截器invoke()方法发现`a.getB()`是`null`值

那么就会单独发送事先保存好的查询关联B对象的sql，把B查询出来，然0后调用`a.setB(b)`，于是a的对象b属性就有值了，接着完成`a.getB().getName()`方法的调用。这就是延迟加载的基本原理

## Mybatis的Xml映射文件和Mybatis内部数据结构之间的映射关系

```xml
<parameterMap type="Book.dao.Book" id="BookParameterMap">
  <parameter property="bookName" resultMap="BookResultMap" />  
  <parameter property="bookPrice" resultMap="BookResultMap" />  
 </parameterMap>
```

在Xml映射文件中

`<parameterMap>`标签会被解析为`ParameterMap`对象

其子标签`<parameter>`会被解析为`ParameterMapping`对象

```xml
 <resultMap type="Book.dao.Book" id="BookResultMap">
  <id column="id" property="id"/>
  <result column="name" property="bookName"/>
  <result column="price" property="bookPrice"/>
 </resultMap>
```

`<resultMap>`标签会被解析为`ResultMap`对象

其子标签`<id>`、`<result>`会被解析为`ResultMapping`对象

`<select>`、`<insert>`、`<update>`、`<delete>`标签均会被解析为`MappedStatement`对象

标签内的`SQL语句`会被解析为`BoundSql`对象

## 接口绑定的实现方式

- 注解绑定

  > 在接口的方法上面加上`@Select`、`@Update`等注解里面包含Sql语句来绑定

- XML绑定

  > 通过xml里面写SQL来绑定
  >
  > 需要指定xml映射文件里面的`namespace`必须为接口的全路径名

## Mybatis中如何执行批处理

使用BatchExecutor完成批处理

## Mybatis都有哪些Executor执行器

- SimpleExecutor

  > 每执行一次`update`或`select`
  >
  > 就开启一个`Statement`对象，用完立刻关闭Statement对象

- ReuseExecutor

  > 执行`update`或`select`时
  >
  > 以sql作为`key`查找`Statement`对象，存在就使用，不存在就创建
  >
  > 用完后，不关闭`Statement`对象，而是放置于`statementMap`中

- BatchExecutor

  > 内部定义的一个`List<Statement> statementList`集合用于存放多个`Statement`
  >
  > 然后调用`Statement`的`executeBatch()`方法来实现批处理

## Mybatis中如何指定Executor

在Mybatis配置文件中

可以指定默认的`ExecutorType`执行器类型

也可以手动给`DefaultSqlSessionFactory`的创建`SqlSession`的方法传递`ExecutorType`类型参数

## Mybatis是否可以映射Enum枚举类

**可以**

不单可以映射枚举类

Mybatis可以映射任何对象到表的一列上

映射方式为自定义一个`TypeHandler`，实现TypeHandler的`setParameter()`和`getResult()`接口方法

`TypeHandler`有两个作用

- `javaType`至`jdbcType`的转换
- `jdbcType`至`javaType`的转换

体现为`setParameter()`和`getResult()`两个方法，分别代表设置sql问号占位符参数和获取列查询结果

## mapper中传递多个参数

- 直接在方法中传递参数，`xml`文件用`#{0}` `#{1}`来获取
- 使用 `@param("xxx")` 注解：这样可以直接在xml文件中通过`#{value}`来获取

## resultType和resultMap的区别

- 类的名字和数据库相同时，可以直接设置resultType参数为Pojo类
- 若不同，需要设置resultMap 将结果名字和Pojo名字进行转换

## MyBatis的mapper接口调用时有哪些要求

- Mapper`接口方法名`和mapper.xml中定义的每个sql的`id`相同
- Mapper接口方法的`输入参数类型`和mapper.xml中定义的每个sql的`parameterType`的类型相同
- Mapper接口方法的`输出参数类型`和mapper.xml中定义的每个sql的`resultType`的类型相同
- Mapper.xml文件中的`namespace`即是mapper`接口的类路径`