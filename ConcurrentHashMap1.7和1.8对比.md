# ConcurrentHashMap1.7和1.8对比

## 数据结构

### 1.7中采用```Segment```+```HashEntry```的方式实现

​	![img](https://upload-images.jianshu.io/upload_images/2184951-af57d9d50ae9f547.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/767/format/webp)

`ConcurrentHashMap`**初始化**时，计算出`Segment`数组的大小`ssize`和每个`Segment`中`HashEntry`数组的大小`cap`，并初始化`Segment`数组的第一个元素；

其中`ssize`大小为**2的幂次方**，**默认为16**，`cap`大小也是**2的幂次方**，**最小值为2**，最终结果根据初始化容量`initialCapacity`进行计算，计算过程如下

```java
if (c * ssize < initialCapacity)
    ++c;
int cap = MIN_SEGMENT_TABLE_CAPACITY;
while (cap < c)
    cap <<= 1;
```

因为```Segment```继承了**ReentrantLock**，所有segment是线程安全的

### 1.8中放弃了```Segment```分段锁的设计，使用的是```Node```+```CAS```+```Synchronized```来保证线程安全性

![img](https://upload-images.jianshu.io/upload_images/2184951-d9933a0302f72d47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/768/format/webp)

只有在第一次执行`put`方法是才会初始化`Node`数组



## put操作

### 1.7 put

当执行`put`方法插入数据的时候，根据key的hash值，在`Segment`数组中找到对应的位置

如果当前位置没有值，则通过**CAS**进行赋值，接着执行`Segment`的`put`方法通过加锁机制插入数据

假如有线程AB同时执行相同`Segment`的`put`方法

> 线程A 执行`tryLock`方法成功获取锁，然后把`HashEntry`对象插入到相应位置
>
> 线程B 尝试获取锁失败，则执行`scanAndLockForPut()`方法，通过重复执行`tryLock()`方法尝试获取锁
>
> 在**多处理器**环境重复**64**次，**单处理器**环境重复**1**次，当执行`tryLock()`方法的次数超过上限时，则执行`lock()`方法挂起线程B
>
> 当线程A执行完插入操作时，会通过`unlock`方法施放锁，接着唤醒线程B继续执行

### 1.8 put

当执行`put`方法插入数据的时候，根据key的hash值在`Node`数组中找到相应的位置

如果当前位置的`Node`还没有初始化，则通过CAS插入数据

```java
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    //如果当前位置的`Node`还没有初始化，则通过CAS插入数据
    if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
        break;                   // no lock when adding to empty bin
}
```

如果当前位置的Node已经有值，则对该节点加`synchronized`锁，然后从该节点开始遍历，直到**插入**新的节点或者**更新**新的节点

```java
if (fh >= 0) {
    binCount = 1;
    for (Node<K,V> e = f;; ++binCount) {
        K ek;
        if (e.hash == hash &&
            ((ek = e.key) == key ||
             (ek != null && key.equals(ek)))) {
            oldVal = e.val;
            if (!onlyIfAbsent)
                e.val = value;
            break;
        }
        Node<K,V> pred = e;
        if ((e = e.next) == null) {
            pred.next = new Node<K,V>(hash, key, value, null);
            break;
        }
    }
}
```

如果当前节点是`TreeBin`类型，说明该节点下的链表已经进化成**红黑树**结构，则通过`putTreeVal`方法向红黑树中插入新的节点

```java
else if (f instanceof TreeBin) {
    Node<K,V> p;
    binCount = 2;
    if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
        oldVal = p.val;
        if (!onlyIfAbsent)
            p.val = value;
    }
}
```

如果`binCount`不为0，说明`put`操作对数据产生了影响，如果当前链表的节点个数达到了**8**个，则通过`treeifyBin`方法将**链表**转化为**红黑树**



## size 操作

### 1.7 size实现

统计每个`segment`对象中的元素个数，然后进行累加

但是这种方式计算出来的结果不一定准确

因为在计算后面的`segment`的元素个数时

前面计算过了的`segment`可能有数据的新增或删除

计算方式为：

先采用不加锁的方式，连续计算两次

> **如果两次结果相等，说明计算结果准确**
>
> **如果两次结果不相等，说明计算过程中出现了并发新增或者删除操作**
>
> 于是给每个`segment`加锁，然后再次计算
>

```java
try {
    for (;;) {
        if (retries++ == RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                ensureSegment(j).lock(); // force creation
        }
        sum = 0L;
        size = 0;
        overflow = false;
        for (int j = 0; j < segments.length; ++j) {
            Segment<K,V> seg = segmentAt(segments, j);
            if (seg != null) {
                sum += seg.modCount;
                int c = seg.count;
                if (c < 0 || (size += c) < 0)
                    overflow = true;
            }
        }
        if (sum == last)
            break;
        last = sum;
    }
} finally {
    if (retries > RETRIES_BEFORE_LOCK) {
        for (int j = 0; j < segments.length; ++j)
            segmentAt(segments, j).unlock();
    }
}
```

### 1.8 size实现

使用一个`volatile`类型的变量`baseCount`记录元素的个数

当新增或者删除节点的时候会调用，`addCount()`更新`baseCount`

```java
if ((as = counterCells) != null ||
    !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
    CounterCell a; long v; int m;
    boolean uncontended = true;
    if (as == null || (m = as.length - 1) < 0 ||
        (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
        !(uncontended =
          U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
        fullAddCount(x, uncontended);
        return;
    }
    if (check <= 1)
        return;
    s = sumCount();
}
```

初始化时`counterCells`为空

在并发量很高时，如果存在两个线程同时执行`CAS`修改`baseCount`值

则失败的线程会继续执行方法体中的逻辑

使用`CounterCell`记录元素个数的变化



如果`CounterCell`数组`counterCells`为空

调用`fullAddCount()`方法进行初始化

并插入对应的记录数，通过`CAS`设置cellsBusy字段

只有设置成功的线程才能初始化`CounterCell`数组

```java
else if (cellsBusy == 0 && counterCells == as &&
         U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
    boolean init = false;
    try {                           // Initialize table
        if (counterCells == as) {
            CounterCell[] rs = new CounterCell[2];
            rs[h & 1] = new CounterCell(x);
            counterCells = rs;
            init = true;
        }
    } finally {
        cellsBusy = 0;
    }
    if (init)
        break;
}
```

如果通过`CAS`设置cellsBusy字段失败的话

则继续尝试通过`CAS`修改`baseCount`字段

如果修改`baseCount`字段成功的话，就退出循环

```java
else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
    break; 
```

否则继续循环插入`CounterCell`对象；

所以在1.8中的`size`实现比1.7简单多，因为元素个数保存`baseCount`中，部分元素的变化个数保存在`CounterCell`数组中，实现如下：

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}

final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

通过累加`baseCount`和`CounterCell`数组中的数量，即可得到元素的总个数；

