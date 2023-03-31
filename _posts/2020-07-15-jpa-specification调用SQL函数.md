---
layout: post
title: JPA Specification调用SQL函数
---

#### JPA Specification 调用SQL函数

通过CriteriaBuilder的function方法。

```java
public interface CriteriaBuilder {
    /**
     * Create an expression for the execution of a database
     * function.
     * @param name  function name
     * @param type  expected result type
     * @param args  function arguments
     * @return expression
     */
   <T> Expression<T> function(String name, Class<T> type,
Expression<?>... args);
}
```

一个例子，可以直接调用原生方法也可以调用注册的自定义方法。

```java
Expression<Boolean> fitRegPredicate = cb.function("regexp", Boolean.class, cb.substring(stringFieldSecondLayerSq, prefix.length()+2));
```

