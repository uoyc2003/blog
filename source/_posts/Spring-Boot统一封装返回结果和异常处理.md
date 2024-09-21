---
title: Spring Boot统一封装返回结果和异常处理
tags:
  - Java
  - Spring Boot
categories:
  - 编程笔记
headimg: https://pic.zeng.cyou/wide/202409042025526.webp
abbrlink: 36990
date: 2024-09-21 08:41:32
---

## 枚举类

```java
@Getter
@AllArgsConstructor
public enum ResultCodeEnum {

    SUCCESS("0000", "成功"),
    UN_ERROR("0001", "未知失败"),
    ILLEGAL_PARAMETER("0002", "非法参数");

    private final Integer code;
    private final String msg;

}
```

## 封装返回结果

### 通用返回体

```java
@Data
public class Result<T> implements Serializable {

    @Serial
    private static final long serialVersionUID = 1L;

    private Integer code;
    private String msg;
    private T data;

    public static <T> Result<T> ok() {
        Result<T> result = new Result<T>();
        result.setCode(ResultCodeEnum.SUCCESS.getCode());
        result.setMsg(ResultCodeEnum.SUCCESS.getMsg());
        return result;
    }

    public static <T> Result<T> ok(T data) {
        Result<T> result = new Result<T>();
        result.setCode(ResultCodeEnum.SUCCESS.getCode());
        result.setMsg(ResultCodeEnum.SUCCESS.getMsg());
        result.setData(data);
        return result;
    }

    public static <T> Result<T> ok(ResultCodeEnum resultCode) {
        Result<T> result = new Result<T>();
        result.setCode(resultCode.getCode());
        result.setMsg(resultCode.getMsg());
        return result;
    }

    public static <T> Result<T> ok(ResultCodeEnum resultCode, String msg) {
        Result<T> result = new Result<T>();
        result.setCode(resultCode.getCode());
        result.setMsg(msg);
        return result;
    }

    public static <T> Result<T> ok(ResultCodeEnum resultCode, T data) {
        Result<T> result = new Result<T>();
        result.setCode(resultCode.getCode());
        result.setMsg(resultCode.getMsg());
        result.setData(data);
        return result;
    }

    public static <T> Result<T> fail() {
        Result<T> result = new Result<T>();
        result.setCode(ResultCodeEnum.USER_ERROR.getCode());
        result.setMsg(ResultCodeEnum.USER_ERROR.getMsg());
        return result;
    }

    public static <T> Result<T> fail(T data) {
        Result<T> result = new Result<T>();
        result.setCode(ResultCodeEnum.USER_ERROR.getCode());
        result.setMsg(ResultCodeEnum.USER_ERROR.getMsg());
        result.setData(data);
        return result;
    }

    public static <T> Result<T> fail(ResultCodeEnum resultCode) {
        Result<T> result = new Result<T>();
        result.setCode(resultCode.getCode());
        result.setMsg(resultCode.getMsg());
        return result;
    }

    public static <T> Result<T> fail(ResultCodeEnum resultCode, String msg) {
        Result<T> result = new Result<T>();
        result.setCode(resultCode.getCode());
        result.setMsg(msg);
        return result;
    }

    public static <T> Result<T> fail(ResultCodeEnum resultCode, T data) {
        Result<T> result = new Result<T>();
        result.setCode(resultCode.getCode());
        result.setMsg(resultCode.getMsg());
        result.setData(data);
        return result;
    }

}
```

### 分页返回体

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class PageResult<T> implements Serializable {

    @Serial
    private static final long serialVersionUID = 1L;

    private Long totalPages;
    private Integer pageNo;
    private Long totalRecords;
    private Integer pageSize;
    private Boolean isLast = Boolean.FALSE;
    private List<T> list;

    public static <T> PageResult<T> empty() {
        PageResult<T> r = new PageResult<>();
        r.setTotalPages(1L);
        r.setPageNo(1);
        r.setTotalRecords(0L);
        r.setPageSize(0);
        r.setIsLast(true);
        r.setList(new ArrayList<>());
        return r;
    }

    public static <T> PageResult<T> init(Long totalPages, Integer pageNo, Long totalRecords, Integer pageSize, Boolean isLast, List<T> list) {
        return new PageResult<T>(totalPages, pageNo, totalRecords, pageSize, isLast, list);
    }

    public static <T> PageResult<T> init(Long totalPages, Integer pageNo, Long totalRecords, Integer pageSize, List<T> list) {
        return new PageResult<T>(totalPages, pageNo, totalRecords, pageSize, isLastPage(pageNo, totalRecords, pageSize), list);
    }

    public static Boolean isLastPage(Integer pageNo, Long totalRecords, Integer pageSize) {
        if (pageSize == 0) {
            return false;
        }
        Long pageTotal = totalRecords / pageSize + (totalRecords % pageSize == 0 ? 0 : 1);
        return pageNo >= pageTotal;
    }

}
```

## 封装异常处理

### 自定义异常类

```java
@Getter
public class CustomException extends RuntimeException {

    private Integer code;
    private String msg;

    public CustomException() {
        super();
    }

    public CustomException(String msg) {
        super(msg);
        this.code = ResultCodeEnum.SYSTEM_ERROR.getCode();
        this.msg = msg;
    }

    public CustomException(ResultCodeEnum resultCode) {
        super(resultCode.getMsg());
        this.code = resultCode.getCode();
        this.msg = resultCode.getMsg();
    }

    public CustomException(ResultCodeEnum resultCode, Throwable cause) {
        super(resultCode.getMsg(), cause);
        this.code = resultCode.getCode();
        this.msg = resultCode.getMsg();
    }

}
```

### 全局异常处理器

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ResponseBody
    @ExceptionHandler(value = CustomException.class)
    public Result<String> CustomExceptionHandler(CustomException e) {
        LOGGER.error("CustomException:{}", e.getMsg());
        return Result.fail(ResultCodeEnum.SYSTEM_ERROR, e.getMsg());
    }

    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public Result<String> ExceptionHandler(Exception e) {
        LOGGER.error("Exception:{}", e.getMessage());
        return Result.fail(ResultCodeEnum.SYSTEM_ERROR, e.getMessage());
    }

}
```
