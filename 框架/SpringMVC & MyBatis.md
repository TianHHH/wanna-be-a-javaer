# SpringMVC

## SpringMVC的常用注解

接收请求参数的注解: 

- `@RequestParam`: **接收非json格式的数据**(比如URL中的查询参数), 然后将请求参数绑定到方法参数上
- `@RequestHeader`: **从http请求头中获取指定字段**, 并注入到方法参数上
  - `@RequestHeader(value = "token", required = false, defaultValue = "anonymous")`
- **<font style="color:blue">`@RequestBody`: 将http请求体中的json数据反序列化为Java对象</font>**
- `@PathVariable`: **从URL路径中提取变量值**并绑定到方法参数上, 经常用于 RESTful 接口

```java
@GetMapping("/user/{id}")
public String getUser(@PathVariable("id") Long userId) {
    return "User ID: " + userId;
}
```

- `@CookieValue`: 从客户端请求携带的Cookie中获取指定的字段值, 并将其绑定到方法参数中

```java
@GetMapping("/profile")
public String profile(@CookieValue("JSESSIONID") String sessionId) {
    return "Session: " + sessionId;
}
```

web请求映射相关的注解:

- `@RequestMapping`: 基础的通用映射注解, 可以用在类或者方法上
- 还有`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`
  - 这些注解都是对 `@RequestMapping` 的封装, **限定了请求方式**. 但它们**符合 RESTful 规范**的设计方式, 使接口语义更清晰

响应参数相关注解:

- `@ResponseBody`: 将接口的返回值写入http响应体中, 并返回给前端(**<font style="color:blue">跳过默认的视图解析器流程</font>**). **如果返回的是对象, 会被自动序列化为json**
- `@RestController`: 是 `@Controller + @ResponseBody` 的组合注解
- `@ResponseStatus`: 用于**设置响应状态码**, 可以加在方法或自定义异常类上, 改变默认的 200、500 等响应状态码

```java
// HttpStatus是Spring定义的一个HTTP状态码枚举类
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {}
```

异常处理器相关注解:

- `@ExceptionHandler`: 在控制器或者全局异常处理类中, 定义某个方法专门处理特定类型的异常. **<font style="color:blue">该异常被抛出时, Spring会自动调用该方法进行处理</font>**
  - **被`@ExceptionHandler`标注的方法写在某个Controller 类中就表示只拦截当前Controller抛出的异常**; 写在全局异常处理类就是拦截所有Controller抛出的异常
- `@ControllerAdvice`: 标注全局异常处理类(**<font style="color:blue">返回的是视图</font>**)
- `@RestControllerAdvice`: 是 `@ControllerAdvice + @ResponseBody` 的组合注解, **也用来标注全局异常处理类, 但返回的是json数据**





# MyBatis

## #和$的区别, 哪个更安全

基本区别

- `#{}` 是**预编译参数**, MyBatis 会将其替换为 `?` 占位符, **<font style="color:blue">底层通过 JDBC 的`PreparedStatement`绑定参数</font>**
- `${}` 是**字符串替换**, MyBatis 会将传入的内容直接拼接到 SQL 中

安全性比较

- `#{}`**更安全**, 可以有效防止 SQL 注入. 因为`#{}`是作为值传递的, 而不是拼接成 SQL 的一部分
  - JDBC会自动对用户输入做转义处理(比如加引号 以及 转义特殊字符等)
  - 所以即使用户输入类似 `admin' OR '1'='1` 这样的恶意SQL片段, 也只会作为普通字符串参数处理, 而不会影响 SQL 结构
- 相反`${}`非常容易被SQL注入攻击利用, 因为MyBatis会将传入的值**原样拼接**到SQL中 (MyBatis也不会做转义处理, 比如自动加引号. 因此是否加引号要根据上下文手动判断)
  - 如果用户传入恶意字符串, 就直接成为 SQL 语句的一部分被执行了

所以在实际开发中

- 所有用户输入的参数, 都应该使用`#{}`传递
- 只有当**需要动态拼接表名 / 列名 / 排序字段等结构性内容**时才考虑使用 `${}`, 但前提是**配合白名单机制使用**, 绝对不能直接信任用户输入

