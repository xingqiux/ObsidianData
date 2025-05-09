
![[源码 - HashMap 源码分析-{{number}}.png]]
### 常见属性
```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
static final float DEFAULT_LOAD_FACTOR = 0.75f;
transient HashMap.Node<K,V>[] table;
transient int size;
```

- `DEFAULT_INITIAL_CAPACITY`：默认的初始容量，值为 16。
- `DEFAULT_LOAD_FACTOR`：默认的加载因子，值为 0.75。

### 扩容阈值
扩容阈值 = 数组容量 * 加载因子

这个意思是，如果HashMap 的数组容量乘加载因子才会扩容

### Node 类定义
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    HashMap.Node<K,V> next;

    Node(int hash, K key, V value, HashMap.Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

### 关键点解释
- `DEFAULT_INITIAL_CAPACITY`：`HashMap` 初始化时的默认容量，初始大小为 16。
- `DEFAULT_LOAD_FACTOR`：`HashMap` 的默认加载因子，当 `HashMap` 中的元素数量超过 `capacity * load factor` 时，`HashMap` 会进行扩容。
- `table`：一个 `Node` 类型的数组，用于存储 `HashMap` 中的元素。
- `size`：`HashMap` 中实际存储的键值对的数量。
- `Node` 类：`HashMap` 中存储键值对的节点类，实现了 `Map.Entry` 接口，包含 `hash`、`key`、`value` 和 `next` 属性。`next` 用于处理哈希冲突，形成链表结构。


### put的操作流程
![[源码 - HashMap 源码分析.png|700]]