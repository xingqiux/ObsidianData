## 一、Java 集合框架（Collection Framework）
[[Java -  集合框架]]

## 二、常用工具类

### 1. Arrays（数组工具类）

```java
int[] numbers = {5, 2, 9, 1, 5};
Arrays.sort(numbers);  // 排序
int index = Arrays.binarySearch(numbers, 5);  // 二分查找
int[] copy = Arrays.copyOf(numbers, 3);  // 复制前3个元素
Arrays.fill(numbers, 0);  // 填充为0
String str = Arrays.toString(numbers);  // 转为字符串
```

### 2. Objects（对象工具类）

```java
String str = null;
Objects.isNull(str);  // true
Objects.requireNonNull(obj, "对象不能为null");  // 非空检查，否则抛异常
Objects.equals(obj1, obj2);  // 安全的equals比较
Objects.hash(obj1, obj2);  // 计算对象的哈希值
```

### 3. Math（数学工具类）

```java
Math.max(5, 10);  // 10
Math.min(5, 10);  // 5
Math.abs(-10);  // 10，绝对值
Math.sqrt(16);  // 4.0，平方根
Math.pow(2, 3);  // 8.0，2的3次方
Math.random();  // 生成0.0到1.0之间的随机数
```

### 4. Random（随机数工具类）

```java
Random random = new Random();
random.nextInt(100);  // 0-99的随机整数
random.nextDouble();  // 0.0-1.0的随机浮点数
random.nextBoolean();  // 随机布尔值
```

### 5. UUID（唯一标识符）

```java
UUID uuid = UUID.randomUUID();
String uuidString = uuid.toString();  // 生成唯一ID字符串
```

## 三、Lambda 表达式

Lambda 表达式是 Java 8 引入的函数式编程特性，允许将函数作为参数传递。

### 1. 基本语法

```java
// 语法: (参数) -> {表达式或语句}
Runnable r = () -> System.out.println("Hello Lambda!");
Consumer<String> c = (s) -> System.out.println(s);
Predicate<Integer> p = n -> n > 0;
Function<Integer, String> f = i -> String.valueOf(i * 10);
```

### 2. 方法引用

```java
// 类名::静态方法
Function<String, Integer> parser = Integer::parseInt;

// 对象::实例方法
String str = "Hello";
Predicate<String> startsWith = str::startsWith;

// 类名::实例方法
Function<String, Integer> lengthFunc = String::length;

// 构造方法引用
Supplier<ArrayList<String>> supplier = ArrayList::new;
```

### 3. 常用函数式接口
```
- **Consumer<T> ** : 消费者，接收一个参数无返回值
- **Supplier<T>**: 提供者，无参数返回一个结果
- **Function<T,R>**: 函数，接收一个参数返回一个结果
- **Predicate<T>**: 断言，接收一个参数返回布尔值
- **BiFunction<T,U,R>**: 接收两个参数返回一个结果
```

```java
List<String> names = Arrays.asList("张三", "李四", "王五");

// Consumer 示例
names.forEach(name -> System.out.println("你好，" + name));

// Predicate 示例
List<String> filtered = names.stream().filter(s -> s.length() > 1).collect(Collectors.toList());
```

## 四、Java 8 Stream API

Stream API 提供了声明式操作集合的方式。

### 1. 创建 Stream

```java
// 从集合创建
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();

// 从数组创建
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);

// 使用Stream.of
Stream<String> stream3 = Stream.of("a", "b", "c");

// 生成无限流
Stream<Integer> stream4 = Stream.iterate(0, n -> n + 1);
Stream<Double> stream5 = Stream.generate(Math::random);
```

### 2. 中间操作

中间操作返回一个新的 Stream，可以链式调用。

```java
List<String> names = Arrays.asList("张三", "李四", "王五", "赵六", "张七");

// filter: 过滤元素
Stream<String> filtered = names.stream().filter(s -> s.startsWith("张"));

// map: 映射转换
Stream<Integer> lengths = names.stream().map(String::length);

// flatMap: 扁平化处理
Stream<Character> chars = names.stream()
    .flatMap(name -> name.chars().mapToObj(c -> (char)c));

// distinct: 去重
Stream<String> distinct = names.stream().distinct();

// sorted: 排序
Stream<String> sorted = names.stream().sorted();
Stream<String> customSorted = names.stream().sorted(Comparator.comparing(String::length));

// peek: 预览（常用于调试）
Stream<String> peeked = names.stream().peek(System.out::println);

// limit: 限制数量
Stream<String> limited = names.stream().limit(3);

// skip: 跳过元素
Stream<String> skipped = names.stream().skip(2);
```

### 3. 终端操作

终端操作会遍历流并生成结果，流在终端操作后不能再使用。

```java
List<String> names = Arrays.asList("张三", "李四", "王五", "赵六", "张七");

// forEach: 遍历
names.stream().forEach(System.out::println);

// count: 计数
long count = names.stream().count();

// collect: 收集结果
List<String> collected = names.stream().collect(Collectors.toList());
Set<String> set = names.stream().collect(Collectors.toSet());
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(s -> s, String::length));
String joined = names.stream().collect(Collectors.joining(", "));

// reduce: 归约
Optional<String> reduced = names.stream().reduce((a, b) -> a + "," + b);

// anyMatch/allMatch/noneMatch: 匹配
boolean anyMatch = names.stream().anyMatch(s -> s.length() > 2);
boolean allMatch = names.stream().allMatch(s -> s.length() > 0);
boolean noneMatch = names.stream().noneMatch(s -> s.isEmpty());

// findFirst/findAny: 查找
Optional<String> first = names.stream().findFirst();
Optional<String> any = names.stream().findAny();

// min/max: 最小/最大值
Optional<String> min = names.stream().min(Comparator.comparing(String::length));
```

### 4. 并行流

```java
// 将顺序流转为并行流
names.stream().parallel().forEach(System.out::println);

// 直接创建并行流
names.parallelStream().forEach(System.out::println);
```




## 五、String 类方法

### 1. 字符串创建与基本操作

```java
// 创建字符串
String s1 = "Hello";
String s2 = new String("Hello");
char[] chars = {'H', 'e', 'l', 'l', 'o'};
String s3 = new String(chars);

// 获取长度
int length = s1.length();  // 5

// 获取字符
char c = s1.charAt(1);  // 'e'

// 获取子字符串
String sub1 = s1.substring(1);     // "ello"
String sub2 = s1.substring(1, 3);  // "el"
```

### 2. 字符串比较

```java
String s1 = "hello";
String s2 = "HELLO";

// 相等比较
boolean equals = s1.equals(s2);               // false
boolean equalsIgnoreCase = s1.equalsIgnoreCase(s2);  // true

// 比较大小
int compare = s1.compareTo(s2);               // 正值，因为'h'大于'H'
int compareIgnoreCase = s1.compareIgnoreCase(s2);    // 0
```

### 3. 查找与匹配

```java
String str = "Hello, world!";

// 检查包含
boolean contains = str.contains("world");     // true

// 检查开头和结尾
boolean startsWith = str.startsWith("Hello"); // true
boolean endsWith = str.endsWith("!");         // true

// 查找位置
int indexOf = str.indexOf("o");               // 4（第一次出现）
int lastIndexOf = str.lastIndexOf("o");       // 8（最后一次出现）
int indexOfFrom = str.indexOf("o", 5);        // 8（从位置5开始查找）
```

### 4. 转换与替换

```java
String s = "  Hello World  ";

// 大小写转换
String upper = s.toUpperCase();    // "  HELLO WORLD  "
String lower = s.toLowerCase();    // "  hello world  "

// 去除首尾空格
String trimmed = s.trim();         // "Hello World"

// 替换
String replaced = s.replace("l", "L");         // "  HeLLo WorLd  "
String replacedFirst = s.replaceFirst("l", "L"); // "  HeLlo World  "
String replacedAll = s.replaceAll("\\s", "-");   // "--Hello-World--"
```

### 5. 字符串拆分与合并

```java
// 拆分
String names = "张三,李四,王五";
String[] nameArray = names.split(",");  // ["张三", "李四", "王五"]

// 合并
String joined = String.join("-", nameArray);  // "张三-李四-王五"
```

### 6. 格式化与转换

```java
// 格式化
String formatted = String.format("姓名: %s, 年龄: %d", "张三", 25);

// 转换成其他类型
int number = Integer.parseInt("123");
double decimal = Double.parseDouble("123.45");
boolean bool = Boolean.parseBoolean("true");

// 转为字符数组
char[] charArray = "Hello".toCharArray();

// 字节数组
byte[] bytes = "Hello".getBytes();
byte[] utf8bytes = "你好".getBytes("UTF-8");
```

### 7. Java 11+ 新增方法

```java
// isBlank - 检查字符串是否为空或只包含空白字符
"  ".isBlank();     // true
"Hello".isBlank();  // false

// strip - 去除首尾空白字符（支持Unicode空白）
"  Hello  ".strip();      // "Hello"

// stripLeading/stripTrailing - 去除首部/尾部空白
"  Hello  ".stripLeading();  // "Hello  "
"  Hello  ".stripTrailing(); // "  Hello"

// repeat - 重复字符串
"abc".repeat(3);    // "abcabcabc"

// lines - 分割多行字符串
"line1\nline2\nline3".lines().count();  // 3
```

以上就是关于Java集合框架、工具类、Lambda表达式、Stream API和String类方法的详细介绍。在实际开发中，熟练掌握这些核心知识点能有效提高编程效率和代码质量。