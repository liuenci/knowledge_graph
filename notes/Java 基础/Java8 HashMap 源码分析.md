#### 背景
HashMap 在日常的开发中，或者是面试当中，都是非常核心的 Api，使用率极其高，下面就来分析 Java 8 的 HashMap 原理。


#### Java7 / Java8 底层机制


面试的时候面试官特别喜欢问 Java7 和 Java8 的底层机制是什么，下面来聊一聊吧。


Java7 的底层机制是数组+链表，使用这种方式的原因主要是为了解决 Hash 碰撞。记得我第一次听到 哈希碰撞这个词还是从阿里一位大佬那听到的，现在那位大佬已经在阿里 P8 了，再看看我，惭愧惭愧。


数组加链表的方式也被称作是链地址法，在每个数组元素上都存在一个链表结构，当键值对的 key 被 hash 之后，通过取模运算得到数组的下标时，将键值对放在该数组下标对应的链表上面。


![](http://ove4nglsb.bkt.clouddn.com/Java7.png#crop=0&crop=0&crop=1&crop=1&id=H6A8a&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)


图有点丑啊，我只是个后端啊，画图真不是我的强项啊。解释一下，左边一列是数组，右边的圆代表链表的节点，数组加链表这种方式是如何实现存取的呢？首先看下一下代码：


```java
map.put("cier","value")
```


代码执行到这一步的时候，首先会调用 hashCode() 方法得到 cier 的 hashCode() 值，然后通过高位运算以及取模运算得到数组的下标索引，当两个键的哈希值相同时，那就发生了哈希碰撞，发生哈希碰撞之后就会将键值对存放到链表中。如果 hash 算法的计算结果分散越均匀，则说明发生哈希碰撞的概率越低，也说明这个哈希算法越好。


如果哈希桶数组（也是我左边画的那个丑不拉几的数组）很大，那么就算是很差的哈希算法，发生哈希碰撞的几率也会比较小。


Java8 的底层机制是数组+链表+红黑树，当链表的长度小于8的时候，Java8 还是以数组加链表的机制存储键值对，当链表的长度大于8的时候，系统将会将链表转换为红黑树，红黑树本身就是一棵二叉平衡搜索树，查找、插入、删除的时间复杂度最坏为O(log n)。


```java
//红黑树
static final class TreeNode<k,v> extends LinkedHashMap.Entry<k,v> {
    TreeNode<k,v> parent;  // 父节点
    TreeNode<k,v> left; //左子树
    TreeNode<k,v> right;//右子树
    TreeNode<k,v> prev;    // needed to unlink next upon deletion
    boolean red;    //颜色属性
    TreeNode(int hash, K key, V val, Node<k,v> next) {
        super(hash, key, val, next);
    }
 
    //返回当前节点的根节点
    final TreeNode<k,v> root() {
        for (TreeNode<k,v> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
}
```


在 Java8 中同样是往 HashMap 中存放数据。


```java
map.put("cier","value");
```


这里我们可以通过一张图来理解。是不是觉得这张图特别好看？好吧，这是我在网上找到，侵权删哦。
![](http://ove4nglsb.bkt.clouddn.com/red-black-tree.png#crop=0&crop=0&crop=1&crop=1&id=tw406&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)


下面解析一下这张图的原理。


首先判断数组是否为空或者说长度为0，如果是，那么需要先扩容。


否则先根据 key 计算 hash 值，然后取模得到要插入的下标索引 i。


如果此时 table[i] != null，则需要判断 key 是否存在，如果存在就直接覆盖该 key 的值，如果不存在则判断 table[i] 是否是 treeNode，如果是树节点，则说明此时是红黑树，可以直接插入键值对。如果不是树节点，那么就需要遍历链表准备插入，如果此时链表的长度大于 8，则需要将链表转为红黑树，如果没有大于8，那么就将键值对插入到链表中，如果 key 存在，那么直接覆盖该 key 的值。


如果此时 table[i] == null，直接插入键值对。


如果此时数组的容量大于最大容量，那么就需要扩容，如果没有大于最大容量，那么整个流程结束。


#### HashMap 的四个构造函数


首先来聊聊它的构造函数吧，HashMap 的构造函数总共有四个。


```java
public HashMap();
```


构造一个具有默认初始容量 (16) 和默认加载因子 (0.75) 的空 HashMap。


```java
public HashMap(int initialCapacity);
```


构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap。


```java
public HashMap(int initialCapacity, float loadFactor);
```


构造一个带指定初始容量和加载因子的空 HashMap。


```
public HashMap(Map< ? extends K, ? extends V> m)
```


构造一个映射关系与指定 Map 相同的新 HashMap。


这里我们需要注意两个参数，分别是 initialCapacity 初始容量和 loadFactor 负载因子。


初始容量表示哈希表刚刚创建的时候的容量，默认是 16。


负载因子表示容量自动扩充之前可以达到多满的一种尺度，默认是 0.75。


当哈希表中的 Entry 数量超过了初始容量 * 负载因子，系统将会对哈希表进行重新哈希操作，从而哈希表将会变成原来两倍的大小。


下面就来讲讲 Java7 到 Java8 中关于 HashMap 做了哪些性能上的优化？


在扩容上面，咱们先来看看 Java7 的源码。


```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```


在 Java7 中，HashMap 对扩容做的比较简单，首先是将原始的哈希表的长度扩展，然后将哈希表中的键值对中的键重新进行哈希，获取哈希值，之后再对哈希值进行取模得到该键值对的存放索引，最后将键值对存放到新的哈希表中。


然后我们看下 Java8 中的扩容源码。


```java
final Node<K,V>[] resize() {
   Node<K,V>[] oldTab = table;
   int oldCap = (oldTab == null) ? 0 : oldTab.length;
   int oldThr = threshold;
   int newCap, newThr = 0;
   if (oldCap > 0) {
       if (oldCap >= MAXIMUM_CAPACITY) {
           threshold = Integer.MAX_VALUE;//如果超过最大容量，无法再扩充table
           return oldTab;
       }
       else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
           newThr = oldThr << 1; // threshold门槛扩大至2倍
   }
   else if (oldThr > 0) // initial capacity was placed in threshold
       newCap = oldThr;
   else {               // zero initial threshold signifies using defaults
       newCap = DEFAULT_INITIAL_CAPACITY;
       newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
   }
   if (newThr == 0) {
       float ft = (float)newCap * loadFactor;
       newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                 (int)ft : Integer.MAX_VALUE);
   }
   threshold = newThr;
   @SuppressWarnings({"rawtypes","unchecked"})
       Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];// 创建容量为newCap的newTab，并将oldTab中的Node迁移过来，这里需要考虑链表和tree两种情况。
   table = newTab;
   if (oldTab != null) {
       for (int j = 0; j < oldCap; ++j) {
           Node<K,V> e;
           if ((e = oldTab[j]) != null) {
               oldTab[j] = null;
               if (e.next == null)
                   newTab[e.hash & (newCap - 1)] = e;
               else if (e instanceof TreeNode)
                   ((TreeNode<K,V>)e).split(this, newTab, j, oldCap); 
                   // split方法会将树分割为lower 和upper tree两个树，
如果子树的节点数小于了UNTREEIFY_THRESHOLD阈值，则将树untreeify，将节点都存放在newTab中。
               else { // preserve order
                   Node<K,V> loHead = null, loTail = null;
                   Node<K,V> hiHead = null, hiTail = null;
                   Node<K,V> next;
                   do {
                       next = e.next;
                       if ((e.hash & oldCap) == 0) {
                           if (loTail == null)
                               loHead = e;
                           else
                               loTail.next = e;
                           loTail = e;
                       }
                       else {
                           if (hiTail == null)
                               hiHead = e;
                           else
                               hiTail.next = e;
                           hiTail = e;
                       }
                   } while ((e = next) != null);
                   if (loTail != null) {
                       loTail.next = null;
                       newTab[j] = loHead;
                   }
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


Java8 扩容的时候首先是去判断阈值是否已经超过整型的最大值，如果超过最大值就不能扩容了。否则将阈值扩展为原来的两倍，然后将新的键值对重新哈希获取哈希值，对哈希值重新取模得到哈希桶的下标索引。如果此时发生哈希碰撞的 key 没有超过 8 个，那么在数组中存储的形态还是以链表的方式存在。如果发生哈希碰撞的 key 超过 8 个，将会把链表转化红黑树。


在 Java7 中如果所有的 key 都发生哈希碰撞，那么哈希表将会退化为链表，在链表中进行插入，查找的时间复杂度都是 O(n)，而在 Java8 中，红黑树的插入和查找的时间复杂度都是 O(log n)，在基数足够大的时候，红黑树的性能比链表的性能提高了不止一星半点。
​

#### HashMap 为什么是线程不安全？
两个线程执行 put()  操作时，可能会导致数据覆盖。Java7 和 Java8 都可能存在该问题。
​

假设 A、B 两个线程同时执行 put() 操作，且两个 key 都指向同一个 bucket，那么此时两个节点，都会头插法。
```java
public V put(K key, V value) {
    ...
    addEntry(hash, key, value, i);
}


void addEntry(int hash, K key, V value, int bucketIndex) {
    ...
    createEntry(hash, key, value, bucketIndex);
}


void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}

```
当执行到最后的 createEntry() 方法，首先获取到 bucket 上的头结点，然后再将新节点作为 bucket 的头部，并指向旧的头结点，完成一次头插法的操作。
当线程 A 和线程 B 都获取到了 bucket 的头结点后，若此时线程 A 的时间片用完，线程 B 将新数据完成了头插法操作，此时轮到线程 A 操作，但是此时线程 A 拿到的头结点已经是之前的旧的头结点了，并不包含线程 B 新插入进来的新节点，线程 A 再做头插法操作，就会抹掉 B 新增的节点，导致数据丢失。
​

解决方案：换 ConcurrentHashMap
​

#### HashMap 扩容带来的死循环（Java7）
Java7 使用的头插法在多线程环境下会产生，在 Java8 中使用的是尾插法并且使用的是两队链表，在多线程环境下无非是再做一次尾插法，不会造成死循环，Java7 因为能造成死循环是因为 resize() 时使用了头插法，将原本的链表顺序做了反转，这样在多线程环境下就会留下死循环的机会。


#### 备注

- 阈值：等于哈希表的长度 * 负载因子，默认是 16*0.75 = 12
- modCount：修改次数，用于迭代遍历 HashMap 时校验是否有出现 Map 的键值对有增加或删除，如果有则在遍历时会抛出 **ConcurrentModificationException。**
- HashMap 允许 key value 都为 null 的，并且 key 为 null 只存一份，多次存储会将旧 Value 值覆盖。
- key 为 null 的存储位置，都统一放在下标为 0 的 bucket 。
- 对于发生 Hash 碰撞的 key，Java7 会使用头插法做链表的插入，Java8 会使用尾插法来做链表的插入，超过 8 个节点就会转化为红黑树。
