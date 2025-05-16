1. EasyExcel的使用流程：
```java
a. 添加依赖：
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.1.0</version>
</dependency>

b. 创建实体类，使用@ExcelProperty注解标注Excel列
c. 创建监听器(如果需要读取Excel)
d. 调用EasyExcel的读/写方法
```

2. 在该文档中的使用方式：
- 读取Excel：
```java
ExcelListener<CategoryExcelVo> excelListener = new ExcelListener<>();
EasyExcel.read(fileName, CategoryExcelVo.class, excelListener).sheet().doRead();
```

- 写入Excel：
```java
EasyExcel.write(response.getOutputStream(), CategoryExcelVo.class)
         .sheet("分类数据")
         .doWrite(categoryExcelVoList);
```

3. 监听器的构建和作用：
```java
// 监听器在读取Excel时使用，用于处理读取到的数据
public class ExcelListener<T> extends AnalysisEventListener<T> {
    private List<T> datas = new ArrayList<>();

    // 每读取一行数据会调用一次invoke方法
    @Override
    public void invoke(T data, AnalysisContext context) {
        datas.add(data);
    }

    // Excel读取完成后的回调
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // 可以在这里处理最终的数据
    }
}
```

4. 常见的EasyExcel方法：

```java
// 读取Excel
EasyExcel.read(fileName, Class, listener).sheet().doRead();

// 写入Excel到文件
EasyExcel.write(fileName, Class)
         .sheet(sheetName)
         .doWrite(dataList);

// 写入Excel到响应流
EasyExcel.write(response.getOutputStream(), Class)
         .sheet(sheetName)
         .doWrite(dataList);

// 常用注解
@ExcelProperty(value = "列名", index = 0)  // 指定Excel列名和索引
@ExcelIgnore  // 忽略该字段，不导出
@DateTimeFormat("yyyy-MM-dd")  // 日期格式化
@NumberFormat("#.##%")  // 数字格式化
```

使用建议：
1. 读取大文件时建议使用监听器
2. 写入大量数据时考虑分批写入
3. 注意设置正确的响应头，特别是在导出文件时
4. 处理好异常情况，确保流正确关闭
