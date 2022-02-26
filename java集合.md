---
title: java集合
author: Wang Yue Niao
top: false
toc: true
date: 2020-11-02 20:51:25
tags: java集合
categories: java集合
---

# java集合

## Collection

- 特点：代表任意类型的对象，无序、无下标、不能重复。
- 特点：
  - boolean add(Object obj)              //添加一个对象。
  - boolean addAll(Collect c)             //将一个集合中的所有对象添加到此集合中。
  - void clear()                                     //清空此集合中的所有对象。
  - bollean contains(Object o)         //检查此集合中是否包含o对象。
  - bollean equals(Object o)           //比较此集合是否与指定对象相等。
  - bollean isEmpty()                       //判断此集合是否为空。
  - boolean remove(Object o)      //在此集合中移除o对象。
  - int size()                                     //返回此集合中的元素个数。
  - Object[] toArray()                    //将此集合转换成数组。

### List

#### ArrayList

```java
//默认初始容量
private static final int DEFAULT_CAPACITY = 10;
//用于空实例的共享空数组实例。
private static final Object[] EMPTY_ELEMENTDATA = {};
//共享的空数组实例，用于默认大小的空实例。我们将其与EMPTY_ELEMENTDATA区别开来，以了解添加第一个元素时需要充气多少。
 private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//当前数据对象存放地方，当前对象不参与序列化
transient Object[] elementData
//ArrayList的大小（它包含的元素数）。
private int size;
```

##### 构造函数

- 无参构造函数：如果不传参默认无参创建ArrayList对象

  - 注意：此时我们创建的ArrayList对象中的elementData中的长度是1，size是0,当进行第一次add的时候，elementData将会变成默认的长度：10.

  - ```java
     public ArrayList() {
            this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
        }
    ```

- 带int类型的构造函数

  - 如果传入参数，则代表指定ArrayList的初始数组长度，传入参数如果是大于等于0，则使用用户的参数初始化，如果用户传入的参数小于0，则抛出异常，构造方法如下：

  - ```java
    public ArrayList(int initialCapacity) {
            if (initialCapacity > 0) {
                this.elementData = new Object[initialCapacity];
            } else if (initialCapacity == 0) {
                this.elementData = EMPTY_ELEMENTDATA;
            } else {
                throw new IllegalArgumentException("Illegal Capacity: "+
                                                   initialCapacity);
            }
        }
    ```

    
    

- 带Collection对象的构造函数

  - 1）将collection对象转换成数组，然后将数组的地址的赋给elementData。

    2）更新size的值，同时判断size的大小，如果是size等于0，直接将空对象EMPTY_ELEMENTDATA的地址赋给elementData

    3）如果size的值大于0，则执行Arrays.copy方法，把collection对象的内容（可以理解为深拷贝）copy到elementData中。

    注意：this.elementData = arg0.toArray(); 这里执行的简单赋值时浅拷贝，所以要执行Arrays,copy 做深拷贝

  - ```java
     public ArrayList(Collection<? extends E> c) {
            elementData = c.toArray();
            if ((size = elementData.length) != 0) {
                // c.toArray might (incorrectly) not return Object[] (see 6260652)
                if (elementData.getClass() != Object[].class)
                    elementData = Arrays.copyOf(elementData, size, Object[].class);
            } else {
                // replace with empty array.
                this.elementData = EMPTY_ELEMENTDATA;
            }
        }
    ```

#### Vector

- 与ArrayList比较
  - Vector是同步的，因此开销就比ArrayList大，访问速度更慢。最好使用ArrayList而不是Vector，因为同步操作完全可以有程序员自己来控制。
  - Vector每次扩容请求其大小的2倍（也可以通过构造函数设置增长的容量），而ArrayList是1.5倍

#### LikedList

- ArrayList在内存中是连续的，LinkedList则是不连续的
- ArrayList节点上只存数据，LinkedList节点上除数据外还存有下一个或相邻两个节点的地址

**优点：**

- 插入、删除灵活 (不必移动节点，只要改变节点中的指针，但是需要先定位到元素上)。
- 有元素才会分配结点空间，不会有闲置的结点。

**缺点：**

- 比顺序存储结构的存储密度小 (每个节点都由数据域和指针域组成，所以相同空间内假设全存满的话顺序比链式存储更多)。
- 查找结点时链式存储要比顺序存储慢（每个节点地址不连续、无规律，导致按照索引查询效率低下。

### Set

#### HashSet

- HashSet特点
  - 非线程安全。
  - 允许null值。
  - 添加值得时候会先获取对象的hashCode方法，如果hashCode 方法返回的值一致，则再调用equals方法判断是否一致，如果不一致才add元素。
- HashSet底层是HashMap，添加一个元素时，先得到hash值-会转换成--->索引值。
- 找到存储数据表table，看这个索引位置是否已经存放元素，如果没有直接加入，如果相同，就放弃添加，如果不相同添加到最后。
- 在JAVA8，如果有一条链表的元素个数到达 TREEIFY——THRESHOLD（默认是8），并且table的大小>=MIN_TREEIFY_CAPACITY（默认64），就会转换成红黑数。

#### LinkedHashSet

- LinkedHashSet底层是一个LinkedHashMap，底层维护了一个数组+双向链表
- LinkedHashSet根据元素的HashCode值来决定元素的存储位置，同时使用链表维护元素的次序，这使得元素看起来是以插入顺序保存的。
- LinkedHashSet不允许添加重复元素

#### TreeSet

- TreeSet是基于TreeMap的NavigableSet实现。
- TreeSet的元素存储在TreeMap中的key中，TreeMap的value是一个常量对象。
- 非线程安全 。
- java8新增分割器spliterator() 方法。

## map

#### HashMap

- jdk7的hashmap底层是【数组+链表】，jdk8的底层是【数组+链表+红黑树】

#### Hashtable

- 存放的元素时键值对：即k-v
- hashtable的键和值都不能为null，否则会抛出NullPointerException
- hashTable使用方法基本上和HashMap一样
- hashTable是线程安全的(synchronized)，而hashmap是线程不安全的
- 底层是一个数组，初始化大小为11（0-10）
- 和hashmap的去呗
  - hashmap不是线程安全的，而hashtable是线程安全的
  - hashtable的键和值都不允许为空，而在HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，即可以表示 HashMap中没有该键，也可以表示该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。
  - HashTable直接使用对象的hashCode。而HashMap重新计算hash值。
  - HashTable中的hash数组初始大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。

#### Properties

- Properties类继承自Hashtable类并且实现了Map接口，也是使用一种键值对的形式来保存数据。
- 他的使用特点和Hashtable类似
- Properties还可以用于从xxx.properties文件中，加载数据到Properties类对象，并进行读取和修改
- 是线程安全的
-  该类没有泛型，因为它的元素类型是固定的：key和value都是只能是String类型，尽管你可以使用put方法存储其它类型的key-value，但是不建议这么做。
-  最重要的是，该集合能够和IO流进行结合，可将数据保存在流中或从流中加载，也能够和外部的.properties配置文件或者是.xml文件进行交互，读取文件中的配置信息，使得用户可以将一些配置信息从代码中移除，实现了代码和配置信息的解耦，避免硬编码。

#### TreeMap

- TreeMap 的实现使用了红黑树数据结构，也就是一棵自平衡的排序二叉树，这样就可以保证快速检索指定节点。对于 TreeMap 而言，它采用一种被称为“红黑树”的排序二叉树来保存 Map 中每个 Entry —— 每个 Entry 都被当成“红黑树”的一个节点对待。举例：

  ```java
  
  public class TreeMapTest  {   
      public static void main(String[] args) {   
          TreeMap<String , Double> map =  new TreeMap<String , Double>();   
          map.put("ccc" , 89.0);   
          map.put("aaa" , 80.0);   
          map.put("zzz" , 80.0);   
          map.put("bbb" , 89.0);   
          System.out.println(map);   
      }   
  }
  
      当程序执行 map.put("ccc" , 89.0); 时，系统将直接把 "ccc"-89.0 这个 Entry 放入 Map 中，这个 Entry 就是该“红黑树”的根节点。接着程序执行 map.put("aaa" , 80.0); 时，程序会将 "aaa"-80.0 作为新节点添加到已有的红黑树中。
      
      以后每向 TreeMap 中放入一个 key-value 对，系统都需要将该 Entry 当成一个新节点，添加成已有红黑树中，通过这种方式就可保证 TreeMap 中所有 key 总是由小到大地排列。例如我们输出上面程序，将看到如下结果（所有 key 由小到大地排列）:
   {aaa=80.0, bbb=89.0, ccc=89.0, zzz=80.0}
  ```

- 对于 TreeMap 而言，由于它底层采用一棵“红黑树”来保存集合中的 Entry，这意味这 TreeMap 添加元素、取出元素的性能都比 HashMap 低（红黑树和Hash数据结构上的区别）：当 TreeMap 添加元素时，需要通过循环找到新增 Entry 的插入位置，因此比较耗性能；当从 TreeMap 中取出元素时，需要通过循环才能找到合适的 Entry，也比较耗性能。但 TreeMap、TreeSet 比 HashMap、HashSet 的优势在于：TreeMap 中的所有 Entry 总是按 key 根据指定排序规则保持有序状态，TreeSet 中所有元素总是根据指定排序规则保持有序状态。

