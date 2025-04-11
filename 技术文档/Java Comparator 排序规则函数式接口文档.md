# **Java中的`Comparator`接口详解**

## **1. 概述**

`Comparator<T>`是Java集合框架中的核心接口，定义在`java.util`包中，用于**定义对象的自定义排序规则**。它提供了一种实现"外部比较策略"的方式，使得即使不修改类本身的代码，也能够按照不同规则对同一类型的对象进行排序。

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    // 其他默认方法和静态方法...
}
```

## **2. 核心方法**

### **2.1 基础比较方法**

#### **`compare(T o1, T o2)`**
- **功能**：比较两个对象的顺序
- **返回值**：
  - **负数**：表示`o1 < o2`（o1应排在o2前面）
  - **零**：表示`o1 = o2`（两者顺序相等）
  - **正数**：表示`o1 > o2`（o1应排在o2后面）
- **示例**：
  ```java
  Comparator<Integer> ascendingOrder = (a, b) -> a - b;
  System.out.println(ascendingOrder.compare(5, 10)); // 返回负数(-5)，表示5应排在10前面
  ```

## **3. 默认方法（Java 8+）**

### **3.1 反转顺序**

#### **`reversed()`**
- **功能**：返回当前比较器的逆序比较器
- **示例**：
  ```java
  Comparator<Integer> descendingOrder = Integer::compare;
  Comparator<Integer> ascendingOrder = descendingOrder.reversed();
  ```

### **3.2 组合比较器**

#### **`thenComparing(Comparator<? super T> other)`**
- **功能**：当主比较器认为两个对象相等时，使用第二个比较器
- **示例**：
  ```java
  // 先按姓氏，再按名字排序
  Comparator<Person> byLastName = Comparator.comparing(Person::getLastName);
  Comparator<Person> byFirstName = Comparator.comparing(Person::getFirstName);
  Comparator<Person> fullNameComparator = byLastName.thenComparing(byFirstName);
  ```

#### **`thenComparing(Function<? super T, ? extends U> keyExtractor, Comparator<? super U> keyComparator)`**
- **功能**：当主比较器判定相等时，使用指定提取函数和比较器进行二次比较
- **示例**：
  ```java
  Comparator<Person> byAge = Comparator.comparing(Person::getAge);
  // 先按年龄排序，年龄相同再按姓名长度排序
  Comparator<Person> ageAndNameLength = byAge.thenComparing(
      Person::getName, 
      Comparator.comparingInt(String::length)
  );
  ```

#### **`thenComparingInt/Long/Double(ToInt/Long/DoubleFunction<? super T> keyExtractor)`**
- **功能**：针对基本类型的优化组合比较器方法
- **示例**：
  ```java
  // 先按姓名排序，再按年龄排序
  Comparator<Person> byNameThenAge = Comparator
      .comparing(Person::getName)
      .thenComparingInt(Person::getAge);
  ```

## **4. 静态工厂方法（Java 8+）**

### **4.1 自然顺序工厂**

#### **`naturalOrder()`**
- **功能**：返回使用自然顺序的比较器（需要比较元素实现Comparable接口）
- **示例**：
  ```java
  Comparator<String> naturalOrder = Comparator.naturalOrder();
  List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
  names.sort(naturalOrder); // [Alice, Bob, Charlie]
  ```

#### **`reverseOrder()`**
- **功能**：返回自然顺序的逆序比较器
- **示例**：
  ```java
  Comparator<Integer> reverseNatural = Comparator.reverseOrder();
  List<Integer> numbers = Arrays.asList(3, 1, 2);
  numbers.sort(reverseNatural); // [3, 2, 1]
  ```

### **4.2 键提取工厂方法**

#### **`comparing(Function<? super T, ? extends U> keyExtractor)`**
- **功能**：根据提取函数返回的键的自然顺序创建比较器
- **示例**：
  ```java
  Comparator<Person> byName = Comparator.comparing(Person::getName);
  ```

#### **`comparing(Function<? super T, ? extends U> keyExtractor, Comparator<? super U> keyComparator)`**
- **功能**：根据提取函数和键比较器创建比较器
- **示例**：
  ```java
  // 按照人名长度逆序排序
  Comparator<Person> byNameLengthDesc = Comparator.comparing(
      Person::getName, 
      (s1, s2) -> s2.length() - s1.length()
  );
  ```

#### **`comparingInt/Long/Double(ToInt/Long/DoubleFunction<? super T> keyExtractor)`**
- **功能**：基本类型优化版的comparing方法，避免装箱拆箱开销
- **示例**：
  ```java
  Comparator<String> byLength = Comparator.comparingInt(String::length);
  Comparator<Person> byAge = Comparator.comparingInt(Person::getAge);
  ```

### **4.3 Null处理工厂方法**

#### **`nullsFirst(Comparator<? super T> comparator)`**
- **功能**：创建一个null友好的比较器，将null值排在非null值前面
- **示例**：
  ```java
  Comparator<String> nullSafeComparator = Comparator.nullsFirst(Comparator.naturalOrder());
  List<String> names = Arrays.asList("Charlie", null, "Alice");
  names.sort(nullSafeComparator); // [null, Alice, Charlie]
  ```

#### **`nullsLast(Comparator<? super T> comparator)`**
- **功能**：创建一个null友好的比较器，将null值排在非null值后面
- **示例**：
  ```java
  Comparator<Integer> nullLastNumbers = Comparator.nullsLast(Integer::compare);
  List<Integer> numbers = Arrays.asList(3, null, 1);
  numbers.sort(nullLastNumbers); // [1, 3, null]
  ```

## **5. 高级用法示例**

### **5.1 复杂排序链**
```java
// 按以下优先级排序：
// 1. 部门名称（升序）
// 2. 职位级别（降序）
// 3. 工作年限（降序）
// 4. 姓名（升序）
Comparator<Employee> complexOrder = Comparator
    .comparing(Employee::getDepartment)
    .thenComparing(Employee::getLevel, Comparator.reverseOrder())
    .thenComparingInt(e -> -e.getYearsOfService())  // 注意负号表示降序
    .thenComparing(Employee::getName);
```

### **5.2 使用Java 8 Lambda简化**
```java
// 旧方式
Collections.sort(people, new Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        return p1.getAge() - p2.getAge();
    }
});

// Lambda方式
people.sort((p1, p2) -> p1.getAge() - p2.getAge());

// 方法引用方式
people.sort(Comparator.comparingInt(Person::getAge));
```

### **5.3 使用Comparator处理Map**
```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 85);
scores.put("Charlie", 90);

// 按值（分数）排序
List<Map.Entry<String, Integer>> sortedByValue = new ArrayList<>(scores.entrySet());
sortedByValue.sort(Map.Entry.comparingByValue(Comparator.reverseOrder()));

// 输出：[Alice=95, Charlie=90, Bob=85]
```

## **6. 与Comparable的区别**

| 特性 | Comparable | Comparator |
|------|------------|------------|
| 定义位置 | 在类内部定义比较逻辑 | 在独立类中定义比较逻辑 |
| 接口方法 | `compareTo(T o)` | `compare(T o1, T o2)` |
| 使用场景 | 定义类的"自然顺序" | 定义多种或临时排序规则 |
| 灵活性 | 一个类只能有一种自然顺序 | 可以定义无限多种排序规则 |
| 示例 | `Integer`, `String`, `Date` | 自定义排序器 |

## **7. 常见错误与最佳实践**

### **7.1 常见错误**
- **不符合比较器约定**：违反自反性、对称性、传递性
  ```java
  // 错误：非一致性比较器，可能导致排序不稳定
  Comparator<Integer> badComparator = (a, b) -> Math.random() > 0.5 ? 1 : -1;
  ```

- **整数溢出风险**
  ```java
  // 有风险：当a-b超出int范围时会溢出
  Comparator<Integer> riskyComparator = (a, b) -> a - b;
  
  // 安全替代方案
  Comparator<Integer> safeComparator = Integer::compare;
  ```

### **7.2 最佳实践**
- **使用静态工厂方法**：优先使用`Comparator.comparing()`等方法而非手动实现
- **避免使用减法**：使用Integer.compare()等方法避免溢出风险
- **链式调用**：使用`.thenComparing()`构建多字段比较器
- **处理null**：使用`nullsFirst`/`nullsLast`包装比较器，明确处理null
- **使用优化方法**：针对基本类型使用`comparingInt`等方法避免装箱/拆箱

## **8. 性能考虑**

1. **避免重复计算**
   ```java
   // 低效：每次比较都计算两次长度
   Comparator<String> inefficient = (s1, s2) -> s1.length() - s2.length();
   
   // 高效：预先计算键值
   Comparator<String> efficient = Comparator.comparingInt(String::length);
   ```

2. **使用基本类型专用方法**
   ```java
   // 有装箱/拆箱开销
   Comparator<Person> withBoxing = Comparator.comparing(Person::getAge);
   
   // 避免装箱/拆箱
   Comparator<Person> noBoxing = Comparator.comparingInt(Person::getAge);
   ```

---

通过合理使用Comparator接口，我们可以轻松实现各种复杂的排序逻辑，使代码更加灵活和可维护。Java 8引入的默认方法和静态工厂方法更是大大简化了比较器的创建和组合。