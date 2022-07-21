### 基础

#### 位移

右移运算也相当于做除法运算，被除数为 2^k。例如，我们执行149 >> 4，相当于执行了 149/16 = 9。

左移运算也相当于做乘法运算，乘积因子为 2^k。例如，我们执行149 << 4，相当于执行了 149*16 = 2384。

### 集合

![image-20220602092936138](C:\Users\麦苗\AppData\Roaming\Typora\typora-user-images\image-20220602092936138.png)

#### ArrayList

1. 数组结构，有序，可重复
2. 非线程安全
3. 扩容机制

​	无参构造，初始化大小0，第一次扩容为10，之后为容量的1.5倍

​	有参构造，初始化为参数值，之后扩容为1.5倍

​	扩容使用的是copyOf

```java
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    // 如果等于空对象数组 则使用默认大小10
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 原数组长度 + 原数组除以2
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //复制原有数组到新数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

#### Vector

1. 数组结构，有序，可重复
2. 线程安全 使用 synchronized
3. 扩容机制

​	无参构造，初始化大小10，之后为容量的2倍

​	扩容使用的是copyOf

```java
public Vector() {
    // 默认10
    this(10);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    
    // capacityIncrement： 这个是增量 在实例化时设置 默认为0 
    // 默认扩容是2倍  如果设置了增量值 则等于 原有容量加增量值
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```



#### LinkedList

1. 双向链表，双端队列


```java
transient Node<E> first;

/**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
transient Node<E> last;

/**
     * Constructs an empty list.
     */
public LinkedList() {
}

// 添加
public boolean add(E e) {
    linkLast(e); // 默认添加到链尾
    return true;
}

//构建链表
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

// node 链表结构
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#### HashSet

1. 底层基于hashMap实现

2. 元素不重复，可以存null

3. 保证插入元素的顺序
4. 数组+链表，红黑树，
5. 扩容机制

​	k-v个数 > 扩容阈值 = table数组容量 * 负载因子（默认是0.75）

​	如果链表个数超过8，并且table 大于等于64 则转化为红黑树

![img](https://pic4.zhimg.com/80/v2-48dd5d2224e09f05eb93dad425990c03_720w.jpg)

```java
static final int hash(Object key) {
    int h;
    // 哈希得到key的值 然后存放到数据表中
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

```java
transient Node<K,V>[] table; // 一个node 数组
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash; 
    final K key;
    V value;
    Node<K,V> next; //单向链表

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

#### LinkedHashSet

​	1,底层是一个LinkedHashMap,维护一个数组+双向链表

​	2，元素节点 有 before 和 after 属性， 这样就可以保证插入的顺序

#### TreeSet

​		底层基于  TreeMap 红黑树结构

### HashMap

​	1，数组+单向链表

​	2，阈值(threshold) = 负载因子(loadFactor) x 容量(capacity) 

当HashMap中table数组(也称为桶)长度 >= 阈值(threshold) 就会自动进行扩容。

扩容的规则是这样的，因为table数组长度必须是2的次方数，扩容其实每次都是按照上一次tableSize位运算得到的就是做一次左移1位运算，

假设当前tableSize是16的话 16转为二进制再向左移一位就得到了32 即 16 << 1 == 32 即扩容后的容量，也就是说扩容后的容量是当前

容量的两倍，但记住HashMap的扩容是采用当前容量向左位移一位（newtableSize = tableSize << 1），得到的扩容后容量，而不是当前容量x2



### Hashtable

​	数组，线程安全

​	Map.Entry<K,V> 数组

​	默认大小11，加载因子0.75

​	扩容  2倍+1

```java
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;

    // overflow-conscious code
    // old * 2
    int newCapacity = (oldCapacity << 1) + 1;
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            // Keep running with MAX_ARRAY_SIZE buckets
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

    modCount++;
    //计算负载因子
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;

    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;

            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```

### TreeMap

1. 红黑树
2. 保证key的有序性
3. 不能插入null

## 函数式方法

|      name      |       type       |           description           |
| :------------: | :--------------: | :-----------------------------: |
|    Consumer    |  Consumer< T >   |       接收T对象，不返回值       |
|   Predicate    |  Predicate< T >  |     接收T对象并返回boolean      |
|    Function    | Function< T, R > |      接收T对象，返回R对象       |
|    Supplier    |  Supplier< T >   | 提供T对象（例如工厂），不接收值 |
| UnaryOperator  |  UnaryOperator   |      接收T对象，返回T对象       |
| BinaryOperator |  BinaryOperator  |    接收两个T对象，返回T对象     |