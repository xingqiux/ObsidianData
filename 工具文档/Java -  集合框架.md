# 1. Collection接口通用方法

## 所有集合类共享的基本方法:

### 创建示例集合
```java
Collection<String> collection = new ArrayList<>();
```

### 添加方法

```java
collection.add("元素1");  // 添加单个元素
collection.addAll(Arrays.asList("元素2", "元素3"));  // 添加多个元素
```
#### `boolean add(E e)`

Ensures that this collection contains the specified element (optional operation). Returns true if this collection changed as a result of the call.(Returns false if this collection does not permit duplicates and already contains the specified element)

确保这个集合元素包含指定的元素(可选操作)，如果这个集合成功调用而修改则返回 true，如果这个集合允许复制或者以及存在这个特定的元素返回 false

Collections that support this operation may place limitations on what elements may be added to this collection. In particular, some collections will refuse to add null elements, and others will impose restrictions on the type of elements that may be added.Collection classes should clearly specify in their documentation any restrictions on what elements may be added.

支持此操作的集合可能会对可以添加到该集合的元素添加限制，特殊情况下，一些集合会拒绝添加 null 元素，还有一些可能对添加的元素类型进行限制。集合类应该清楚的在文档中说明对于任何元素的任何限制

If a collection **refuses** to add a **particular** element for any **reason** other than that it already contains the element, it _must_ throw an exception (**rather** than returning false).This **preserves** the **invariant** that a collection always **contains** the specified element after this call returns.

如果当集合出于任何原因拒绝添加特定元素(除了已经包含怪元素的情况)，它必须要抛出异常，而不是抛出错误，这保持了集合在调用返回后是找包含指定元素不变。

#### `boolean addAll(Collection<? extends E> c)`

Adds all of the elements in the specified collection to this collection (optional operation).The behavior of this operation is undefined if the specified collection is **modified** while the operation is in progress.(This implies that  the behavior of this call isundefined if the specified collection is this conllection ,and this  collection is nonempty)

添加指定集合中所有的元素添加到这个集合(可选操作)，如果在操作过程中修改了指定集合，这个操作的行为未定义。这意味着如果指定集合是此集合且此集合非空，此调用行为未定义

**Parameters:**
c - collection **containing** elements to be added to this collection

c - 集合，包含将被添加到此集合的元素





### 删除方法
```Java
collection.remove("元素1");  // 删除指定元素
collection.removeIf(e -> e.contains("2"));  // 条件删除(Java 8+)
collection.clear();  // 清空所有元素
```
#### `boolean remove(Object o)`

Removes a single instance of the specified element from this collection, if it is present (optional operation).More formally, removes an element e such that (o == null ? e == null : o.equals(e)), if this collection contains one or more such elements. Returns true if this collection contained the specified element (or **equivalently**, if this collection changed as a result of the call).

从这个集合移除特定元素的一个实例，如果存在这个元素(可选方法)，更规范的说是移除满足 `e` 的元素例如 `o == null ? e == null : o.equals(e) ` ，如果这个集合包含一个或更多的元素 ,如果这个集合包含特定的元素返回 true（或者相当于，如果本集合因调用而发生变化），则返回 true 。

**Parameters:**
o - element to be removed from this collection, if present

o - 从这个集合中移除元素，如果存在

#### `default boolean removeIf(Predicate<? super E> filter)`

Removes all of the elements of this collection that satisfy the given predicate.Errors or runtime exceptions thrown during iteration or by the predicate are relayed to the caller.

删除所有满足指定给定 predicate 的元素，在迭代或由 pedicate 引发的错误或运行时异常将 relayed 调用者
	 
##### **Implementation Requirements:**
实现要求

The default implementation traverses all elements of the collection using its iterator(). Each matching element is removed using Iterator.remove(). If the collection's iterator does not support removal then an UnsupportedOperationException will be thrown on the first matching element.

这个默认的是现实遍历所有的元素，使用 `iterator() ` 每个 matching 元素都通过`Iterator.remove()` 删除，如果集合的迭代器不支持移除操作，则会在第一个匹配的元素时抛出 `UnsupportedOperationException`

##### **Parameters:**
`filter` - a predicate which returns `true` for elements to be removed

`filter` - 一个 predicate 如果返回 true 就移除元素

##### **Returns:**
`true` if any elements were removed

#### `void clear()`

Removes all of the elements from this collection (optional operation). The collection will be empty after this method returns.


### 查询方法

```java
// 查询方法
boolean isEmpty = collection.isEmpty();  // 检查是否为空
int size = collection.size();  // 获取集合大小
boolean contains = collection.contains("元素1");  // 检查是否包含元素
boolean containsAll = collection.containsAll(Arrays.asList("元素1", "元素2"));  // 检查是否包含所有指定元素
```

#### `boolean isEmpty();`

Returns true if this collection contains no elements.

#### `int size()`

Returns the number of elements in this collection. If this collection contains more than Integer.MAX_VALUE elements, returns Integer.MAX_VALUE.

#### `boolean contains(Object o)`

如果特定的 o 在集合内返回

#### `boolean containsAll(Collection<?> c)`

此集合包含所有指定的集合就返回 true 

### 遍历方法
```java
// 遍历方法
for (String item : collection) {  // 增强for循环
    System.out.println(item);
}

Iterator<String> iterator = collection.iterator();  // 使用迭代器
while (iterator.hasNext()) {
    String item = iterator.next();
    System.out.println(item);
    // iterator.remove();  // 安全地删除当前元素
}

collection.forEach(System.out::println);  // Java 8 forEach方法
```
### 转换方法
```java
// 转换操作
Object[] array = collection.toArray();  // 转换为Object数组
String[] strArray = collection.toArray(new String[0]);  // 转换为指定类型数组
```


# 2. List接口实现类详细方法

## ArrayList常见方法

### 创建示例集合
```java
ArrayList<String> list = new ArrayList<>();
```

### 添加方法

```java
// 添加元素
list.add("苹果");  // 在末尾添加元素
list.add(1, "香蕉");  // 在指定位置添加元素
list.addAll(Arrays.asList("橙子", "梨子"));  // 添加集合所有元素
list.addAll(1, Arrays.asList("葡萄", "西瓜"));  // 在指定位置添加集合元素
```
#### `boolean add(E e)`

添加到 ArrayList  的列表末尾

#### `void add(int index,E element)`

在指定的位置添加元素，如果当前位置和右边位置有元素，则全部往右移动（增加它们的索引值各一个）。

#### `boolean addAll(Collection<? extends E> c)`

#### `boolean addAll(int index,Collection<? extends E> c)`

The new elements will appear in the list in the order that they are returned by the specified collection's iterator.

### 删除方法
```Java
// 删除元素
list.remove(0);  // 删除指定位置元素
list.remove("香蕉");  // 删除指定元素(第一次出现的)
list.removeAll(Arrays.asList("橙子", "梨子"));  // 删除所有指定的元素
list.retainAll(Arrays.asList("苹果", "香蕉"));  // 仅保留指定的元素(取交集)
```
#### `boolean retainAll(Collection<?> c)`
Retains only the elements in this list that are contained in the specified collection. In other words, removes from this list all of its elements that are not contained in the specified collection.


### 查询方法

```java
// 访问元素
String fruit = list.get(0);  // 获取指定位置元素
int index = list.indexOf("香蕉");  // 查找元素第一次出现的位置
int lastIndex = list.lastIndexOf("苹果");  // 查找元素最后一次出现的位置
```

#### `E get(int index);`

返回指定位置元素

#### `int indexOf(Object o)`

返回指定的元素第一次出现的下标，如果没有返回下标-1

#### `int lastIndexOf(Object o)`

返回指定的元素最后一次出现的下标，如果没有返回下标-1

### 遍历方法
```java
// 遍历方法
for (String item : collection) {  // 增强for循环
    System.out.println(item);
}

Iterator<String> iterator = collection.iterator();  // 使用迭代器
while (iterator.hasNext()) {
    String item = iterator.next();
    System.out.println(item);
    // iterator.remove();  // 安全地删除当前元素
}

collection.forEach(System.out::println);  // Java 8 forEach方法
```
### 转换方法
```java
// 转换操作
// 转换为线程安全列表
List<String> synchronizedList = Collections.synchronizedList(list);
```

### 修改方法

```java
// 修改元素
list.set(1, "芒果");  // 替换指定位置元素
```

#### `E set(int index,E element)`

Replaces the element at the specified position in this list with the specified element.

### 子列表

```java
// 子列表操作
List<String> subList = list.subList(1, 3);  // 获取子列表(包括起始索引，不包括结束索引)
```
#### `List<E> subList(int fromIndex,int toIndex)`

![[Java 集合框架.png|500]]

- **视图（View）**：  
    **子列表并不是一个全新的独立列表**，而是原列表的某个范围的**动态映射视图**。这意味着它直接引用原列表的数据存储结构，因此：
    - **非结构性修改**（如修改元素值）会直接影响原列表和所有关联的子列表。
    - **结构性修改**（如添加/删除元素）通过子列表进行时，也会反映到原列表。

- **基于原列表的存储结构**：  
    子列表通过维护对原列表的引用以及当前视图的偏移（`offset`）和长度（`size`），**避免复制数据**。例如：   
    ```java
// ArrayList中的子列表实现大致逻辑（伪代码）
class SubList {
    private final ArrayList<E> parent;
    private final int offset;
    private int size;

    public E get(int index) {
        return parent.get(offset + index);
    }
}
    ```
    
    所有操作通过**索引偏移**定位到原列表的实际位置。

##### **典型应用场景**

- **范围操作的高效替代**：  
    删除/替换/批量操作原列表的某个区间时，直接通过子列表操作。
    
    ```java
    list.subList(start, end).replaceAll(x -> x * 2); // 批量修改元素list.subList(start, end).sort(Comparator.naturalOrder());  // 排序子区间
    ```
    
- **算法复用**：  
    可直接将子列表传递给 `Collections` 工具类方法，如搜索、打乱顺序等。
    
    ```java
    int index = Collections.binarySearch(list.subList(5, 10), key);
    ```


`subList` 是一个高效的工具方法，通过**共享存储**实现动态子列表视图。正确使用时能简化代码（避免显式索引循环），但需严格避免对原列表的**非法结构性修改**以防止意外错误。


### 排序

 [[Java Comparator 排序规则函数式接口文档]] Comparator 接口的使用文档

```java
// 排序
list.sort(Comparator.naturalOrder());  // 自然排序
list.sort(Comparator.comparing(String::length));  // 按长度排序
```
根据指定的比较规则对列表元素进行**原地排序**（直接修改原列表，不返回新列表）。它支持两种排序模式：

1. **自然排序**：使用元素的自然顺序（需元素实现`Comparable`接口）。
2. **自定义排序**：使用传入的`Comparator`比较器定义排序规则。
##### **工作原理**

1. **底层实现**
    - 实际排序算法因列表实现类不同而异。例如：
        - `ArrayList` 将内部数组转换为`Object[]`，使用优化的排序算法（如`TimSort`）排序后覆盖原数据。
        - `LinkedList` 可能将元素复制到数组中排序，再重新构造链表。
    - 时间复杂度通常为 **O(n log n)**。
2. **比较器优先级**  
    当传入非 `null` 的 `Comparator` 时，优先使用其规则，**忽略元素的`Comparable`实现**。

#### **示例代码**

##### 1. 自然排序（`Comparator` 为 `null`）
```java
List<Integer> numbers = new ArrayList<>(List.of(3, 1, 4, 1, 5));
numbers.sort(null); // 使用自然顺序（升序）System.out.println(numbers); 
// 输出 [1, 1, 3, 4, 5]
```

**要求**：元素类型（如 `Integer`）必须实现 `Comparable` 接口。

---

##### 2. 自定义排序：按字符串长度排序

```java
List<String> words = new ArrayList<>(List.of("apple", "banana", "cherry"));
words.sort(Comparator.comparingInt(String::length)); // 按字符串长度升序System.out.println(words); 
// 输出 [apple, banana, cherry]（同长度则保持原顺序）
```

##### 3. 降序排序

```java
List<Integer> numbers = new ArrayList<>(List.of(3, 1, 4, 1, 5));
numbers.sort(Comparator.reverseOrder()); // 降序System.out.println(numbers); 
// 输出 [5, 4, 3, 1, 1]
```
### 替换
```java
// Java 8+特性
list.replaceAll(String::toUpperCase);  // 替换所有元素
```

#### `void replaceAll(UnaryOperator<E> operator)`

将此列表中的每个元素替换为应用操作符后得到的结果


### LinkedList特有方法(队列和双端队列操作)

```java
LinkedList<String> linkedList = new LinkedList<>();

// 添加元素(链表特性)
linkedList.addFirst("首元素");  // 添加到链表头部
linkedList.addLast("尾元素");  // 添加到链表尾部
linkedList.push("新首元素");  // 入栈操作(添加到头部)

// 获取元素(不移除)
String first = linkedList.getFirst();  // 获取第一个元素
String last = linkedList.getLast();  // 获取最后一个元素
String peek = linkedList.peek();  // 获取第一个元素(不抛异常)
String peekFirst = linkedList.peekFirst();  // 同peek()
String peekLast = linkedList.peekLast();  // 获取最后一个元素(不抛异常)

// 获取并移除元素
String removed = linkedList.removeFirst();  // 移除第一个元素
String removedLast = linkedList.removeLast();  // 移除最后一个元素
String pop = linkedList.pop();  // 出栈操作(移除头部元素)
String poll = linkedList.poll();  // 移除第一个元素(不抛异常)
String pollFirst = linkedList.pollFirst();  // 同poll()
String pollLast = linkedList.pollLast();  // 移除最后一个元素(不抛异常)

// 队列操作
linkedList.offer("排队元素1");  // 添加元素到队尾
linkedList.offerFirst("插队元素");  // 添加元素到队首
linkedList.offerLast("队尾元素");  // 添加元素到队尾
```

## 3. Set接口实现类详细方法

### HashSet常见方法

```java
HashSet<String> hashSet = new HashSet<>();

// 添加元素
hashSet.add("元素1");  // 添加单个元素
hashSet.addAll(Arrays.asList("元素2", "元素3", "元素4"));  // 添加多个元素

// 删除元素
hashSet.remove("元素1");  // 删除指定元素

// 查询操作
boolean contains = hashSet.contains("元素2");  // 检查是否包含元素
int size = hashSet.size();  // 获取集合大小
boolean isEmpty = hashSet.isEmpty();  // 检查是否为空

// 批量操作
Set<String> set2 = new HashSet<>(Arrays.asList("元素2", "元素5"));
hashSet.retainAll(set2);  // 取交集，只保留两个集合中都有的元素
hashSet.addAll(set2);  // 取并集，添加另一个集合的所有元素
hashSet.removeAll(set2);  // 差集，移除另一个集合中包含的所有元素

// 转换为线程安全的Set
Set<String> synchronizedSet = Collections.synchronizedSet(hashSet);
```

### LinkedHashSet特有方法

LinkedHashSet继承自HashSet，主要区别是维护了插入顺序，方法基本相同:

```java
LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>();

// 基本操作与HashSet相同，但会保持插入顺序
linkedHashSet.add("C");
linkedHashSet.add("A");
linkedHashSet.add("B");

// 迭代时会按插入顺序返回: C, A, B
for (String s : linkedHashSet) {
    System.out.println(s);
}

// 其他操作与HashSet相同
```

### TreeSet特有方法(有序集合)

```java
// 创建自然排序的TreeSet
TreeSet<String> treeSet = new TreeSet<>();

// 使用比较器创建TreeSet
TreeSet<Person> personSet = new TreeSet<>(Comparator.comparing(Person::getAge));

// 添加元素
treeSet.add("B");
treeSet.add("A");
treeSet.add("C");
// 迭代时元素按排序顺序: A, B, C

// 导航方法(TreeSet特有)
String first = treeSet.first();  // 获取第一个(最小)元素
String last = treeSet.last();  // 获取最后一个(最大)元素
String lower = treeSet.lower("B");  // 返回小于指定元素的最大元素
String higher = treeSet.higher("B");  // 返回大于指定元素的最小元素
String floor = treeSet.floor("B");  // 返回小于等于指定元素的最大元素
String ceiling = treeSet.ceiling("B");  // 返回大于等于指定元素的最小元素

// 获取子集
SortedSet<String> headSet = treeSet.headSet("B");  // 获取小于指定元素的所有元素
SortedSet<String> tailSet = treeSet.tailSet("B");  // 获取大于等于指定元素的所有元素
SortedSet<String> subSet = treeSet.subSet("A", "C");  // 获取范围内的元素[A,C)

// 降序视图
NavigableSet<String> descendingSet = treeSet.descendingSet();

// 弹出操作
String pollFirst = treeSet.pollFirst();  // 移除并返回第一个元素
String pollLast = treeSet.pollLast();  // 移除并返回最后一个元素
```

## 4. Queue接口实现类详细方法

### PriorityQueue常见方法

```java
// 默认最小堆
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// 最大堆
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

// 自定义比较器
PriorityQueue<Task> taskQueue = new PriorityQueue<>(Comparator.comparing(Task::getPriority));

// 添加元素
minHeap.add(30);
minHeap.add(10);
minHeap.add(20);
minHeap.offer(5);  // 同add，但不抛异常

// 查看元素(不移除)
Integer peek = minHeap.peek();  // 查看堆顶元素(不删除)，为null时返回null
Integer element = minHeap.element();  // 查看堆顶元素(不删除)，为空时抛异常

// 移除元素
Integer poll = minHeap.poll();  // 移除并返回堆顶元素，为null时返回null
Integer remove = minHeap.remove();  // 移除并返回堆顶元素，为空时抛异常
boolean removed = minHeap.remove(20);  // 移除指定元素(如果存在)

// 其他操作
minHeap.clear();  // 清空队列
boolean isEmpty = minHeap.isEmpty();  // 检查是否为空
int size = minHeap.size();  // 获取队列大小
```

### ArrayDeque常见方法(双端队列)

```java
ArrayDeque<String> deque = new ArrayDeque<>();

// 添加元素
deque.add("元素1");  // 添加到队尾
deque.addFirst("首元素");  // 添加到队首
deque.addLast("尾元素");  // 添加到队尾
deque.push("栈顶元素");  // 添加到队首(用作栈)
deque.offer("排队元素");  // 添加到队尾
deque.offerFirst("排队插队");  // 添加到队首
deque.offerLast("排队末尾");  // 添加到队尾

// 查看元素(不移除)
String first = deque.getFirst();  // 获取队首元素(空时抛异常)
String last = deque.getLast();  // 获取队尾元素(空时抛异常)
String peek = deque.peek();  // 获取队首元素(空时返回null)
String peekFirst = deque.peekFirst();  // 获取队首元素(空时返回null)
String peekLast = deque.peekLast();  // 获取队尾元素(空时返回null)

// 移除元素
String removeFirst = deque.removeFirst();  // 移除队首元素(空时抛异常)
String removeLast = deque.removeLast();  // 移除队尾元素(空时抛异常)
String poll = deque.poll();  // 移除队首元素(空时返回null)
String pollFirst = deque.pollFirst();  // 移除队首元素(空时返回null)
String pollLast = deque.pollLast();  // 移除队尾元素(空时返回null)
String pop = deque.pop();  // 移除队首元素(用作栈)(空时抛异常)

// 容量相关
deque.clear();  // 清空双端队列
boolean isEmpty = deque.isEmpty();  // 检查是否为空
int size = deque.size();  // 获取大小
```

## 5. Map接口实现类详细方法

### HashMap常见方法

```java
HashMap<String, Integer> map = new HashMap<>();

// 添加/更新元素
map.put("张三", 25);  // 添加键值对，如果键已存在则更新值
map.putIfAbsent("李四", 30);  // 仅当键不存在时添加
map.putAll(Collections.singletonMap("王五", 28));  // 添加另一个Map的所有键值对

// 获取元素
Integer age = map.get("张三");  // 获取指定键的值，不存在返回null
Integer defaultAge = map.getOrDefault("赵六", 0);  // 获取值，不存在返回默认值

// 删除元素
map.remove("张三");  // 移除指定键值对
boolean removed = map.remove("李四", 30);  // 仅当键和值都匹配时移除

// 查询操作
boolean containsKey = map.containsKey("王五");  // 检查是否包含指定键
boolean containsValue = map.containsValue(28);  // 检查是否包含指定值
int size = map.size();  // 获取键值对数量
boolean isEmpty = map.isEmpty();  // 检查是否为空

// Java 8+新增方法
map.forEach((k, v) -> System.out.println(k + ": " + v));  // 遍历所有键值对
map.replaceAll((k, v) -> v + 5);  // 替换所有值
map.compute("张三", (k, v) -> (v == null) ? 20 : v + 5);  // 计算新值
map.computeIfAbsent("赵六", k -> 35);  // 键不存在时计算新值
map.computeIfPresent("李四", (k, v) -> v + 10);  // 键存在时计算新值
map.merge("王五", 10, (oldVal, newVal) -> oldVal + newVal);  // 合并值

// 视图操作
Set<String> keySet = map.keySet();  // 获取所有键的Set视图
Collection<Integer> values = map.values();  // 获取所有值的Collection视图
Set<Map.Entry<String, Integer>> entrySet = map.entrySet();  // 获取所有键值对的Set视图

// 遍历方式
for (String key : map.keySet()) {
    System.out.println(key + ": " + map.get(key));
}
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
map.forEach((k, v) -> System.out.println(k + ": " + v));  // Java 8+方式
```

### LinkedHashMap特有方法

```java
// 创建保持插入顺序的LinkedHashMap
LinkedHashMap<String, Integer> linkedMap = new LinkedHashMap<>();

// 创建保持访问顺序的LinkedHashMap(LRU缓存)
LinkedHashMap<String, Integer> accessOrderMap = new LinkedHashMap<>(16, 0.75f, true);

// 基本操作与HashMap相同，但会保持顺序
linkedMap.put("C", 3);
linkedMap.put("A", 1);
linkedMap.put("B", 2);

// 迭代时按插入顺序: C:3, A:1, B:2
linkedMap.forEach((k, v) -> System.out.println(k + ":" + v));

// 创建固定大小的LRU缓存
LinkedHashMap<String, Integer> lruCache = new LinkedHashMap<String, Integer>(16, 0.75f, true) {
    private static final int MAX_ENTRIES = 3;
  
    @Override
    protected boolean removeEldestEntry(Map.Entry<String, Integer> eldest) {
        return size() > MAX_ENTRIES;
    }
};
```

### TreeMap特有方法(有序Map)

```java
// 创建自然排序的TreeMap
TreeMap<String, Integer> treeMap = new TreeMap<>();

// 使用比较器创建TreeMap
TreeMap<Person, String> personMap = new TreeMap<>(Comparator.comparing(Person::getAge));

// 添加元素
treeMap.put("B", 2);
treeMap.put("A", 1);
treeMap.put("C", 3);
// 迭代时元素按键排序: A:1, B:2, C:3

// 导航方法(TreeMap特有)
Map.Entry<String, Integer> firstEntry = treeMap.firstEntry();  // 第一个(键最小)键值对
Map.Entry<String, Integer> lastEntry = treeMap.lastEntry();  // 最后一个(键最大)键值对
String firstKey = treeMap.firstKey();  // 第一个键
String lastKey = treeMap.lastKey();  // 最后一个键
Map.Entry<String, Integer> lowerEntry = treeMap.lowerEntry("B");  // 键小于给定键的最大键值对
Map.Entry<String, Integer> higherEntry = treeMap.higherEntry("B");  // 键大于给定键的最小键值对
Map.Entry<String, Integer> floorEntry = treeMap.floorEntry("B");  // 键小于等于给定键的最大键值对
Map.Entry<String, Integer> ceilingEntry = treeMap.ceilingEntry("B");  // 键大于等于给定键的最小键值对

// 获取子Map
SortedMap<String, Integer> headMap = treeMap.headMap("B");  // 获取键小于指定键的部分
SortedMap<String, Integer> tailMap = treeMap.tailMap("B");  // 获取键大于等于指定键的部分
SortedMap<String, Integer> subMap = treeMap.subMap("A", "C");  // 获取键在指定范围内的部分[A,C)

// 弹出操作
Map.Entry<String, Integer> pollFirstEntry = treeMap.pollFirstEntry();  // 移除并返回第一个键值对
Map.Entry<String, Integer> pollLastEntry = treeMap.pollLastEntry();  // 移除并返回最后一个键值对
```

### ConcurrentHashMap线程安全操作

```java
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();

// 基本操作与HashMap类似
concurrentMap.put("A", 1);
concurrentMap.get("A");
concurrentMap.remove("A");

// 原子操作(避免竞态条件)
concurrentMap.putIfAbsent("B", 2);  // 不存在时放入
concurrentMap.replace("B", 2, 3);  // 值匹配时替换
concurrentMap.remove("B", 3);  // 值匹配时移除

// Java 8+并发批量操作
concurrentMap.forEach(4, (k, v) -> System.out.println(k + ":" + v));  // 并行遍历(指定并行度)
Integer sum = concurrentMap.reduceValues(2, v -> v, Integer::sum);  // 并行计算值的和
int max = concurrentMap.reduceValues(2, v -> v, Integer::max);  // 并行计算最大值
long count = concurrentMap.mappingCount();  // 获取元素数量(返回long)

// 复合操作(原子)
concurrentMap.compute("C", (k, v) -> (v == null) ? 1 : v + 1);  // 计算新值
concurrentMap.computeIfAbsent("D", k -> 1);  // 仅当不存在时计算值
concurrentMap.computeIfPresent("C", (k, v) -> v + 1);  // 仅当存在时计算值
concurrentMap.merge("C", 1, Integer::sum);  // 合并值
```

## 6. Collections工具类详细方法

```java
List<String> list = new ArrayList<>(Arrays.asList("香蕉", "苹果", "橙子"));
Set<Integer> set = new HashSet<>(Arrays.asList(5, 2, 9, 1));
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);

// 排序相关
Collections.sort(list);  // 自然排序
Collections.sort(list, Collections.reverseOrder());  // 逆序排序
Collections.sort(list, String::compareToIgnoreCase);  // 使用Comparator排序
Collections.reverse(list);  // 反转顺序
Collections.shuffle(list);  // 随机打乱
Collections.shuffle(list, new Random(42));  // 使用指定随机源打乱
Collections.rotate(list, 2);  // 旋转元素(将末尾n个元素移到开头)
Collections.swap(list, 0, 2);  // 交换两个位置的元素

// 查找和替换
int binarySearchIndex = Collections.binarySearch(list, "苹果");  // 二分查找(仅适用于有序列表)
int frequency = Collections.frequency(list, "香蕉");  // 计算元素出现次数
boolean disjoint = Collections.disjoint(list, Arrays.asList("梨子", "西瓜"));  // 检查两个集合是否不相交
Collections.fill(list, "水果");  // 用指定元素填充整个列表
Collections.replaceAll(list, "苹果", "红苹果");  // 替换所有匹配元素
String max = Collections.max(list);  // 获取最大元素
String min = Collections.min(list);  // 获取最小元素
String min2 = Collections.min(list, Comparator.comparing(String::length));  // 使用比较器获取最小元素

// 不可变集合(只读视图)
List<String> unmodifiableList = Collections.unmodifiableList(list);
Set<Integer> unmodifiableSet = Collections.unmodifiableSet(set);
Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);

// 空集合(不可变)
List<Object> emptyList = Collections.emptyList();
Set<Object> emptySet = Collections.emptySet();
Map<Object, Object> emptyMap = Collections.emptyMap();

// 单例集合(不可变，只包含一个元素)
List<String> singletonList = Collections.singletonList("单例");
Set<Integer> singletonSet = Collections.singleton(42);
Map<String, Integer> singletonMap = Collections.singletonMap("key", 123);

// 线程安全包装器
List<String> synchronizedList = Collections.synchronizedList(list);
Set<Integer> synchronizedSet = Collections.synchronizedSet(set);
Map<String, Integer> synchronizedMap = Collections.synchronizedMap(map);

// 检查型集合(运行时类型检查)
List<String> checkedList = Collections.checkedList(list, String.class);
Set<Integer> checkedSet = Collections.checkedSet(set, Integer.class);
Map<String, Integer> checkedMap = Collections.checkedMap(map, String.class, Integer.class);

// 复制集合
ArrayList<String> destination = new ArrayList<>(Arrays.asList("", "", ""));
Collections.copy(destination, list);  // 将source复制到destination(须确保destination空间足够)

// 集合包装和适配
List<Integer> indexList = Arrays.asList(2, 0, 1);
List<String> newList = new ArrayList<>(list);
Collections.copy(newList, new ArrayList<>(3));  // 复制操作
newList = Arrays.asList(new String[list.size()]);  // 固定大小的列表
Collections.copy(newList, list);

// 列表与数组之间的转换
String[] array = list.toArray(new String[0]);
List<String> listFromArray = Arrays.asList(array);  // 注意：这是固定大小的视图列表
List<String> mutableList = new ArrayList<>(Arrays.asList(array));  // 可变列表
```
