# Java集合

## 1 ArrayList 和 Vector 的区别

- ArrayList 

  > 非线程安全，扩容为原来的1.5倍

- Vector 

  > 线程安全，扩容为原来的2倍

## 2 ArrayList，Vector，LinkedList 的存储性能和特性

`ArrayList` 和 `Vector` 都是使用`数组`方式存储数据，查询效率高，增删效率低

`LinkedList` 使用`双向链表`的方式存储数据，查询效率低，增删效率高

## 3 快速失败 (fail-fast) 和安全失败 (fail-safe)

- `fail-fast`

  > 在遍历一个集合的时候，如果调用集合的方法进行增删操作，会立即抛出`ConcurrentModificationException`异常
  >
  > 因为在原集合上增删，会影响遍历的过程

- `fail-safe`

  > 通过`Iterator`迭代器来遍历一个集合，是可以通过迭代器的`remove()`方法来操作集合的
  >
  > 因为迭代器是操作集合的拷贝