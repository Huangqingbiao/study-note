# Java集合框架
------
#### 概述

Java集合框架提供了一套性能优良、使用方便的接口和类，位于java.util包下。

集合是一个对象，可容纳其他对象的引用，**任何对象加入集合后会自动转变为Object类型，所以在取出时需要进行强制类型转换。**

> Java集合框架是一个用来代表和操作集合的统一架构，包含以下内容：	

  * 接口：代表集合的抽象数据类型，例如：Collection、List、Set、Map等，定义多个接口方便用户以不同的方式操作数据集合
  * 实现（类）：是集合的具体实现，例如：ArrayList、LinkedList、HashSet、HashMap。
  * 算法：实现对集合接口对象的一些操作，例如：搜索和排序。

> Java 集合框架主要包括两种类型的容器

 * 集合（Collection）：存储一个元素
 * 图（Map)：存储键值对映射

> 集合接口

| 接口        | 简介                                       |
| --------- | ---------------------------------------- |
|           | Collection是最基本的集合接口，java不直接提供继承自Collection接口的类，而是提供其子接口（List和Set）。Collection接口存储一组**不唯一**，**无序**的对象 |
| List      | List接口继承于Collection接口，是一个有序的集合，能够通过索引来访问List中的元素。List接口存储一组 **不唯一** 、 **有序**（插入顺序的对象 |
|           | Set接口继承于Collection接口，具有与Collection完全一样的接口，只是行为上不同，Set不保存重复的元素。Set接口存储一组 **唯一**、 **无序** 的对象 |
| SortedSet | SortedSet接口继承于Set接口，存储 **有序** 的对象        |
|           | Map接口存储一组键值对象，提供key(键)和value(值)的映射       |
|           | Map.Entry是Map的内部接口，描述Map中的一个元素（键值对）      |
|           | Enumeration接口可以通过枚举（一次获取一个）对象集合中的元素。这个接口已被迭代器取代 |

#### <u> *Set和List的区别*</u>

* Set接口存储无序、不重复的数据，List接口存储有序、可以重复的数据
* Set接口的检索效率低下，删除和插入效率高，插入和删除不会引起元素的位置（索引值）改变
* List接口和数组类似，可以动态增长。查找元素效率高，插入和删除效率低，插入和删除操作会引起元素位置的改变

>集合实现类

Java提供了一套实现了Collection接口的标准集合类：具体类和抽象类（实现部分方法）

| 实现类                   | 简介                                       |
| --------------------- | ---------------------------------------- |
| AbstractCollection    | 实现了大部分Collection的方法                      |
| AbstractList          | 继承于AbstractCollection并且实现了大部分List接口中的方法  |
| AbstractSequentiaList | 继承于AbstractList，提供对数据元素的链式访问而不是随机访问      |
| LinkedList            | LinkedList类实现了List接口，允许有null（空)元素，主要用于创建链表数据结构，没有同步方法，需自己实现访问同步（创建List时构造一个同步的List)，LinkedList查找效率低。<br /> `List list = Collection.synchronized(newLinkedList(...))` |
| ArrayList             | ArrayList实现了List接口，是一个大小可变的数组，在随机访问和遍历元素时又更好的性能，非同步（多线程情况下不建议使用）。ArrayList增长了当前长度的50%时，插入删除效率低 |
|                       | 继承于AbstractCollection并且实现了大部分Set接口的方法    |
|                       | HashSet实现了Set接口，存储 **无序** 、 **不可重复** 元素的集合，最多允许存储一个值为Null元素 |
|                       | 具有可预知迭代顺序的Set接口的哈希表和链接列表实现               |
|                       | TreeSet实现了Set接口，可以实现排序等功能                |
|                       | 实现了大部分Map接口的方法                           |
|                       | HashMap是一个散列表，存储键值对（key-value）映射，实现了Map接口，根据键的HashCode值存储数据，具有很快的访问速度，最多允许一条记录的键为Null，不支持线程同步 |
| TreeMap               | 继承于AbstractMap，是树型结构                     |
|                       | 继承于AbstractMap，使用弱密钥的哈希表                 |
| LinkedHashMap         | 继承于HashMap，使用元素的自然顺序对元素进行排序              |
|                       | 继承于AbstractMap                           |














