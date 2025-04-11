
> 因为路径的空格导致的请求路径不匹配问题


```java
@RestController  
@Tag(name = "品牌管理")  
@SuppressWarnings({"unchecked", "rawtypes"})  
// 修正后代码  
@RequestMapping(" /api/product/brand/")  
public class BrandController {  
  
    @Autowired  
    private BrandService brandService;  
  
    @Operation(summary = "获取全部品牌")  
    @GetMapping("findAll")  
    public Result<List<Brand>> findAll(){  
        return Result.build(brandService.list(), ResultCodeEnum.SUCCESS);  
    }  
  
}
```

在这个代码，RequestMapping 中多加了一个空格，导致不匹配路径
根本原因是
- **Spring路径解析规则**：空格会被编码为`%20`，导致实际访问路径变成`/%20api/product/brand/findAll`，从而引发路径匹配失败
然后会报错

```shell
C:\Users\34045>curl -v "http://127.0.0.1:8500/api/product/brand/findAll"
*   Trying 127.0.0.1:8500...
* Connected to 127.0.0.1 (127.0.0.1) port 8500
* using HTTP/1.x
> GET /api/product/brand/findAll HTTP/1.1
> Host: 127.0.0.1:8500
> User-Agent: curl/8.10.1
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 400 Bad Request
< transfer-encoding: chunked
< Vary: Origin
< Vary: Access-Control-Request-Method
< Vary: Access-Control-Request-Headers
< Content-Type: application/json
< Date: Tue, 08 Apr 2025 03:20:59 GMT
<
{"timestamp":"2025-04-08T03:20:59.807+00:00","status":400,"error":"Bad Request","path":"/api/product/brand/findAll"}* Connection #0 to host 127.0.0.1 left intact

C:\Users\34045>
```

所以把这个空格去除即可解决 bug 

```java
2025-04-08 11:16:26 [WARN ] org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver Resolved [org.springframework.web.method.annotation.MethodArgumentTypeMismatchException: Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'; For input string: "brand"]
```

