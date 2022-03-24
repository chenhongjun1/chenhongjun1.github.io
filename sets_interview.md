HashMap 与HashTable 的区别。
主要有以下几点区别。
HashMap 没有考虑同步，是线程不安全的；HashTable 在关键方法（put、get、contains、size 等）上使用了Synchronized 关键字，是线程安全的。
HashMap 允许Key/Value 都为null，后者Key/Value 都不允许为null。
HashMap 继承自AbstractMap 类；而HashTable 继承自Dictionary 类。
在jdk1.8 中，HashMap 的底层结构是数组+链表+红黑树，而HashTable 的底层结构是数组+链表。
HashMap 对底层数组采取的懒加载，即当执行第一次put 操作时才会创建数组；而HashTable 在初始化时就创建了数组。
HashMap 中数组的默认初始容量是16，并且必须是2 的指数倍数，扩容时newCapacity=2*oldCapacity；而HashTable 中默认的初始容量是11，并且不要求必须是2 的指数倍数，扩容时newCapacity=2*oldCapacity+1。
在hash 取模计算时，HashTable 的模数一般为素数，简单的做除取模结果会更为均匀，int index = (hash & 0x7FFFFFFF) % tab.length;
HashMap 的模数为2 的幂，直接用位运算来得到结果，效率要大大高于做除法，i = (n - 1) & hash。

HashMap 是怎么解决哈希冲突的
哈希冲突：当两个不同的输入值，根据同一散列函数计算出相同的散列值的现象，我们就把它叫做哈希碰撞。
在Java 中，保存数据有两种比较简单的数据结构：数组和链表。数组的特点是：寻址容易，插入和删除困难。链表的特点是：寻址困难，但插入和删除容易。所以我们将数组和链表结合在一起，发挥两者的优势，使用一种叫做链地址法的方式来解决哈希冲突。这样我们就可以拥有相同哈希值的对象组织成的一个链表放在hash 值对应的bucket 下，但相比
Key.hashCode()
返回的int 类型，我们HashMap 初始的容量大小DEFAULT_INITIAL_CAPACITY = 1 << 4（即2 的四次方为16）要远小于int 类型的范围，所以我们如果只是单纯的使用hashcode 取余来获取对应位置的bucket，这将会大大增加哈希碰撞的几率，并且最坏情况下还会将HashMap 变成一个单链表。所以肯定要对hashCode 做一定优化。
来看HashMap 的hash()函数。上面提到的问题，主要是因为如果使用hashCode 取余，那么相当于参与运算的只有hashCode 的低位，高位是没有起到任何作用的，所以我们的思路就是让**hashCode 取值出的高位也参与运算，进一步降低hash 碰撞的概率，使得数据分布更平均，我们把这样的操作称为扰动，在JDK 1.8 中的hash()函数相比在JDK 1.7 中的4 次位运算，5 次异或运算（9 次扰动），在1.8 中，只进行了1 次位运算和1 次异或运算（2 次扰动），更为简洁了。两次扰动已经达到了高低位同时参与运算的目的，提高了对应数组存储下标位置的随机性和均匀性。
通过上面的链地址法（使用散列表）和扰动函数，数据分布更为均匀，哈希碰撞也减少了。但是当HashMap 中存在大量的数据时，假如某个bucket 下对应的链表中有n 个元素，那么遍历时间复杂度就变成了O(n)，针对这个问题，JDK 1.8 在HashMap 中新增了红黑树的数据结构，进一步使得遍历复杂度降低至O(logn)。
简单总结一下HashMap 是如何有效解决哈希碰撞的：
使用链地址法（散列表）来链接拥有相同hash 值的元素；
使用2 次扰动（hash 函数）来降低哈希冲突的概率，使得概率分布更为均匀；
引入红黑树进一步降低遍历的时间复杂度。

HashMap中为什么数组长度要保证2 的幂次方。
只有当数组长度为2 的幂次方时，h&(length-1)才等价于h%length，可以用位运算来代替做除取模运算，实现了key 的定位，2 的幂次方也可以减少冲突次数，提高HashMap 的查询效率；
当然，HashTable 就没有采用2 的幂作为数组长度，而是采用素数。素数的话是用简单做除取模方法来获取下标index，而不是位运算，效率低了不少，但分布也很均匀。

ArrayList 和LinkedList 的区别。
主要有以下几点区别：
LinkedList 实现了List 和Deque 接口，一般称为双向链表；ArrayList 实现了List 接口，是动态数组。
LinkedList 在插入和删除数据时效率更高，ArrayList 在查找某个index的数据时效率更高。
LinkedList 比ArrayList 需要更多内存。

HashSet 是如何保证数据不可重复的。
HashSet 的底层其实就是HashMap，只不过HashSet 是实现了Set 接口，并且把数据作为Key 值，而Value 值一直使用一个相同的虚值来保存。由于ashMap 的K 值本身就不允许重复，并且在HashMap 中如果K/V 相同时，会用新的V 覆盖掉旧的V，然后返回旧的V，那么在HashSet 中执行这一句话始终会返回一个false，导致插入失败，这样就保证了数据的不可重复性。

JDK8中HashMap 为什么选用红黑树而不是AVL树
在平常我们用HashMap的时候，HashMap里面存储的key是具有良好的hash算法的key（比如String、Integer等包装类），冲突几率自然微乎其微，此时链表几乎不会转化为红黑树，但是当key为我们自定义的对象时，我们可能采用了不好的hash算法，使HashMap中key的冲突率极高，但是这时HashMap为了保证高速的查找效率，引入了红黑树来优化查询了。
因为从时间复杂度来说，链表的查询复杂度为o(n)；而红黑树的复杂度能达到o(logn); 比如若hash算法写的不好，一个桶中冲突1024个key，使用链表平均需要查询512次，但是红黑树仅仅10次，红黑树的引入保证了在大量hash冲突的情况下，HashMap还具有良好的查询性能。
红黑树相比avl树，在检索的时候效率其实差不多，都是通过平衡来二分查找。但对于插入删除等操作效率提高很多。红黑树不像avl树一样追求绝对的平衡，他允许局部很少的不完全平衡，这样对于效率影响不大，但省去了很多没有必要的调平衡操作，avl树调平衡有时候代价较大，所以效率不如红黑树。

Hashmap 什么时候进行扩容呢？
当 hashmap 中的元素个数超过数组大小 loadFactor 时，就会进行数组扩容，loadFactor 的默认值为 0.75，也就是说，默认情况下，数组大小为 16，那么当 hashmap 中元素个数超过 160.75=12 的时候，就把数组的大小扩展为 216=32，即扩大一倍，然后重新
计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知
hashmap 中元素的个数，那么预设元素的个数能够有效的提高 hashmap 的性能。
比如说，我们有 1000 个元素 new HashMap(1000), 但是理论上来讲 new HashMap(1024) 更合适，不过上面已经说过，即使是 1000，hashmap 也自动会将其设置为 1024。 但是 new HashMap(1024) 还不是更合适的，因为 0.75*1000 < 1000, 也就是说为了让 0.75 * size > 1000, 我们必须这样 new HashMap(2048) 才最合适，既考虑了 & 的问题，也避免了 resize 的问题。

HashSet 和 TreeSet 有什么区别？
HashSet 是由一个 hash 表来实现的，因此，它的元素是无序的。
TreeSet 是由一个树形的结构来实现的，它里面的元素是有序的。

LinkedHashMap 的实现原理?
LinkedHashMap 也是基于 HashMap 实现的，不同的是它定义了一个 Entry header，这个 header 不是放在 Table 里，它是额外独立出来的。
LinkedHashMap 通过继承 hashMap 中的 Entry, 并添加两个属性 Entry before,after, 和 header 结合起来组成一个双向链表，来实现按插入顺序或访问顺序排序。LinkedHashMap 定义了排序模式 accessOrder，该属性为 boolean 型变量，对于访问顺序，为 true；对于插入顺序，则为 false。一般情况下，不必指定排序模式，其迭代顺序即为默认为插入顺序。

