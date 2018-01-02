# java.util.Map.java接口
[https://github.com/zxiaofan/JDK/blob/master/JDK1.8/src/java/util/Map.java](https://github.com/zxiaofan/JDK/blob/master/JDK1.8/src/java/util/Map.java)

#### Map家族  

``` 

     -HashMap -LinkedHashMap
Map -  
     -TreeMap

```

一个key 最多对应一个 value  
collection views：key集合，value集合，key-value映射集合。  
order：迭代器返回集合元素的顺序。TreeMap有特定的顺序，HashMap没有。  
可变对象作为key时要小心。  
Map对象不能把自身用作一个key，但用作value是允许的，但是the **_equals_** and _**hashCode**_ methods are no longer well defined on such a map.

```

Map<K,V>

    V get(Object key);
    V put(K key, V value);
    V remove(Object key);
    Set<Map.Entry<K, V>> entrySet();

    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }
    boolean containsKey(Object key);

===================================
//equals方法比较： 类型 size 每个元素
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<?,?> m = (Map<?,?>) o;
        if (m.size() != size())
            return false;

        try {
            Iterator<Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(m.get(key)==null && m.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(m.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused) {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }

    public int hashCode() {
        int h = 0;
        Iterator<Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext())
            h += i.next().hashCode();
        return h;
    }

```

#### HashMap 

HashMap和Hashtable都是实现Map的哈希表/散列表，  
get/put操作的时间复杂度是常量级，  
影响性能的两个因素：initial capacity初始容量 and load factor负载因子，因为到达临界时，容量翻倍，需要rehashed。  load factor默认时容量的.75。  所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。
fail-fast： HashMap不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了map（除了iterator的remove），将抛出ConcurrentModificationException

HashMap: 非同步/key允许null。Hashtable: 同步  


```

public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {

// 的确是一个数组    resize()
    transient Node<K,V>[] table;

    //hash函数，对key进行处理
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    // key-value数据结构
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

    }

    get() 和 put() 都是基于Node结构和hash(key)，get()用node.next遍历Map，比较hash(key)是否相等。

}

```

#### HashMap子类及应用

* java.util.LinkedHashMap  

```
class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{

// doubly linked list 双链表
    transient LinkedHashMap.Entry<K,V> head;
    transient LinkedHashMap.Entry<K,V> tail;

// 每个节点记录了前后兄弟节点,双向绑定
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
    }
// 插入新节点
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
// 这样的结构就可以有order属性了
}

```

* sun.net.www.http.KeepAliveCache extends HashMap<KeepAliveKey, ClientVector> 
缓存


#### TreeMap
红黑树

