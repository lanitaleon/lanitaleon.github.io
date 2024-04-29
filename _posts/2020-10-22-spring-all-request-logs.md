---
layout: post
title: Spring项目中对所有API增加日志记录
---

场景：对所有数据交互请求增加日志记录。

通过简单检索，大致有如下方式：

1.filter

顺便试了一下interceptor不行，filter可以，但是要封装一层。

2.aop+注解

切点可以设为注解，也可以设为所有Controller。

好处是可以直接获取返回值，不需要过滤静态资源请求。

但是如果用注解方式，需要添加大量接口日志的话就比较麻烦。

3.spring-actuator

https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html

优点是官方、功能全面，如健康监测、请求记录、统计、整合外部监控系统等。

缺点是仅记录请求，不能记录参数和返回值。



1.ContentCachingResponseWrapper/ContentCachingRequestWrapper

request/response body只能读一次，读取过body数据，请求就关闭了。

spring提供了两个类解决这个事：

ContentCachingResponseWrapper/ContentCachingRequestWrapper

```java
ContentCachingResponseWrapper wrapper = new ContentCachingResponseWrapper(response);
chain.doFilter(requestWrapper, wrapper);

// do something
String responseData = new String(responseWrapper.getContentAsByteArray())
    
wrapper.copyBodyToResponse();        
```

读完以后再将原数据拷贝回去。







