---
title: HashMap和ConcurrentHashMap
date: 2019年8月13日 22:00:00
categories: Java
tags: [HashMap,ConcurrentHashMap,源码]
---

## HashMap源码分析

>Map这样的`Key Value`在软件开发中是非常经典的结构，常用于在内存中存放数据。

### HashMap

众所周知HashMap底层是基于`数组+链表`组成的，不过jdk1.7和1.8中实现稍有不同，为了便于理解，以下源码分析以JDK1.7为主

<!--more-->

#### 存储结构

内部包含了一个Entry类型的数组table。

```java
transient Entry[] table;
```

Entry存储着键值对。它包含了四个字段，从next字段我们可以看出来Entry是一个链表。即数组中的每个位置被当成一个同，一个桶存放一个链表。HashMap使用**拉链法**来解决冲突，同一个链表存放哈希值和散列桶取模运算结果相同的Entry。

![image-20190813153728141](https://img.moilk.top/img/zhen/2019-08-13-073728.png)

下面是1.7中的实现。

```java
static class Entry<K,V> implements Map.Entry<K,V> {
  final K key;
  V value;
  Entry<K, V> next;
  int hash;
  
  Entry(int h, K k, V v, Entry<K,V> n){
    value = v;
    key = k;
    next = n;
    hash = h;
  }
  public final K getKey(){
    return key;
  }
  public final boolean equals(Object o){
    if (!(o instanceof Map.Entry))
      return false;
    Map.Entry e = (Map.Entry)o;
    Object k1 = getKey();
    Object k2 = e.getKey();
    if(k1 == k2 || (k1 != null && k1.equals(k2))){
      Object v1 = getValue();
      Object v2 = e.getValue();
      if(v1 == v2 || (v1 != null && v1.equals(v2)))
        return true;
    }
    return false;
  }
  public final int hashCode(){
    return Object.hashCode(getKey()) ^ Object.hashCode(getValue());
  }
  public final String toString() {
    return getKey() + "=" + getValue();
  }
}
```

#### 拉链法的工作原理

```java
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```

1. 新建一个HashMap，默认大小为16；
2. 插入<K1, V1>键值对，先计算K1的hashCode为115，使用除留余数法得到所在的桶下标115%16=3
3. 插入<K2, V2>键值对，先计算K2的hashCode为118，使用除留余数法达到所在的桶下标118%16=6；
4. 插入<K3, V3>键值对，计算K3的hashCode为118，使用除留余数法得到所在的桶下标118%16=6，插在<K2,V2>前面。

这里应该注意到链表的插入是以**头插法**进行的。例如上面的<K3,V3>不是插在<K2,V2>后面，而是插入在链表头部。

查找需要分成两步进行：

- 计算键值所在的桶
- 在链表上循序查找，时间复杂度显然与链表的长度成正比

#### HashMap核心成员变量

```java
static final int DEFAULT_INITIAL_CAPACITY = 1<<4;

static final int MAXMUM_CAPACITY = 1<<30;

static final float DEFAULT_LOAD_FACTOR = 0.75f;

static final Entry<?,?> [] EMPTY_TABLE = {};

transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

transient int size;

int threshold;

final float loadFactor;
```

1. 初始化桶大小，因为底层是数组，所以这是数组默认的大小，16(0-15);
2. 桶的最大值，2^30
3. 默认的负载因子（0.75）
4. table是真正存放数据的数组
5. size是Map的真实存放元素数量，
6. threshold，桶大小可在初始化时显式指定
7. 负载因子，可在初始化时显示指定。

给定的默认容量为16，负载因子为0.75.Map在使用过程中不断的往里面存放数据，当数量达到了`16*0.75=12`时，就需要将当前16的容量进行扩容。而扩容这个操作设计rehash、复制数据等操作，所以非常消耗性能。

**因此通常建议能提前预估HashMap的大小，尽量的减少扩容带来的性能损耗。**

知晓了基本结构，下面来看看重要的put和get操作。

### put操作

```java
public V put(K key, V value){
  //判断当前数组是否需要初始化
  if(table == EMPTY_TABLE){
    inflateTable(threshold);
  }
  //键为空时单独处理
  if(key == null)
    return putForNullKey(value);
  int hash = hash(key);
  //确定桶下标
  int  i =  indexFor(hash, table.length);
  // 先找出是否已经存在键为key的键值对，如果存在的话就更新这个键值对的值为value，并返回原来的值
  for(Entry<K, V> e = table[i]; e != null; e=e.next){
    Object k;
    if(e.hash == hash && ((k=e.key) == key || key.equals(k))){
      V oldValue = e.valu;
      e.value = value;
      e.recordAccess(this);
      return oldValue;
    }
  }
  modCount++;
  //插入新键值对
  addEntry(hash, key, value, i);
  return null;
}
```

HashMap允许插入键为null的键值对，但是因为无法调用null的hashCode()方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。HashMap使用第0个桶存放键为null的键值对。

使用链表的头插法，也就是新的键值对插在链表的头部，而不是链表的尾部。

```java
void addEntry(int hash, K key, V value, int bucketIndex){
  //判断是否需要扩容
  if((size >= threshold) && (null != table[bucketIndex])){
    //如果需要就进行两倍扩容，并将当前的key重新hash并定位
    resize(2 * table.length);
    hash = (null != key)?hash(key):0;
    bucketIndex = indexFor(hash, table.length);
  }
  createEntry(hash, key, value, buckeyIndex);
}
void createEntry(int hash, K key, V value, int bucketIndex){
  Entry<K,V> e = table[bucketIndex];
  //头插法，链表头部指向新的键值对，将当前位置的桶传入新建的桶中。
  table[bucketIndex] = new Entry<>(hash, key, value, e);
  size++;
}
```

### get操作

```java
public V get(Object key){
  if (key == null){
    return getForNullKey();
  }
  Entry<K, V> entry = getEntry(key);
  
  return null == entry ? null : entry.getValue();
}
final Entry<K,V> getEntry(Object key){
  if(size == 0){
    return null;
  }
  int hash = (key == null)? 0:hash(key);
  for (Entry<K,V> e = table[indexFor(hash, table.length)];
      e != null; e = e.next){
    Object k;
    if(e.hash == hash && ((k = e.key) == key) || (key != null) && key.equals(k))
      return e;
  }
  return null;
}
```

首先根据key计算出hashCode，然后定位到具体的桶中；

判断该位置是否为链表；

不是链表就根据key，key的hashCode是否相等来返回值；

为链表则需要遍历直到key及hashCode相等时就返回值

啥都没取到就直接返回null。

### put操作中的一些小心机

#### 计算hash值

```java
final int hash(Object k){
  int h = hashSeed;
  if (0 != h && k instaceof String){
    return sun.misc.Hashing.stringHash32((String) k);
  }
  
  h ^= k.hashcode();
  //类似于扰动函数
  h ^= (h>>>20) ^(h >>> 12);
  return h ^ (h >>> 7) ^(h >>> 4);
}

public final int hashCode(){
  return Objects.hashCode(key) ^ Objects.hashCode(value);
}
```

#### 取模

另x=1<<4,即x为2的四次方，它具有以下性质：

```java
x  : 0001 0000
x-1: 0000 1111
```

令一个数y与x-1做与运算，可以去除y位级表示的第四位以上数：

```java
y      : 1011 0010
x-1    : 0000 1111
y&(x-1): 0000 0010
```

这个性质和y对x取模效果是一样的。我们知道，位运算的代价比求模运算小得多，因此在这种计算时用位运算的话能带来更高的性能。

确定桶下标的最后一步是将key的hash值对桶个数取模：hash%capacity，如果能保证capacity为2的n次方，那么就可以将这个操作转换为位运算。

```java
static int indexFor(int h, int length){
  return h & (length-1);
}
```

#### 扩容基本原理

设HashMap的table长度为M，需要存储的键值对数量为N，如果哈希函数满足均匀性的要求，那么每条链表的长度大约为N/M，因此平均查找次数的复杂度为O(M/N).

为了让查找的成本降低，应该尽可能使得N/M尽可能小，因此需要保证M尽可能大，也就是说table要尽可能大。HashMap采用动态扩容来根据当前的N值来调整M值，使得空间效率和时间效率都能得到保证。

和扩容相关的参数有：capacity，size，threshold和load_factor。

| 参数        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| capacity    | table的容量大小，默认为16。capacity必须保证为2的n次方        |
| size        | 键值对的数量                                                 |
| threshold   | size的临界值，当size大于等于threshold就必须进行扩容操作      |
| load_factor | 装载因子，table能够使用的比例，threshold=capacity*load_factor |

当需要扩容时，令capacity为原来的两倍。扩容使用resize()实现，需要注意的是，扩容操作同样需要把oldTable的所有键值对重新插入到newTable中，因此这一步是非常费时的。

```java
void resize(int newCapacity){
  Entry[] oldTable = table;
  int oldCapacity = oldTable.length;
  if(oldCapacity == MAXIMUM_CAPACITY){
    threshold = Integer.MAX_VALUE;
    return;
  }
  Entry[] newTable = new Entry[newCapacity];
  transfer(newTable);
  table = newTable;
  threshold = (int)(newCapacity * loadFactor);
}
void transfer(Entry[] newTable){
  Entry[] src = table;
  int newCapacity = newTable.length;
  for(int j=0; j<src.length; j++){
    Entry<K,V> e = src[j];
    if(e != null){
      src[j] = null;
      do{
        Entry<K,V> next = e.next;
        int i = indexFor(e.hash, newCapacity);
        e.next = newTable[i];
        newTable[i] = e;
        e = next;
      }while(e != null);
    }
  }
}
```

#### 扩容-重新计算桶下标

在进行扩容时，需要把键值对重新放到对应的桶上。HashMap使用了一个特殊的机制。可以降低重新计算桶下标的操作。假设源数组长度为capacity为16，扩容之后new capacity为32.

```java
capacity   : 0001 0000
newCapacity: 0010 0000
```

对于一个key，

1. 如果它的哈希值如果在第五位上为零，那么取模得到的结果和之前一样。
2. 如果为1，那么得到的结果为原来的结果+16.

#### 计算数组容量

HashMap构造函数允许用户传入的容量不是2的n次方，因为它可以自动地将传入的容量转换为2的n次方。先考虑如何求一个数的掩码，对于10010000，它的掩码为11111111，可以使用如下方法得到：

```bash
mask |= mask >> 1 1101 1000
mask |= mask >> 2 1111 1110
mask |= mask >> 4 1111 1111
```

mask+1是大于原始数字的最小的2的n次方。

```bash
num    1001 0000
mask+1 1000 0000
```

一下是HashMap中计算数组容量的代码：

```java
static final int tableSizeFor(int cap){
  //为了避免cap本来就是2的n次方的情况
  int n = cap -1;
  n |= n >>> 1;
  n |= n >>> 2;
  n |= n >>> 4;
  n |= n >>> 8;
  n |= n >>> 16;
  return (n<0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n +1;
}
```

### Base 1.8

1.7有一个很明显需要优化的点，当Hash冲突严重时，在桶上形成的链表会变得越来越长，这样在查询时的效率就会越来越低；时间复杂度为O(N). 因此1.8中重点优化了这个查询效率

和1.7大体上都差不多，有几个重要的区别：

- TREEIFY_THRESHOLD = 8, 用于判断是否需要将链表转换为红黑树的阈值；
- HashEntry修改为Node。

Node的核心组成和HashEntry一样，存放的都是key value hashCode next等数据。

其put操作要比1.7复杂一些：

1. 判断当前桶是否为空， 空的就需要初始化（resize中会判断是否进行初始化）
2. 根据当前key的hashCode定位到具体的桶并判断是否为空，为空表明没有Hash冲突就直接在当前位置创建一个新桶即可。
3. 如果当前桶有值（Hash冲突），那么就要比较当前桶中的key、key的hashCode于写入的key是否相等，相等就复制给e，然后统一返回。
4. 如果当前桶为红黑树，那么久按照红黑树的方式写入数据。
5. 如果是个链表，就需要将当前的key、value封装成一个新节点写入到**当前桶的后面**（形成链表）。（1.7是头插法）
6. 接着判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树。
7. 如果在遍历过程中找到key相同时直接退出遍历
8. 如果e != null 就相当于存在相同的key，那就需要将值覆盖。
9. 最后判断是否需要进行扩容。

get方法：

1. 首先将key hash之后取得所定位的桶。
2. 如果桶为空直接返回null。
3. 否则判断桶的第一个位置（有可能是链表、红黑树）的key是否为查询的key，是就直接返回value。
4. 如果第一个不匹配，则判断它的下一个是红黑树还有链表。
5. 红黑树就按照树的查找方法返回值。
6. 不然就按照链表的方式遍历匹配返回值。

从这两个核心方法可以看出1.8对大链表做了优化，修改为红黑树之后查询效率直接提高到了`O(logn)`.

但是HashMap原有的问题也都存在，比如**在并发场景下使用容易出现死循环**。

```java
final HashMap<String, String> map = new HashMap<String, String>();
for(int i=0; i < 1000; i++){
  new Thread(new Runnable(){
    @Override
    public void run(){
      map.put(UUID.randomUUID().toString(), "");
    }
  }).start();
}
```

在HashMap扩容时会调用resize()方法，这里的并发操作容易在一个桶上形成**环形链表**；这样当获取一个key时，**计算出的index正好是环形链表的下标就会出现死循环**。

### 遍历方式

HashMap有以下几种遍历方式：

```java
Iterator<Map.Entry<String, String>> entryIterator = map.entryset().iterator();
while(entryIterator.hasNext()){
  Map.Entry<String,String> next = entryItreator.next();
  System.out.println("key="+next.getKey() + " value="+next.getValue());
}

Iterator<String> iterator = map.keySet().iterator();
while(iterator.hasNext()){
  String key = iterator.next();
  System.out.printlln("key=" + key + " value="+map.get(key));
}
```

**强烈建议使用第一种EntrySet进行遍历。**第一种可以把key value同时取出，第二种还得需要通过key去一次value，效率较低。

**总结：**无论是 1.7 还是 1.8 其实都能看出 JDK 没有对它做任何的同步操作，所以并发会出问题，甚至 1.7 中出现死循环导致系统不可用（1.8 已经修复死循环问题）。

下面就可以引入ConcurrentHashMap了。

## ConcurrentHashMap源码分析

### Base1.7

#### 1.存储结构

```java
static final class HashEntry<K,V> {
  final int hash;
  final K key;
  volatile V value;
  volatile HashEntry<K,V> next;
}
```

ConcurrentHashMap和HashMap实现上类似，核心数据如value，以及链表都是由volatile修饰，保证了获取时的可见性。最主要的差别是ConcurrentHashMap采用了分段锁(Segment)，每个分段锁维护者几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是Segment的个数）。一个线程占用所访问一个Segment时，不会影响到其它的Segment。

Segment继承自ReentrantLock。

```java
static final class Segment<K, V> extends ReentrantLock implements Serializable{
  private static final long serialVersionUID = xxxxxxx;
  static final int MAX_SCAN_RETRIES = 
    Runtime.getRuntime().availableProcessors() > 1?64:1;
  transient volatile HashEntry<K,V>[] table;
  transient int count;
  transient int modCount;
  transient int threshold;
  final float loadFactor;
}

final Segment<K,V>[] segments;
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

默认的并发级别为16，也就是说默认创建16个Segment。

![image-20190813204112100](https://img.moilk.top/img/zhen/2019-08-13-124117.png)

#### 2. 用分离锁实现多个线程间的并发写操作

在ConcurrentHashMap中，线程对映射表做读操作时，一般情况下不需要加锁就可以完成。对容器做结构性修改的操作才需要加锁。以put操作为例：

首先根据key计算出对应的hash值：

```java
public V put(K key, V value){
  int (value == null){
    throw new NullPointerException();
  }
  int hash = hash(key.hashCode());
  return segmentFor(hash).put(key, hash, value, false);
}
```

然后，根据hash值找到对应的Segment对象。

```java
//使用 key 的散列码来得到 segments 数组中对应的 Segment
final Segment<K,V> segmentFor(int hash){
  //将散列值右移SegmentShift位，并在高位填充0
  //然后把得到的值与SegmentMask做与运算
  //从而得到hash值对应的Segment数组的下标值
  //最后根据下标值返回散列码对应的Segment对象
  return segments[(hash>>> segmentShift) & segmentMask];  
}
```

最后，在这个Segment中执行具体的put操作：

```java
V put(K key, int hash, V value, boolean onlyIfAbsent){
  //加锁，这里是锁定某个Segment而非整个ConcurrentHashMap
  lock();
  try {
    int c = count;
    
    //如果超过再散列的阈值，执行再散列，table数组的长度扩充一倍
    if(c++ > threshold){
      rehash();
    }
    HashEntry<K,V>[] tab = table;
    // 把散列码值与 table 数组的长度减 1 的值相“与”
    // 得到该散列码对应的 table 数组的下标值
    int index = hash & (tab.length-1);
    //找到散列码对应的具体的那个桶
    HashEntry<K, V> first = tab[index];
    HashEntry<K,V> e = first;
    while (e != null && (e.hash != hash || !key.equals(e.key))){
      e = e.next;
    }
    V oldValue;
    if(e != null){
      //如果键值对已经存在
      oldValue = e.value;
      if(!onlyIfAbsent){
        e.value = value;
      }
    }else{
      oldValue = null;
      // 要添加新节点到链表中，所以 modCont 要加 1
      ++modCount;
      tab[index] = new HashEntry<K,V>(key, hash, first, value);
      count = c;
    }
    return oldValue;
  }finally {
    unlock(); //解锁
  }  
}
```

注意： 这里的加锁操作是真的（键的hash值对应的）某个具体的Segment，锁定的是该Segment而不是整个ConcurrentHashMap。因为插入键值对操作只是在Segment包含的某个桶中完成，不需要锁定整个ConcurrentHashMap。此时其他写线程对另外15个Segment的加锁并不会因为当前线程对这个Segment的加锁而阻塞。同时，所有读线程几乎不会因为本线程的加锁而阻塞（除非读线程刚好读到这个 Segment 中某个 `HashEntry 的 value 域的值为 null，此时需要加锁后重新读取该值`）。

对比HashTable和由同步器包装的HashMap每次只能有一个线程执行读或写操作，ConCurrentHashMap在并发访问性能上有了质的提高。在理想状态下，ConCurrentHashMap可以支持16个线程执行并发写操作（如果并发级别设置为16），及任意数量线程的读操作。

#### get方法

```java
public V get(Object key){
  Segment<K,V> s;
  HashEntry<K,V>[] tab;
  int h = hash(key);
  long u = (((h >>> segmentShift) & segmentMask << SSHIFT) + SBASE);
  if (( s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null
     && (tab.s.table) != null){
    for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
        (tab, ((long)(((tab.length-1) & h)<< TSHIFT)+TBASE); 
        e != null; e=e.next)){
      K k;
      if((k = e.key) == key || (e.hash == h && key.equals(k)))
        return e.value;
    }
  }
  return null;
}
```

get逻辑比较简单，只需要将key通过Hash之后定位到具体的Segment，再通过一次Hash定位到具体的元素上。由于HashEntry中value属性是用volatile关键词修饰的，保证了内存可见性，所以每次获取到的都是最新值。

ConCurrentHashMap的get方法**是非常高效的，因为整个过程都不需要加锁。**

### Base 1.8

1.7已经解决了并发问题，并且能支持N个Segment的并发度，但是依然存在HashMap在1.7版本中的问题，即查询遍历链表效率太低。因此1.8做了一些数据结构上的调整。

底层的数据结构：

![image-20190813213623283](https://img.moilk.top/img/zhen/2019-08-13-133625.png)

同时抛弃了原有的Segment分段锁，而采用了**CAS+synchronized**来保证并发安全性。

#### put方法

1. 根据key计算出hashCode；
2. 判断是否需要进行初始化
3. f即为当前key定位出的Node，如果为空表示当前位置可以写入数据，利用CAS尝试写入，失败则自旋保证成功；
4. 如果当前位置的hashCode == MOVED == -1，则需要进行扩容。
5. 如果都不满足，则利用synchronized锁写入数据。
6. 如果数量大于TREEIFY_THRESHOLD则要转换为红黑树。

#### get方法

1. 根据计算出来hashCode寻址，如果就在桶上那么就直接返回值。
2. 如果是红黑树那么按照树的额方式获取值。
3. 都不满足就按照链表的方式遍历获取值。

1.8在1.7的数据结构上做了大的改动，采用红黑树之后可以保证查询效率O(logn),甚至取消了ReentrantLock改为了synchronized，这样可以看出在新版的JDK中对synchronized优化是很到位的。

## 参考资料

[1] [HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你! ](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)

[2] [技术面试必备基础知识 ]([https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20%E5%AE%B9%E5%99%A8.md](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java 容器.md))

[3] [探索 ConcurrentHashMap 高并发性的实现机制](https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/)