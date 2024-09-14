---
title: MyBatis拦截器自动填充字段
tags:
  - Java
  - MyBatis
categories:
  - 编程笔记
headimg: https://pic.zeng.cyou/wide/202409042025525.webp
abbrlink: 21450
date: 2024-09-11 21:10:49
---

## 自定义注解

### CreatedTime

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface CreatedTime {

}
```

### UpdatedTime

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface UpdatedTime {

}
```

## 拦截器

```java
@Intercepts({@Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class})})
public class AutoTimestampInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement) args[0];
        SqlCommandType sqlCommandType = ms.getSqlCommandType();
        Object parameter = args[1];
        if (parameter != null) {
            if (sqlCommandType == SqlCommandType.INSERT) {
                fillCreatedTime(parameter);
                fillUpdatedTime(parameter);
            } else if (sqlCommandType == SqlCommandType.UPDATE) {
                fillUpdatedTime(parameter);
            }
        }
        return invocation.proceed();
    }

    private void fillCreatedTime(Object parameter) throws IllegalAccessException {
        Field[] fields = parameter.getClass().getDeclaredFields();
        for (Field field : fields) {
            if (field.isAnnotationPresent(CreatedTime.class)) {
                field.setAccessible(true);
                if (field.get(parameter) == null) {
                    field.set(parameter, new Date());
                }
            }
        }
    }

    private void fillUpdatedTime(Object parameter) throws IllegalAccessException {
        Field[] fields = parameter.getClass().getDeclaredFields();
        for (Field field : fields) {
            if (field.isAnnotationPresent(UpdatedTime.class)) {
                field.setAccessible(true);
                field.set(parameter, new Date());
            }
        }
    }

}
```

## 使用方法

```java
/**
 * 创建时间
 */
@CreatedTime
private Date createdTime;

/**
 * 更新时间
 */
@UpdatedTime
private Date updatedTime;
```

```xml
<insert id="register" parameterType="User">
    INSERT INTO user
        (id, account, password, name, avatar, created_time, updated_time)
    VALUES (#{id}, #{account}, #{password}, #{name}, #{avatar}, #{createdTime}, #{updatedTime})
</insert>
```
