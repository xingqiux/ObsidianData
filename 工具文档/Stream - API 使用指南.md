# Java Stream API 使用指南

## 1. 概述
Stream API是Java 8引入的新特性,提供了一种声明式的方式来处理集合数据。它可以让我们以一种更简洁和函数式的方式来操作集合,特别适合数据处理场景。

主要特点:
- 声明式处理数据
- 支持链式操作
- 可以并行处理
- 延迟执行
- 不修改源数据

## 2. 基本用法

### 2.1 创建Stream
```java
// 从集合创建
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();

// 从数组创建
String[] arr = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);

// 直接创建
Stream<String> stream = Stream.of("a", "b", "c");
```

### 2.2 常用操作

中间操作:
```java
// 过滤
.filter(s -> s.startsWith("a"))

// 转换
.map(String::toUpperCase)

// 排序
.sorted()

// 去重
.distinct()
```

终止操作:
```java
// 收集为List
.collect(Collectors.toList())

// 遍历
.forEach(System.out::println)

// 归约
.reduce((a, b) -> a + b)
```

## 3. 实际示例

### 3.1 处理字符串列表
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> filtered = names.stream()
    .filter(name -> name.length() > 4)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

### 3.2 处理对象集合
```java
class Employee {
    String name;
    double salary;
    // 构造器和getter/setter
}

List<Employee> employees = //...
double averageSalary = employees.stream()
    .filter(e -> e.getSalary() > 5000)
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);
```

## 4. 并行流
```java
// 转换为并行流
list.parallelStream()
    .filter(...)
    .map(...)
    .collect(...);
```

## 5. 注意事项

1. Stream只能使用一次
```java
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
// 下面的代码会抛出异常
stream.forEach(System.out::println); 
```

2. 延迟执行
```java
Stream<String> stream = list.stream()
    .filter(s -> {
        System.out.println("filtering: " + s);
        return true;
    });
// 直到有终止操作,filter才会执行
```

3. 避免在并行流中使用有状态操作
```java
// 不推荐
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> result = new ArrayList<>();
numbers.parallelStream().forEach(result::add); // 结果不确定
```

## 6. 常见应用场景

1. 数据过滤和转换
2. 数据分组和统计
3. 复杂对象处理
4. 大数据量并行处理

## 7. 参考文档
- [Java Stream API Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Java Streams Tutorial](https://www.baeldung.com/java-8-streams)

这个指南涵盖了Stream API的主要用法,建议先从简单的示例开始尝试,逐步掌握更复杂的操作。如有疑问,可以参考官方文档获取更详细的信息。
