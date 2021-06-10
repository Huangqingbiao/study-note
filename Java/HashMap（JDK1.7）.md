## HashMap（JDK1.7

### 一、HashMap构造函数

* 实例化HashMap对象

```java
//通常我们会这样来实例化一个HashMap对象
Map hashMap = new HashMap();
//指定容量
Map hashMap = new HashMap(16);
//指定容量和加载因子
Map hashMap = new HashMap(16,0.75f);
```

而这三个构造函数背后是怎样的呢？我们依次来看源码。

在HashMap类中，有两个静态常量与HashMap对象的实例化息息相关，它们分别是默认初始容量DEFAULT_INITIAL_CAPACITY和默认加载因子DEFAULT_LOAD_FACTOR。

```java 
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable{
//默认初始化容量
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
//最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
//默认加载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
//保存键值对的数组
static final Entry<?,?>[] EMPTY_TABLE = {};
//保存键值对的数组，数组大小必须是2的幂次方2^n
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
//键值对的总数量值
transient int size;
//阈值，用于HashMap扩容的标志，threshold = capacity * loadFactor,
//即默认的话，threshold = 16 * 0.75 = 12,当HashMap中的键值对数量大于12（size>12)就会扩容
int threshold;
//加载因子
final float loadFactor;
//修改次数，记录HashMap结构改变的次数（扩容时HashMap中的元素(entry对象)下标可能改变，即结构发生变化）
transient int modCount;
//默认阈值
static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
transient int hashSeed = 0;
}
```

* 无参构造（new HashMap()）

  ```java
  public HashMap() {
          this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
      }
  ```

  可见，当调用HashMap的无参构造器时会去调用HashMap带有两个参数的构造器，并把默认初始容量（16）和默认加载因子（0.75）传进去。

* 一参构造（new HashMap(16)）

  ```java
  public HashMap(int initialCapacity) {
          this(initialCapacity, DEFAULT_LOAD_FACTOR);
      }
  ```

  当指定一个参数（即容量）来实例化HashMap对象时，也是调用了HashMap两个参数的构造器，并把我们指定的初始化容量和默认的加载因子传进去。

* 两参构造（new HashMap(16,0.75f)）

  ```java
  public HashMap(int initialCapacity, float loadFactor) {
      	// 初始容量不能为负数  
      	if (initialCapacity < 0)
              throw new IllegalArgumentException("Illegal initial capacity: " +
                                                 initialCapacity);
      	// 如果初始容量大于最大容量，容量为最大容量  
      	if (initialCapacity > MAXIMUM_CAPACITY)
              initialCapacity = MAXIMUM_CAPACITY;
      	// 负载因子必须大于 0 的数值，否则抛出异常  
      	if (loadFactor <= 0 || Float.isNaN(loadFactor))
              throw new IllegalArgumentException("Illegal load factor: " +
                                                 loadFactor);

          this.loadFactor = loadFactor;
          threshold = initialCapacity;
          init();//空方法，即什么都没做
      }
  ```

  在两参构造函数中，我们可以指定初始化容量和加载因子，但这两个变量的值有一定的限制。

  为什么把初始容量赋值给阈值（threshold）？因为HashMap中没有定义一个用来标识初始化容量的变量哈哈，其实这样做是有好处的（至少省了一个变量的空间），请看下文。

> HashMap构造总结

​	无论是调用无参构造还是一参构造，再后还是会去调用两参构造，而无参构造使用默认初始容量（16）和默认加载因子（0.75）来实例化HashMap对象，一参构造用指定初始化容量和默认加载因子来实例化HashMap对象，两参构造可以指定初始化容量和加载因子来实例化HashMap对象，而初始化容量和加载因子有一定的范围限制。

### 二、HashMap使用

​	首先先来了解下HashMap内部的数据结构，我把HashMap内部的部分定义提取出来

```java 
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable{
//保存键值对的数组
static final Entry<?,?>[] EMPTY_TABLE = {};
//保存键值对的数组，数组大小必须是2的幂次方2^n
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
//HashMap类内部有一个泛型静态内部类Entry<K,V>，用于存放键-值对
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;	//指向下一个Entry对象，多个Entry对象构成一个单向链表
        int hash; 
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
       //省去setter、getter方法
        
    }
}
```

​	由此，我们可以确定一个HashMap对象由一个数组构成，数组内存放的是Entry对象，而Entry对象中存放的才是我们put进去的键值对，而Entry对象中又有指向下一个Entry对象的引用，从而组成一个链表。

![](images/HashMap结构图.png) 

​	而我们知道，对象是存储在堆中的，也就是说，对象内部（对象头和实例数据）没有存放另一个对象的空间，而只是存放了一个对象的引用。

![](images/堆空间对象分布.png) 

​	实例化HashMap对象后，我们就可以对其使用了，接下来对HashMap中方法的执行过程进行解析。

​	在实例化HashMap后，HashMap中还没有什么元素可以让我们使用，所有我们先往里面放东西吧。

> put方法

```java
//实例化HashMap
Map hashMap = new HashMap();
//往HashMap中放东西，为键-值对
hashMap.put("name","张三");
```

------

```java
public V put(K key, V value) {
  //判断这个表是否为空，为空则进行初始化，
  if (table == EMPTY_TABLE) {
    inflateTable(threshold);
  }
  // 如果 key 为 null，调用 putForNullKey 方法进行处理  
  if (key == null)
    return putForNullKey(value);
  // 计算key的hash值
  int hash = hash(key);
  // 根据key的hash值和表长计算该Entry对象应存放的table下标 
  int i = indexFor(hash, table.length);
  // 计算到下标后，如果当前下标已有Entry对象，则遍历该Entry链表
  for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    //两个对象的hash值相等不代表两个对象相等，只能说这两个对象存储在同一内存空间中
    //判断两个对象是否存储在同一内存空间，虽然两者在table中的下标一样，但在堆中空间地址可能不同
    //如果不同的话，就不需要判断后面的了，提高了效率，大概是这样吧
    //姑且认为这里是判断要插入的Entry对象的key是否已经存在吧
    //如果key存在则覆盖该key对应的value，并返回旧vlaue
    if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
      V oldValue = e.value;
      e.value = value;
      // 将该元素的访问存入历史记录中（在LinkedHashMap才发挥作用）
      e.recordAccess(this);
      return oldValue;
    }
  }

  //上面的循环结束表示当前的key不存在与表中，需要另外增加Entry对象（头插法）
  // 标志容器被修改次数的计数器，在使用迭代器遍历时起作用
  modCount++;
  // 将 key、value 添加到 i 索引处  	// 为新值创建一个新元素，并添加到数组中
  addEntry(hash, key, value, i);
  return null;
}
```

​	在向HashMap中put对象时，需要判断HashMap对象存放Entry<K,V> 的数组初始化了没，如果没有则需要先初始化。

​	注意，之前调用HashMap的构造函数实例化HashMap对象时，并没有对该数组初始化，只是声明了该数组要初始化的大小而已。

> inflateTable(threshold)

```java
 if (table == EMPTY_TABLE) {
    inflateTable(threshold);
  }
```

------

```java
private void inflateTable(int toSize) {
        // 将数组容量大小初始化为2^n
        int capacity = roundUpToPowerOf2(toSize);

        //重新设置阀值 
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        //这里初始化table数组的大小 
        table = new Entry[capacity];
        //根据capacity初始化hashSeed
        initHashSeedAsNeeded(capacity);
    }
```

------

```java
private static int roundUpToPowerOf2(int number) {
     //如果初始化容量大于MAXIMUM_CAPACITY则容量为MAXIMUM_CAPACITY
  	//容量小于1则为1，容量在（1，MAXIMUM_CAPACITY）之间就取大于等于该初始化容量的最小的2^n值
    return number >= MAXIMUM_CAPACITY
      ? MAXIMUM_CAPACITY
      : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
```

------

```java
public static int highestOneBit(int i) {
        // HD, Figure 3-1
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        return i - (i >>> 1);
    }
/*
	假设我们使用new HashMap(17);来实例化HashMap对象，则HashMap会创建一个容量为32的数组，而不是17
	为什么呢？下面为table初始化的4个过程：
      1.threshold = initialCapacity;
      2.inflateTable(threshold);
      3.int capacity = roundUpToPowerOf2(toSize)——>Integer.highestOneBit((number - 1) << 1)
      4.table = new Entry[capacity];
    当传入17后，capacity的值为Integer.highestOneBit((17 - 1) << 1
    我们知道，在Java中，int为32bit	|= 按位或运算，有1则为1
    首先 将 17-1 = 16 转为二进制位	0000 0000 0000 0000 0000 0000 0001 0000
    每次右移后再与自己按位或运算，由此可知，最后i为：0000 0000 0000 0000 0000 0000 0001 1111	
    >>> 无符号右移 无论正负，右移后高位都是补0
    i - (i >>> 1);	 0000 0000 0000 0000 0000 0000 0001 1111
    				0000 0000 0000 0000 0000 0000 0000 1111
    			=   0000 0000 0000 0000 0000 0000 0001 0000 = 32
    
	OK，最后把32返回，并初始化
*/
```

```java
if (key == null)
    return putForNullKey(value);
```

------

```java
//这里和没啥好说的，只需要记住HashMap可以存储key为null的键就行了
private V putForNullKey(V value) {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
```

------

```java
addEntry(hash, key, value, i);
```

------

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 如果 Map 中的 key-value 对的数量超过阈值并且要添加Entry对象的数组下标已有Entry对象，则进行扩容
    if ((size >= threshold) && (null != table[bucketIndex])) {
      // 把 table 对象的长度扩充到 2 倍。  
      resize(2 * table.length);
      hash = (null != key) ? hash(key) : 0;
      //寻找指定hash值对应的存放位置 
      bucketIndex = indexFor(hash, table.length);
    }

    // 创建新元素并添加到数组中
    createEntry(hash, key, value, bucketIndex);
}
```

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
  	//如果数组大小已是最大值，则没法扩容，直接返回
    if (oldCapacity == MAXIMUM_CAPACITY) {
      threshold = Integer.MAX_VALUE;
      return;
    }
	//创建容量为原来2倍的新数组
    Entry[] newTable = new Entry[newCapacity];	 
    //重新hash 
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
  }


void transfer(Entry[] newTable, boolean rehash) {	// rehash 是否重新hash 
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {		//依次从旧数组元素下标0开始循环这个旧表
            while(null != e) {		//循环处理对应数据元素，重新hash，并放入到新表
                Entry<K,V> next = e.next;		//先取出下一个元素
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];	//让先进来的往链表下移，新近的放在链表头	
                newTable[i] = e;	//将e元素保存到新表的newTable[i]位置中
                e = next;		 //e元素链表下移一位
            }
        }
    }
```

```java
void createEntry(int hash, K key, V value, int bucketIndex) {
    // 获取指定 bucketIndex 索引处的 Entry   
    Entry<K,V> e = table[bucketIndex];
    // 将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry   
    // 头插法建立链 
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    size++;
  }
```

> 总结

​	当我们往HashMap中put键值对时，其实是在HashMap类中的静态内部类Entry<K,V> 保存我们的键值数据，而Entry对象保存在一个Entry数组对象table中，而每一个Entry对象内又有一个Entry<K,V> 对象。这样，每一个table下标就保存了一个Entry对象链表。

​	而Entry对象要存储时会根据自己的hash值和table表长来计算要存在哪个数组下标下，并判断该下标是否已有元素，如果没有则直接插入，有则需要遍历Entry对象链表，判断是否已存在key值，没有则直接插入，有则覆盖，并返回旧value

​	如果存入的键值对数量超过了阈值则进行扩容，能够扩容则扩容为旧数组长度的两倍。

​	扩容时遍历数组以及Entry链表，并对Entry对象重写哈希运算，计算出新的数组下标值

​	最后完成插入

> get方法

```java
public V get(Object key) {
    // 如果 key 是 null，调用 getForNullKey 取出对应的 value  	//如果key为null，则按照null规则处理  
    if (key == null)
      return getForNullKey();
    //key!=null;调用getEntry方法
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}
```

------

```java
 private V getForNullKey() {
     if (size == 0) {
       return null;
     }
     for (Entry<K,V> e = table[0]; e != null; e = e.next) {
       if (e.key == null)
         return e.value;
     }
     return null;
 }
```

------

```java
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
      return null;
    }

    //通过key的hash值确定table下标（null对应下标0）
    // 根据该 key 值计算它的 hash 码  
    int hash = (key == null) ? 0 : hash(key);
    // 直接取出 table 数组中指定索引处的值，
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         // 搜索该 Entry 链的下一个 Entry
         e = e.next) {
      Object k;
      // 如果该 Entry 的 key 与被搜索 key 相同  
      if (e.hash == hash &&
          ((k = e.key) == key || (key != null && key.equals(k))))		//所以不仅要判断hash还要判断key（因为不同的key可能有相同的hash值） 
        return e;
    }
    return null;
}
```

> 总结

​	get方法比较简单，如果HashMap中没有元素，则返回null，key为空则返回key为null的value

​	如果key不为null则计算key的哈希值获取数组table下标，再遍历Entry链表，找到key相同的Entry对象并返回value

> remove方法

```java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}
```

------

```java
final Entry<K,V> removeEntryForKey(Object key) {
    if (size == 0) {
      return null;
    }
    int hash = (key == null) ? 0 : hash(key);
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    while (e != null) {
      Entry<K,V> next = e.next;
      Object k;
      if (e.hash == hash &&
          ((k = e.key) == key || (key != null && key.equals(k)))) {
        modCount++;
        size--;
        if (prev == e)
          table[i] = next;
        else
          prev.next = next;
        e.recordRemoval(this);
        return e;
      }
      prev = e;
      e = next;
    }

    return e;
}
```

> 总结

​	remove(key)根据key来计算数组下标——>遍历Entry链表，修改指针。

> clear方法

```java
public void clear() {
    modCount++;
    Arrays.fill(table, null);
    size = 0;
}
```

```java
public static void fill(Object[] a, Object val) {
        for (int i = 0, len = a.length; i < len; i++)
            a[i] = val;
    }
```

> 总结

​	clear方法将table中的所有值都置为空，并把size变为0。



*接下来介绍jdk1.8中的HashMap，并与jdk1.7做对比* 

## HashMap（jdk1.8)

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    static final int MAXIMUM_CAPACITY = 1 << 30;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //当Node对象数目为8时转换为红黑树
    static final int TREEIFY_THRESHOLD = 8;
    //当Node对象数目为6时转换为链表
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;
    //保存键-值对对象的数组
    transient Node<K,V>[] table;
    transient int size;
    transient int modCount;
    int threshold;
    final float loadFactor;
    //HashMap的内部类，Node类用于保存键-值对
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
}
```

> 总结

​	jdk1.8中的HashMap中没有Entry内部类，取而代之的是Node内部类，但这两者其实功能是一样的

​	并且多了TREEIFY_THRESHOLD = 8、UNTREEIFY_THRESHOLD = 6、MIN_TREEIFY_CAPACITY = 64属性，这几个属性用于Node对象存储结构的转变（链表和红黑树）

![](images/jdk1.8结构.png)

![](images/jdk1.7结构.png)

> HashMap构造函数

```java
   public HashMap(int initialCapacity, float loadFactor) {
     if (initialCapacity < 0)
       throw new IllegalArgumentException("Illegal initial capacity: " +
                                          initialCapacity);
     if (initialCapacity > MAXIMUM_CAPACITY)
       initialCapacity = MAXIMUM_CAPACITY;
     if (loadFactor <= 0 || Float.isNaN(loadFactor))
       throw new IllegalArgumentException("Illegal load factor: " +
                                          loadFactor);
     this.loadFactor = loadFactor;
	//构造HashMap时就将容量转换为2的幂次方，相比于jdk1.7提前了
     //jdk1.7是在第一次put时判空后才将容量转换为2的幂次方倍
     this.threshold = tableSizeFor(initialCapacity);
   }
```

> 总结

​	在jdk1.8中，使用HashMap构造函数进行实例化HashMap对象时，就将容量转换为2的幂次方倍数，并且直接在tableSizeFor()方法中进行转换;而在jdk1.8中，容量转为2的幂次方倍是在进行第一次put时进行的，并且需要调用Integer类中的方法进行转换(inflateTable—>roundUpToPowerOf2—>Integer.highestOneBit)

![](images/jdk1.8.png) 

![](images/jdk1.7.png)

![](images/Integer.png)



> put()方法

```java
public V put(K key, V value) {
  return putVal(hash(key), key, value, false, true);
}
```

------

```java
//参数onlyIfAbsent用于判断是否更新，为true时表示如果插入的key已经存在，可不更新value值，默认为false
//参数evict用于判断是否使用已有表，默认为ture，为false时表示新建表插入
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
      n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
      tab[i] = newNode(hash, key, value, null);
    else {
      Node<K,V> e; K k;
      if (p.hash == hash &&
          ((k = p.key) == key || (key != null && key.equals(k))))
        e = p;
      else if (p instanceof TreeNode)
        e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
      else {
        for (int binCount = 0; ; ++binCount) {
          if ((e = p.next) == null) {
            p.next = newNode(hash, key, value, null);
            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
              treeifyBin(tab, hash);
            break;
          }
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
    ++modCount;
    if (++size > threshold)
      resize();
    afterNodeInsertion(evict);
    return null;
}
```

