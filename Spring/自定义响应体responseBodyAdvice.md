保持原来接口的代码，然后通过给接口类添加注解，来实现返回内容的统一。
# 方法一：利用@RestControllerAdvice注解，保持原来接口的代码(要有@Restcontrller)，然后通过给接口类添加注解，来实现返回内容的统一。
#### 我们要创建三个类：
https://juejin.cn/post/6986800656950493214
分三步：
## 一、定义ResultData<T>类
定义了返回体长什么样：code mesg data
> 泛型<T>:T是类型参数（也叫类型变量）；由尖括号 (<>) 分隔，跟在类名之后。  
>T 只是一个符号，就像在编写类文件时声明的变量名（可以是任何名称）。稍后 T 将
在初始化期间替换为有效的类名*/

```
@Data
public class ResultData<T> {
    /** 结果状态 ,具体状态码参见ResultData.java*/
    private int status;
    private String message;
    private T data;
    private String  timestamp ;


    public ResultData (){
        this.timestamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(System.currentTimeMillis());
    }


    public static <T> ResultData<T> success(T data) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setStatus(ReturnCode.RC100.getCode());
        resultData.setMessage(ReturnCode.RC100.getMessage());
        resultData.setData(data);
        return resultData;
    }

    public static <T> ResultData<T> fail(int code, String message) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setStatus(code);
        resultData.setMessage(message);
        return resultData;
    }

}

```
## 二、定义枚举类ReturnCode
定义了不同状态码
```
public enum ReturnCode {
    /**操作成功**/
    RC100(100,"操作成功"),
    RC101(101,"空指针异常"),
    RC102(102,"请求缺少参数"),
    RC103(103,"传入参数不符合校验要求"),
    RC201(201,"字段重复啊"),
    
    /**自定义状态码**/
    private final int code;
    /**自定义描述**/
    private final String message;

    ReturnCode(int code, String message){
        this.code = code;
        this.message = message;
    }


    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

}

```
## 三、定义实现类 实现ResponseBodyAdvice
>  ResponseBodyAdvice的作用：拦截Controller方法的返回值，统一处理返回值/响应体，一般用来统一返回格式，加解密，签名等等。
1. 将正常response放进ResultData里
2. 绑定全局异常、全局数据
```
@RestControllerAdvice
/*@RestControllerAdvice注解

@RestControllerAdvice是 @RestController注解的增强，可以实现三个方面的功能：

- 全局异常处理
- 全局数据绑定
- 全局数据预处理*/
public class ResponseBodyAdviceImpl implements ResponseBodyAdvice<Object> {
    //ResponseBodyAdvice的作用：拦截Controller方法的返回值，统一处理返回值/响应体，一般用来统一返回格式，加解密，签名等等。
    @Autowired
    private ObjectMapper objectMapper;
    //判断是否要执行beforeBodyWrite方法，true为执行，false不执行
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return true;
    }

    //对response处理的执行方法
    @SneakyThrows
    //@SneakyThrows注释允许您在不使用throws声明的情况下抛出已检查的异常
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        // 这里面参数很多，一般使用如下几个：
        // body 返回的内容 request 请求 response 响应

        //这段代码一定要加，如果Controller直接返回String的话，SpringBoot是直接返回，故我们需要手动转换成json
        if(body instanceof String){
            return objectMapper.writeValueAsString(ResultData.success(body));
        }

       
    }


}
```
## 四、定义全局异常
此时没定义接口异常，此时如果接口异常，data会显示错误，但仍然报成功
```
@Slf4j
@RestControllerAdvice
@ResponseBody
//dto统一了响应体，全局异常也要统一响应体
public class GlobalDefaultExceptionHandler {
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ResultData<String> exception(Exception e) {
        log.error("全局异常信息 ex={}", e.getMessage(), e);
        return ResultData.fail(ReturnCode.RC500.getCode(),e.getMessage());
    }

}
```
## 五、统一全局异常的响应体
在响应体实现类中beforeBodyWrite方法加上
```
 //处理全局异常的格式，否则会出现嵌套的响应体，data部分显示为新的响应体
        if(body instanceof ResultData){
            return body;
        }
        return ResultData.success(body);
```
***
# 方法二：不用注解，请求里要返回ResultData<T>
同上，第第三步之后不用

