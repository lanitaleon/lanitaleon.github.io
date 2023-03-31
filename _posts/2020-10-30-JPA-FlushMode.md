---
layout: post
title: JPA FlushMode
---

#### 一、问题描述

简化代码模型如下 

```
Device device = deviceRepo.findById(1).orElse(null);
Factory factory = new Factory();
device.setFactory(factory);
factoryRepo.findAll(PageRequest.of(0, 1));
```

异常信息如下

```
org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.TransientPropertyValueException:
object references an unsaved transient instance - save the transient instance before flushing : fscut.entity.
device.Device.factory -> fscut.entity.factory.Factory; nested exception is java.lang.IllegalStateException: org.
hibernate.TransientPropertyValueException: object references an unsaved transient instance - save the transient
instance before flushing : fscut.entity.device.Device.factory -> fscut.entity.factory.Factory
```

这个异常还是比较常见的，通常在没有处理好实体之间的级联关系时执行保存会触发类似问题。

由异常信息推测是查询时触发flush，在flush时发生的异常。

Device和Factory的关系如下:

```
public class Device {
    private Integer factoryId;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "factory_id", unique = true, insertable = false, updatable = false)
    @NotFound(action = NotFoundAction.IGNORE)
    private Factory factory;
}
```

#### 二、断点调试异常链路

##### 2.1SessionImpl.list

执行列表查询，查询时触发了autoFlush。

```
public List list(String query, QueryParameters queryParameters) throws HibernateException {
   ...
   autoFlushIfRequired( plan.getQuerySpaces() );
   ... 
}
```

##### 2.2SessionImpl.autoFlushIfRequired

注册AutoFlush事件，到这里基本上可以断定是flush时刷新导致异常。

```
protected boolean autoFlushIfRequired(Set querySpaces) throws HibernateException {
   checkOpen();
   if ( !isTransactionInProgress() ) {
      // do not auto-flush while outside a transaction
      return false;
   }
   AutoFlushEvent event = new AutoFlushEvent( querySpaces, this );
   for ( AutoFlushEventListener listener : listeners( EventType.AUTO_FLUSH ) ) {
      listener.onAutoFlush( event );
   }
   return event.isFlushRequired();
}
```

##### 2.3DefaultAutoFlushEventListener.onAutoFlush

先判断是否需要flush，然后执行flush。

```
public void onAutoFlush(AutoFlushEvent event) throws HibernateException {
    final EventSource source = event.getSession();
    try {
        source.getEventListenerManager().partialFlushStart();
        if ( flushMightBeNeeded(source) ) {
            // Need to get the number of collection removals before flushing to executions
            // (because flushing to executions can add collection removal actions to the action queue).
            final int oldSize = source.getActionQueue().numberOfCollectionRemovals();
            flushEverythingToExecutions(event);
        }
        ... 
    }
    ... 
}
```

##### 2.4DefaultAutoFlushEventListener.flushMightBeNeeded

source的flushMode与是否开启事务有关，在开启事务的情况下默认flushMode是AUTO，没开(断点显示)是MANUAL。

```
private boolean flushMightBeNeeded(final EventSource source) {
    return !source.getHibernateFlushMode().lessThan( FlushMode.AUTO )
            && source.getDontFlushFromFind() == 0
            && ( source.getPersistenceContext().getNumberOfManagedEntities() > 0 ||
                    source.getPersistenceContext().getCollectionEntries().size() > 0 );
}
```

##### 2.5AbstractFlushingEventListener.flushEverythingToExecutions

这里的flushEverything是指session中被管理的所有实体的所有字段。

```
protected void flushEverythingToExecutions(FlushEvent event) throws HibernateException {
    ...
    prepareEntityFlushes( session, persistenceContext );
        ...
}
```

##### 2.6AbstractFlushingEventListener.prepareEntityFlushes

在刷新前检查实体类中的级联实体，以便执行级联保存或更新。

```
private void prepareEntityFlushes(EventSource session, PersistenceContext persistenceContext) throws
HibernateException {
    LOG.debug( "Processing flush-time cascades" );
    final Object anything = getAnything();
    //safe from concurrent modification because of how concurrentEntries() is implemented on IdentityMap
    for ( Map.Entry<Object,EntityEntry> me : persistenceContext.reentrantSafeEntityEntries() ) {
//      for ( Map.Entry me : IdentityMap.concurrentEntries( persistenceContext.getEntityEntries() ) ) {
        EntityEntry entry = (EntityEntry) me.getValue();
        Status status = entry.getStatus();
        if ( status == Status.MANAGED || status == Status.SAVING || status == Status.READ_ONLY ) {
            cascadeOnFlush( session, entry.getPersister(), me.getKey(), anything );
        }    
    }
}
```

##### 2.7AbstractFlushingEventListener.cascadeOnFlush >> Cascade.cascade >> CascadingActions.noCascade

检查并处理需要级联的实体，这里已经看到了抛出异常的地方。

```
public void noCascade(
        EventSource session,
        Object parent,
        EntityPersister persister,
        Type propertyType,
        int propertyIndex) {
    if ( propertyType.isEntityType() ) {
        Object child = persister.getPropertyValue( parent, propertyIndex );
        String childEntityName = ((EntityType) propertyType).getAssociatedEntityName( session.getFactory() );
        if ( child != null
                && !isInManagedState( child, session )
                && !(child instanceof HibernateProxy) //a proxy cannot be transient and it breaks ForeignKeys.isTransient
                && ForeignKeys.isTransient( childEntityName, child, null, session ) ) {
            String parentEntiytName = persister.getEntityName();
            String propertyName = persister.getPropertyNames()[propertyIndex];
            throw new TransientPropertyValueException(
                "object references an unsaved transient instance - save the transient instance before flushing",
                childEntityName,
                parentEntiytName,
                propertyName
            );
        }
    }
}
```

##### 2.8CascadingActions.isInManagedState

可以看到从已经持久化的上下文中找不到对应的关联实体，判定为异常。

```
private boolean isInManagedState(Object child, EventSource session) {
    EntityEntry entry = session.getPersistenceContext().getEntry( child );
    return entry != null &&
            (
                entry.getStatus() == Status.MANAGED ||
                    entry.getStatus() == Status.READ_ONLY ||
                    entry.getStatus() == Status.SAVING
            );
}
```

#### 三、小结

在查询前会检查当前上下文中管理的所有实体和实体的所有字段，包括级联实体，此时如果发现未被持久化的实体，就抛出异常。

为避免误解，先将第一小节中的代码模型简化如下:

```
Blog blog = blogRepo.findById(1).orElse(null);
Location location = new Location();
blog.setLocation(location);
factoryRepo.findAll(PageRequest.of(0, 1));
```

##### 3.1为什么在查询前要刷新

设计如此，为了保证查询的结果是最新的。

##### 3.2为什么是立刻刷新

其实也不是立刻刷新，是根据FlushMode来的，**JPA默认的FlushMode是AUTO**。 

其实这也意味着**JPA不推荐开发者自行flush**，flush应当隐藏在JPA背后，由JPA决定。

将代码模型改为如下，设置FlushMode为COMMIT。 

可以看到在查询执行后并未立刻抛出异常，而是在整个函数执行完成后触发异常。

```
Blog blog = blogRepo.findById(1).orElse(null);
Location location = new Location();
blog.setLocation(location);
factoryRepo.getEntityManager().setFlushMode(FlushModeType.COMMIT);
factoryRepo.findAll(PageRequest.of(0, 1));
```

所以，可以直接将代码模型简化成这样。

```
Blog blog = blogRepo.findById(1).orElse(null);
Location location = new Location();
blog.setLocation(location);
```

另外，如果是在事务没有开启或直接设置FlushMode为MANUAL的情况下，不会触发异常。 

##### 3.2为什么要刷新所有实体

在小节2.2中可以看到AuthFlushEvent事件的source是SessionImpl本身，所以就把Session中所有实例都检查了一遍。

##### 3.3解决问题

在3.2中提到了，如果FlushMode为MANUAL就不会触发异常，那除了设置FlushMode为MANUAL，是否还有其他方案? 有，设置CascadeType为ALL。 

这么做的含义就是告诉JPA，这个关联实体完全由父级Blog做主，其结果就是在JPA提交事务后，临时对象Location被持久化到数据库了。 

这显然不是我们期望的结果。

**所以，在使用JPA(或者类似的ORM)时，不要写这样的代码。**

**查询就是查询，变更就是变更，不应该变更到一半就不管了。**

比如这个问题的原代码实现中，device中set的Factory其实是为了传参数。

传参数就传参数，传完就要处理掉，哪怕是这样处理。

```
Blog blog = blogRepo.findById(1).orElse(null);
Location location = new Location();
blog.setLocation(location);
blog.setLocation(null);
```

当然了，最好不要这样传参数，会给阅读代码的人造成不适。

JPA在数据库和应用层之间又做了一层，这一层的内容与数据库的数据是不完全对应的，什么时候提交到数据库、什么时候刷新都是由框架去做设计的，所以不能任意操作。

另，**仅使用hibernate不会有这样的问题**。
