## LazyInitializationException

参考博客：

[the-best-way-to-handle-the-lazyinitializationexception](https://vladmihalcea.com/the-best-way-to-handle-the-lazyinitializationexception/)

懒加载的机制：

- 在非刚需查询`LAZY`属性时，生成代理类代替真实对象，代理类的属性值均为默认值。

- 在调用`get`方法刚需查询时，生成`query`语句查询真实对象。

由此可知，懒加载的前提条件是必须在`session`未关闭的情况下进行，否则无法查询真实对象。

解决该异常的思路：

- 保持`session`开启
- 一步到位查询所需数据

### 1.不推荐

参考博客：

[The Open Session In View Anti-Pattern](https://vladmihalcea.com/the-open-session-in-view-anti-pattern/)

缺点是集成测试速度慢；在无持续事务的情况下，保持`session`开启，增加连接池压力。

其中提到了`N+1`问题，参考：

[what-is-the-n1-selects-problem-in-orm-object-relational-mapping](https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm-object-relational-mapping)

如果不是`view`层查询导致的异常，则开启这个属性也无作用。验证如下：

直接触发`@Component`下的方法调用`testQuery()`,与@`Controller@Service`调用`testQuery()`，前者未进入`OpenSessionInViewFilter.doFilterInternal`方法。

[hibernate.enable_lazy_load_no_trans Anti-Pattern](https://vladmihalcea.com/the-hibernate-enable_lazy_load_no_trans-anti-pattern/)

在`session`关闭后，新建连接查询`LAZY`属性，增大连接池压力。

### 2.DTO映射查询

除了需要写很多针对性的`DTO`和查询语句以外，在`@ManyToOne@OneToOne`的情况下更为适用，在`@OneToMany`的情况下，通过`DTO`无法查询包含`list`的对象。

参考博客：

https://stackoverflow.com/questions/12379238/hql-new-list-within-new-object

### 3.小结

无完美的解决方案，具体情况具体分析。

另外，注解中`FetchType`的默认值并非总是`LAZY`。

如`@OneToMany`的`FetchType`默认值是`LAZY`，而`@ManyToOne`默认为`EAGER`。

