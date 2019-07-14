## HashMap 1.8

## hashmap构造函数

```java
 //最大容量 2的30次方
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //默认的加载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    //哈希桶，存放链表。 长度是2的N次方，或者初始化时为0.
    transient Node<K,V>[] table;

    //加载因子，用于计算哈希表元素数量的阈值。  threshold = 哈希桶.length * loadFactor;
    final float loadFactor;
    //哈希表内元素数量的阈值，当哈希表内元素数量超过阈值时，会发生扩容resize()。
    int threshold;

    public HashMap() {
        //默认构造函数，赋值加载因子为默认的0.75f
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    public HashMap(int initialCapacity) {
        //指定初始化容量的构造函数
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    //同时指定初始化容量 以及 加载因子， 用的很少，一般不会修改loadFactor
    public HashMap(int initialCapacity, float loadFactor) {
        //边界处理
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //初始容量最大不能超过2的30次方
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //显然加载因子不能为负数
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        //设置阈值为  =初始化容量的 2的n次方的值
        this.threshold = tableSizeFor(initialCapacity);
    }
    //新建一个哈希表，同时将另一个map m 里的所有元素加入表中
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```



## hashmap的hash算法

**扰动函数**就是为了解决hash碰撞的。它会综合hash值高位和低位的特征，并存放在低位，因此在与运算时，相当于高低位一起参与了运算，以减少hash碰撞的概率。（在JDK8之前，扰动函数会扰动四次，JDK8简化了这个操作）

```java
//通过key的hashCode的高16位与低16位做异或运算，相同为0，不同为1，目的是为了得到更离散的hash值，在后面会对数组长度减1做与运算。
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```



## hashmap的put方法

```java
//1 判断Node数组是否初始化，如果没有就调用resize()方法初始化
if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
//2 对当前key应该处于数组中的哪个下标，方法是用当前node数组的长度减1对key的hash进行与运算，如果数组当前下标为空，则直接放置新的node
if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
//3 如果数组当前下标不为空，则说明产生了hash碰撞
else {
            Node<K,V> e; K k;
    		//4 如果节点key存在，则直接覆盖value
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
    		//5 判断该链p是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向下面
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
    		//6 该链为链表遍历p，判断链表长度是否大于8，如果大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //7 链表长度大于8转换为红黑树进行处理
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //8 遍历过程中若发现key已经存在直接覆盖value即可
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
```

## hashmap扩容方法

```java
final Node<K,V>[] resize() {
        //oldTab 为当前表的哈希桶
        Node<K,V>[] oldTab = table;
        //当前哈希桶的容量 length
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //当前的阈值
        int oldThr = threshold;
        //初始化新的容量和阈值为0
        int newCap, newThr = 0;
        //如果当前容量大于0
        if (oldCap > 0) {
            //如果当前容量已经到达上限
            if (oldCap >= MAXIMUM_CAPACITY) {
                //则设置阈值是2的31次方-1
                threshold = Integer.MAX_VALUE;
                //同时返回当前的哈希桶，不再扩容
                return oldTab;
            }//否则新的容量为旧的容量的两倍。 
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)//如果旧的容量大于等于默认初始容量16
                //那么新的阈值也等于旧的阈值的两倍
                newThr = oldThr << 1; // double threshold
        }//如果当前表是空的，但是有阈值。代表是初始化时指定了容量、阈值的情况
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;//那么新表的容量就等于旧的阈值
        else {}//如果当前表是空的，而且也没有阈值。代表是初始化时没有任何容量/阈值参数的情况               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;//此时新表的容量为默认的容量 16
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//新的阈值为默认容量16 * 默认加载因子0.75f = 12
        }
        if (newThr == 0) {//如果新的阈值是0，对应的是  当前表是空的，但是有阈值的情况
            float ft = (float)newCap * loadFactor;//根据新表容量 和 加载因子 求出新的阈值
            //进行越界修复
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //更新阈值 
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //根据新的容量 构建新的哈希桶
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //更新哈希桶引用
        table = newTab;
        //如果以前的哈希桶中有元素
        //下面开始将当前哈希桶中的所有节点转移到新的哈希桶中
        if (oldTab != null) {
            //遍历老的哈希桶
            for (int j = 0; j < oldCap; ++j) {
                //取出当前的节点 e
                Node<K,V> e;
                //如果当前桶中有元素,则将链表赋值给e
                if ((e = oldTab[j]) != null) {
                    //将原哈希桶置空以便GC
                    oldTab[j] = null;
                    //如果当前链表中就一个元素，（没有发生哈希碰撞）
                    if (e.next == null)
                        //直接将这个元素放置在新的哈希桶里。
                        //注意这里取下标 是用 哈希值 与 桶的长度-1 。 由于桶的长度是2的n次方，这么做其实是等于 一个模运算。但是效率更高
                        newTab[e.hash & (newCap - 1)] = e;
                        //如果发生过哈希碰撞 ,而且是节点数超过8个，转化成了红黑树（暂且不谈 避免过于复杂， 后续专门研究一下红黑树）
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //如果发生过哈希碰撞，节点数小于8个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。
                    else { // preserve order
                        //因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位=  low位+原哈希桶容量
                        //低位链表的头结点、尾节点
                        Node<K,V> loHead = null, loTail = null;
                        //高位链表的头节点、尾节点
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;//临时节点 存放e的下一个节点
                        do {
                            next = e.next;
                            //这里又是一个利用位运算 代替常规运算的高效点： 利用哈希值 与 旧的容量，可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位
                            if ((e.hash & oldCap) == 0) {
                                //给头尾节点指针赋值
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }//高位也是相同的逻辑
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }//循环直到链表结束
                        } while ((e = next) != null);
                        //将低位链表存放在原index处，
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        //将高位链表存放在新index处
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```



## 链表的Node属性

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;//哈希值
        final K key;//key
        V value;//value
        Node<K,V> next;//链表后置节点

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        //每一个节点的hash值，是将key的hashCode 和 value的hashCode 亦或得到的。
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
        //设置新的value 同时返回旧value
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```





## 与HashTable的区别

- 与之相比**HashTable**是**线程安全**的，且**不允许key、value是null**。
- **HashTable默认容量是11**。
- HashTable是**直接使用key的hashCode(key.hashCode())作为hash值**，不像HashMap内部**使用static final int hash(Object key)扰动函数对key的hashCode进行扰动后作为hash值**。
- **HashTable取哈希桶下标是直接用模运算%**.（因为其默认容量也不是2的n次方。所以也无法用位运算替代模运算）
- 扩容时，**新容量是原来的2倍+1**。int newCapacity = (oldCapacity << 1) + 1;
- Hashtable是**Dictionary的子类**同时也**实现了Map接口**，HashMap是Map接口的一个实现类；



## 小结： 

* 运算尽量都用位运算代替，更高效。 
* 对于扩容导致需要新建数组存放更多元素时，除了要将老数组中的元素迁移过来，也记得将老数组中的引用置null，以便GC 
* 取下标 是用 哈希值 与运算 （桶的长度-1） i = (n - 1) & hash。 由于桶的长度是2的n次方，这么做其实是等于 一个模运算。但是效率更高 
* 扩容时，如果发生过哈希碰撞，节点数小于8个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。 
* 因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位= low位+原哈希桶容量 
* 利用哈希值 与运算 旧的容量 ，if ((e.hash & oldCap) == 0),可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位。这里又是一个利用位运算 代替常规运算的高效点 
* 如果追加节点后，链表数量>=8，则转化为红黑树 
* 插入节点操作时，有一些空实现的函数，用作LinkedHashMap重写使用。