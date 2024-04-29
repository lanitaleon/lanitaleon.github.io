---
layout: post
title: Hibernate-Search 无法获取 ORM session
---

起因是引入hibernate-search后，通过massIndexes生成索引时获取ORM session失败。

```java
SearchSession searchSession = Search.session(entityManager);
```

#### 表象

1.阶段1

项目目录结构为com.fscut，配置文件KnowledgeWebConfig位于包conf下。

此时失败，修改项目结构为com.fscut.kb（存疑），

ORM session获取成功

2.阶段2

原项目git下源码位于PlatformKnowledge文件夹下，

将src从PlatformKnowledge直接移到根目录下，删除PlatformKnowledge文件夹，

此时失败，将配置文件KnowledgeWebConfig的package修改如下：

```
// 原
package com.fscut.kb.config;
// 新 此时标红
package com.fscut.config;
```

修改为错误的包路径后，ORM session获取成功。

3.小结

不管是1，2阶段，普通jpa查询@query查询均正常。

#### 分析

经过测试，去掉KnowledgeWebConfig的继承关系也可以成功获取ORM session。

```java
package com.fscut.kb.config;

@Configuration
public class KnowledgeWebConfig { //extends WebMvcConfigurationSupport {
```

与该父类相关的两个override并不在影响之列，只要继承了就会获取失败。

初步判断，该配置影响了springboot默认的autoconfiguration。

类似博客参考：

https://blog.csdn.net/qq_38951372/article/details/96423781

但是未发现完全一致的博客参考，毕竟在自定义配置之后依然启用了默认设置。

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.add(responseBodyConverter());
    converters.add(jackson2HttpMessageConverter());
    addDefaultHttpMessageConverters(converters);
}
```

或者说，并不知道为什么`WebMvcConfigurationSupport`的实现方式跟项目路径还有关系。

#### 方案

重写WebConfig，通过WebMvcConfigurer而非WebMvcConfigurationSupport。

```java
@Configuration
public class CustomWebConfig extends WebMvcConfigurationSupport {}

@Configuration
public class CustomWebConfig implements WebMvcConfigurer {}
```

参考博客：

https://www.jianshu.com/p/4f2807a5468c

#### 测试

1.跨域是否生效

生效。

测试环境和流程描述：

本地，通过origin：null发送请求，在携带tgc cookie的情况下未发生跨域异常。

周心萌本地，由于跨域未携带tgc cookie导致登录校验未通过，导致“跨域异常”。

经过讨论分析，由于金色平台前端和cas域名原本就一样，如下Origin所示，同源的情况下默认就会带上cookie，故金色平台请求携带了tgc cookie。

```
GET /api/support/listTreeType HTTP/1.1
Host: kb.bcjgy.com
Connection: keep-alive
Accept: application/json, text/plain, */*
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.113 Safari/537.36
Origin: http://gold.bcjgy.com
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://gold.bcjgy.com/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: TGC=TGT-dc8a6801-50a7-48f3-98f7-4a479c6a0870
```

而真正跨域的时候，不会默认带上cookie，如同周心萌本地zxm.bcjgy.com请求我本地IP。

此时cookie默认不携带，tgc校验失败，表征为console出现跨域异常提示。

故金色平台前端需检查配置，使得所有请求在跨域情况下依然携带tgc cookie。

> tgc校验未通过表征为cors异常提示，以及服务不能访问或500异常时表征为cors异常提示此处不做讨论，以上并非真正意义上的跨域。

2.response-json是否生效

经过测试，原配置也没生效，去掉配置反而可以响应`@JsonFormat`。

