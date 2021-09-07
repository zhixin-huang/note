# HashMap
HashMap在开发中用的比较多

```java
    /**
     * 默认初始容量，必须是2的幂次方。（位与运算更快）
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * 最大容量，如果带参初始化时，指定容量大于这个值，则会直接用这个值代替
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 负载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 链表转为树的临界值
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 树转化为链表的临界值
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 链表转为树的最小容量限制
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

## HashMap的容量
使用new HashMap()时，未指定initialCapacity（初始化容量），默认容量为16。如果指定了initialCapacity，最后会通过tableForSize()方法计算出比initialCapacity大的最小2的幂次方（例如initialCapacity为7，最后容量为8。因为8是比7的大的最小2的幂次方），但是如果initialCapacity值大于MAXIMUM_CAPACITY，则容量为MAXIMUM_CAPACITY。  
## HashMap的负载因子
当HashMap中的元素超过一定值时，为了避免查询的效率问题，需要将HashMap进行扩容，即reSize()一次。这个值则是由HashMap的容量和负载因子所确定，即上限是loadFactor * capacity。  
## HashMap的数据结构
![HashMap数据结构](/images/java/util/HashMap结构图.jpg)
HashMap采用数组 + 链表/红黑树结构。  
## put方法解析
我们调用map.put(key, value)方法时，真正执行的是putVal()方法
```java
        /**
         *  将指定的值与此映射中的指定键相关联。如果映射先前包含键的映射,则替换旧值。
         */
        public V put(K key, V value) {
            return putVal(hash(key), key, value, false, true);
        }
    
        /**
         * <p>
         *  实现Map.put和相关方法
         */
        final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                       boolean evict) {
            //tab存放 当前的哈希桶， p用作临时链表节点
            Node<K, V>[] tab;
            Node<K, V> p;
            int n, i;
            // 如果哈希桶数组为空,则调用resize()初始化
            if ((tab = table) == null || (n = tab.length) == 0)
                n = (tab = resize()).length;
            //tab[i = (n - 1) & hash] 数组最大下标n-1与哈希值hash进行按位与运算来获取要插入的元素的索引
            if ((p = tab[i = (n - 1) & hash]) == null)
                //不冲突的情况，直接创建一个节点对象插入数组中
                tab[i] = newNode(hash, key, value, null);
            else {
                //冲突的情况，会怎么样？
                Node<K, V> e;//要保存的新节点
                K k;
                //1、先获取key保存的位置
                if (p.hash == hash &&
                        ((k = p.key) == key || (key != null && key.equals(k))))
                    //1.1 存在相同的key,则新值替换旧值
                    e = p;
                else if (p instanceof TreeNode)
                    //1.2 红黑树的情况
                    e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
                else {
                    //1.3 链表的情况
                    for (int binCount = 0; ; ++binCount) {
                        //遍历到链表尾部，往尾结点插值（JDK1.8为尾插法）
                        if ((e = p.next) == null) {
                            p.next = newNode(hash, key, value, null);
                            //如果链表的节点大于等于8，则转换成红黑树
                            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                                treeifyBin(tab, hash);
                            break;
                        }
                        // 已经往尾结点插入数据了，跳出循环
                        if (e.hash == hash &&
                                ((k = e.key) == key || (key != null && key.equals(k))))
                            break;
                        p = e;
                    }
                }
                //2 保存value
                if (e != null) { // existing mapping for key
                    V oldValue = e.value;
                    if (!onlyIfAbsent || oldValue == null)
                        e.value = value;
                    afterNodeAccess(e);
                    return oldValue;
                }
            }
            ++modCount;
            //3 每次put元素都会判断要不要扩容。如果元素的数量大于阈值，则扩容
            if (++size > threshold)
                resize();
    
            //LinkedHashMap使用
            afterNodeInsertion(evict);
            return null;
        }
```
过程：  
1、判断当前Node数组是否进行过初始化，如果没有则进行reSize()初始化  
2、计算当前元素插入的索引值。根据元素hashCode与capacity-1进行按位与计算  
3、判断当前索引值下的数组元素是否是空的  
3.1、如果是空的，直接new一个新的node放进去  
3.2、如果不为空，则说明有冲突  
3.2.1、先看数组元素与插入元素key的hashCode是否相同，是否是同一个key，或者equals相等，则替换旧值  
3.2.1、先看数组元素与插入元素key的hashCode是否相同，是否是同一个key，或者equals相等，则替换旧值  
3.2.2、如果数组元素是红黑树，则需要向红黑树中插入元素，遍历红黑树（利用hashCode值取定位），元素与插入元素key的hashCode是否相同，且同一个key，或者equals相等，则替换旧值，否则放入插入元素  
3.2.3、其他情况则是链表，从头到尾遍历整个链表，元素与插入元素key的hashCode是否相同，且同一个key，或者equals相等，则替换旧值，否则将插入元素放入尾部（jdk1.8尾插法，防止链表死循环）。同时判断链表长度是否大于8并且容量是否大于64，如果满足则将链表转为红黑树  
4、每次put元素都会判断要不要扩容。如果元素的数量大于阈值，则扩容。（阈值计算loadFactor * capacity）

解析：  
1、为什么HashMap的容量总是2的幂次方？  
当n为2次幂时，会满足一个公式：(n - 1) & hash = hash % n，在计算插入元素索引值时，为了更快的定位下标，采用按位与计算（%有试商的过程，看资料说按位与运算比%的快十倍）。  
2、判断元素相同为什么先进行HashCode比较，再进行=？  
HashCode不同，key肯定不同。HashCode相同，key不一定相同。  
3、为什么链表达到一定条件后会变成红黑树结构？  
当链表达到一定长度后，搜索效率会变慢，因为链表的时间复杂度为O(n)，而红黑树为O(logN)  
4、HashMap为什么要扩容？  
当元素到达一定量后，hash碰撞会越来越激烈，导致效率慢慢变低，所以需要扩容来减少hash碰撞，扩容为原来的2倍。负载因子设置为0.75有一个好处那就是0.75正好是3/4，而capacity又是2的幂。所以，两个数的乘积都是整数。  
## reSize方法解析
```java
     /**
     *  初始化或将表大小加倍。如果为null,则按照字段阈值中保存的初始容量目标进行分配。
     * 否则,因为我们使用二次幂扩展,来自每个bin的元素必须保持在相同的索引,或者在新表中以两个偏移的幂移动。
     */
    final Node<K, V>[] resize() {
        Node<K, V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;//旧阈值
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                //容量不能大于最大容量
                threshold = Integer.MAX_VALUE;
                return oldTab;
                //阈值和容量都翻倍扩容
            } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        } else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            //初始化容量16和阈值16*0.75=12
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float) newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                    (int) ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        //扩容时都会创建一个新的数组，长度为原先的两倍
        @SuppressWarnings({"rawtypes", "unchecked"})
        Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            //扩容时，元素会怎么样？
            for (int j = 0; j < oldCap; ++j) {
                Node<K, V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K, V> loHead = null, loTail = null;
                        Node<K, V> hiHead = null, hiTail = null;
                        Node<K, V> next;
                        do {
                            next = e.next;
                            //判断是否需要移位
                            if ((e.hash & oldCap) == 0) {
                                //不需要移位的链表
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            } else {
                                //需要移位的链表
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
过程：  
1、容量不能大于最大容量  
2、初始化一个长度为之前2倍的数组  
3、循环原table，把原table中的每个数组中的每个元素放入新table。通过hashCode与newCap进行位于运算，定位新的索引值  
3.1、如果node.next为空，直接计算新的索引值放入对应下标中  
3.2、如果node是树结构，则遍历树结构，重新将树中的元素与旧容量做按位与操作，分成两个树结构，按位与结果为0的在原索引位置，不为0的将该结构移至当前索引值+旧容量的位置。同时判断树的元素是否少于6，少于6就将该树结构转为链表结构  
3.3、node为链表，分为需要移位的链表和不需要移位的链表，与树同样的判断下标位置  

解析：  
1、扩容时树跟链表元素重新定位是和旧元素进行按位与操作？  
当n为2次幂时，会满足一个公式：当 hash & n/2 = 0时，hash & (n-1) = hash & (n/2-1)，当 hash & n/2 不为 0时，hash & (n-1) = hash & (n/2-1) + n/2
## remove方法
```java
    /**
     * 如果存在,从此映射中删除指定键的映射。
     */
    public V remove(Object key) {
        Node<K, V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
                null : e.value;
    }

    /**
     * 实现Map.remove和相关方法
     */
    final Node<K, V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
        Node<K, V>[] tab;
        Node<K, V> p;
        int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (p = tab[index = (n - 1) & hash]) != null) {
            Node<K, V> node = null, e;
            K k;
            V v;
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K, V>) p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash &&
                                ((k = e.key) == key ||
                                        (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                    (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K, V>) node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```
过程：  
1、跟put方法类死先定位到key所在的地址，得到当前node的前置节点p  
2、node不为空则判断node属性  
2.1、node是树结构，从红黑树中删除该元素（红黑树通过左旋右旋进行自平衡）  
2.2、node与p相同，本质将table数组所在下标置空  
2.3、node与p不相同，说明node链在p后面，直接将p.next置空  
