+++
title = "LRUCache"
description = ""
date = 2021-10-09T14:24:56+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  "编程思想"
]
tags = [
  "算法"
]
series = ["算法"]
images = []

+++



```java
public class LRUCache<K,V>{

    private final int cap;
    private final Map<K,V> map;
    private final LinkedList<K> list;

    public LRUCache(int cap) {
        this.cap = cap;
        map = new HashMap<>(cap);
        list = new LinkedList<>();
    }

    public void put(K key, V value) {
        if (map.size() == cap){
            K first = list.removeFirst();
            map.remove(first);
        }
        list.addLast(key);
        map.put(key, value);
    }

    public V get(K key) {
        V value = map.get(key);
        if (value == null){
            return null;
        }else {
            list.remove(key);
            list.addLast(key);
            return value;
        }
    }
}
```

