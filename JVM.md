# JVM

## 1 Java类的加载过程

1. 加载

   > 将一个类的`二进制字节流`加载到内存中，生成Class字节码对象

2. 链接

   1. 验证

      > 验证`二进制字节流`是否符合Class文件规范

   2. 准备

      > 为类的`静态变量`分配内存并将其初始化为默认值

   3. 解析

      > 完成`符号引用`到`直接引用`的转换动作

3. 初始化

   > 初始化类中定义的Java代码

可以这么记忆`加链初，验准解`

------

## 2 JVM加载Class文件的原理机制

JVM加载Class文件是通过`ClassLoader（类加载器）`来实现的

`类加载器`本身就是一个类，Java中有四种类加载器

- Bootstrap ClassLoader

  > 负责加载JDK中核心的jar包中的类，比如`Integer`、`String`这种Java原生的类
  >
  > 无法被 Java 程序直接引用

- Extensions ClassLoader

  > 负责加载`/jdk1.8.0_131/jre/lib/ext`下的类，即扩展类

- Application ClassLoader

  > 负责加载用户自定义的类
  
- 用户自定义类加载器

  > 通过继承 java.lang.ClassLoader 类的方式实现

类的加载方式分为`隐式加载`和`显示加载`

- `隐式加载`是指`new`等方式创建对象时，会隐式地调用类的加载器把对应的类加载到 JVM 中
- `显示加载`是指调用`class.forName()`这种反射的方法来把所需的类加载到 JVM 中

------

## 3 JMM（JVM内存模型）

JVM内存模型具体组成如下：

- 程序计数器（线程私有）

  > 存储当前线程正在执行的方法地址（字节码指令位置），如果是`native`方法则存储`undefined`，每个线程单独一份，所以是**线程私有**区域。
  >
  > `循环`、`分支`、`方法跳转`、`异常处理`，`线程恢复`都是依赖程序计数器来完成。

- Java栈（线程私有）

  > 用于存储`基本类型`的变量和`引用变量`，而且每调用一个方法都会在栈中压入一个`栈帧`。
  >
  > `栈帧`中有`局部变量表`，`操作数栈`、`指向常量池对象的引用`，`方法返回地址`。
  >
  > 栈的大小决定了方法递归的次数。

- Java堆

  > 堆是`运行时数据区`，所有`类的实例`和`数组`都是在堆上分配内存。
  >
  > 堆分为`年轻代`和`老年代`，默认比例是`1:2`，可以通过参数 `–XX:NewRatio` 来修改。
  >
  > `年轻代`分为`eden`、`survivor0`、`survivor1`三个区，默认比例是`8:1:1`，可以同过JVM参数修改。
  >
  > `老年代`主要存放不易被回收的对象。

- 本地方法栈（线程私有）

- > 与`Java栈`类似，只有`native方法`的调用才会往本地方法栈中压栈

- 方法区

  > JDK7中也称`永久代(PermGen)`，JDK8中叫作`MetaSpace`并且存在`直接内存`中，用于存储JVM加载的`类信息`、`常量`、`静态变量`、`编译后的代码（字节码）`
  
- 直接内存

  > 使用`native函数库`直接分配`堆外内存`，`DirectBuffer` 是不经过JVM内存直接访问系统物理内存的类

![1563162620204](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\1563162620204.png)

------

## 4 如何判断一个对象是否需要被GC

- 引用计数法

  > 给每一个对象设置一个`引用计数器`，每当有一个地方引用这个对象时，就将计数器+1，引用失效时，计数器就-1。当一个对象的引用计数器为0时，说明此对象没有被引用
  >
  > 缺点：
  >
  > 无法解决循环引用问题

- GC Root可达性算法

  > 从一个`GC Roots` 对象开始向下搜索，如果一个对象到 GC Roots 没有任何引用链相连时，则说明此对象没有被引用
  >
  > `GC Roots` 对象分为：
  >
  > - `栈帧（Java栈、本地方法栈）`中`局部变量表`中引用的对象
  > - `方法区`中`类静态变量`和`常量池`引用的对象

如果一个对象没有被引用，那么这个对象还有最后一次被拯救的机会

> 如果重写了一个即将被回收的对象的`finalize()`方法，并且在该方法中重新使得该对象被引用，那么它就不会被回收



------

## 5 浅拷贝和深拷贝

- 浅拷贝

  > 复制变量，如果对象1内还有对象2，则只能复制对象1的地址，共用对象2的地址

- 深拷贝

  > 能复制变量，也能复制对象的内部对象

深浅拷贝只是一种概念，在java中实现深浅拷贝需要依赖`Cloneable`接口，和重写`Object.clone()`方法，

```java
@Override
protected T clone() {
	T clone = null;
	try {
        //仅仅只克隆当前类属于浅拷贝
		clone = (T) super.clone();
	} catch (CloneNotSupportedException e) {
		throw new RuntimeException(e);
		}
    //obj为当前类的对象引用，调用对象引用的clone方法才是属于深拷贝
    //clone.obj = obj.clone();
	return clone;
}
```

------

## 6 JVM 的永久代中会发生垃圾回收么

垃圾回收不会发生在`永久代`，如果`永久代`满了或者是超过了临界值，会触发`Full GC`。
注：Java 8 中已经移除了永久代，新加了一个叫做`MetaSpace 元数据区`的 `直接内存区`。

------

## 7 垃圾回收算法

- 复制（新生代，速度快，但需要额外内存区域）

  > 当`eden`区满了，触发`MinorGC`，将`eden`区中的存活对象复制到`survivor0`区
  >
  > 当`eden`区又满了，触发`MinorGC`，将将`eden`区和`survivor0`区中的存活对象复制到`survivor1`区
  >
  > 当`eden`区又满了，触发`MinorGC`，将将`eden`区和`survivor1`区中的存活对象复制到`survivor0`区
  >
  > 如果某个`survivor`区无法存放存活对象，则直接将`这一次新产生的存活对象`放入`老年代`，之前`survivor`区中的已经存放的对象不移动

- 标记 - 清除（老年代，效率不高，产生内存碎片）

  > 标记那些要被回收的对象，然后统一回收

- 标记 - 整理（老年代，适用于存活率较高的场景）

  > 标记那些要被回收的对象，然后统一回收，并把存活对象统一移动到内存的一端

- 分代回收（大多数使用）

  > 根据对象的生存周期，将堆分为`新生代`和`老年代`。
  >
  > 在新生代中，对象生存期短，每次回收都会有大量对象死去，那么这时就采用`复制算法`。
  >
  > 老年代里的对象存活率较高，没有额外的空间进行分配担保，根据实际情况采用`标记 - 整理`或者`标记 - 清除`算法

------

## 8 类加载器双亲委派模型机制

当一个类收到类加载请求时，它不会自己去加载这个类

而是委派给上一级类加载器，（这个类是你负责加载的吗？）如果上一级加载器仍然不负责加载，则继续往上委派

如果最上级加载器（Bootstrap ClassLoader）不负责加载，才能让下级类加载器去加载

为什么要这么做呢？

是为了确保类能够被的安全加载，假如用户自己写了一个类，`java.lang.String`，然后去加载它

假如`Application ClassLoader`直接加载了，那么程序中所有使用到`java.lang.String`类的地方全部会出错

因为这个类是java的核心Jar包中的类，必须交给`Bootstrap ClassLoader`记载

所以加载一个类的时候，会向上询问，如果上级类加载器能够加载这个类，就说明这个类是安全的

如果上级类加载器都不负责加载这个类，那说明这个类是用户自己写的类，那么往下传递到`Application ClassLoader`，让它负责加载就行了