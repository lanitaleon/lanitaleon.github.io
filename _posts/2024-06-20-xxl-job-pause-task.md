---
layout: post
title: XXL-JOB > Task start and stop
---

如何知道 Task 的状态是否停止？

# v2.4.1

从数据库的表结构可知：

`xxl_job_info.trigger_status` 代表任务的状态，0 是停止 1 是运行。

查看如下接口的实现：

```
/jobinfo/pageList
/jobinfo/start
/jobinfo/stop
```

注意到如下配置：

```java
XxlJobAdminConfig#afterPropertiesSet
XxlJobScheduler#init
```

查看后发现 xxl-job-admin 在项目启动配置加载完成后，启动了一堆线程。

```java
public void init() throws Exception {
    // init i18n
    initI18n();

    // admin trigger pool start
    JobTriggerPoolHelper.toStart();

    // admin registry monitor run
    JobRegistryHelper.getInstance().start();

    // admin fail-monitor run
    JobFailMonitorHelper.getInstance().start();

    // admin lose-monitor run ( depend on JobTriggerPoolHelper )
    JobCompleteHelper.getInstance().start();

    // admin log report start
    JobLogReportHelper.getInstance().start();

    // start-schedule  ( depend on JobTriggerPoolHelper )
    JobScheduleHelper.getInstance().start();

    logger.info(">>>>>>>>> init xxl-job admin success.");
}
```

其中 `JobScheduleHelper` 会以大约 5000 毫秒的间隔，

将 `xxl_job_info` 表中配置的 `trigger_status = 1` 的任务加载出来，

如果当前时间戳大于 `trigger_next_time` 就将该任务加入 trigger 队列中准备执行。

也就是说，

如果想要批量暂停 task，只需要将 `xxl_job_info.trigger_status` 设为 1。

# v1.8.2

查看如下接口的实现：

```
/jobinfo/pageList
/jobinfo/resume
/jobinfo/pause
```

注意到：

```java
XxlJobDynamicScheduler#init
```

查看发现，`job-admin` 注册该 bean 时，仅启动动态扫描注册器和异常报警的线程，和 2.4.1 的架构大不相同。

```java
public void init() throws Exception {
    // admin registry monitor run
    JobRegistryMonitorHelper.getInstance().start();

    // admin monitor run
    JobFailMonitorHelper.getInstance().start();

    // admin-server(spring-mvc)
    NetComServerFactory.putService(AdminBiz.class, XxlJobDynamicScheduler.adminBiz);
    NetComServerFactory.setAccessToken(accessToken);

    // valid
    Assert.notNull(scheduler, "quartz scheduler is null");
    logger.info(">>>>>>>>> init xxl-job admin success.");
}
```

任务活动周期的管理全部交由 `quartzScheduler` 管理。

```xml
<bean id="quartzScheduler" lazy-init="false" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="autoStartup" value="true" />            <!--自动启动 -->
    <property name="startupDelay" value="20" />             <!--延时启动，应用启动成功后在启动 -->
    <property name="overwriteExistingJobs" value="true" />  <!--覆盖DB中JOB：true、以数据库中已经存在的为准：false -->
    <property name="applicationContextSchedulerContextKey"  value="applicationContextKey" />
    <property name="configLocation" value="classpath:quartz.properties"/>
</bean>

<bean id="xxlJobDynamicScheduler" class="com.xxl.job.admin.core.schedule.XxlJobDynamicScheduler" init-method="init" destroy-method="destroy" >
    <!-- (轻易不要变更“调度器名称”, 任务创建时会绑定该“调度器名称”) -->
    <property name="scheduler" ref="quartzScheduler"/>
    <property name="accessToken" value="${xxl.job.accessToken}" />
</bean>
```

也就是完全依赖 `quartz` 框架的任务调度管理。

## quartz

查看 `quartz` 的实现，注意到表 `xxl_job_qrtz_triggers.trigger_state`;

暂停 task 是不是控制这个值就可以了呢？

顺着 `/jobinfo/pageList` 继续看：

```java
StdScheduler#getTriggerState
QuartzScheduler#getTriggerState
JobStoreSupport#getTriggerState
StdJDBCDelegate#selectTriggerState
```

发现确实是通过查询数据库的值来确定任务状态。

以及它在 `quartz` 中的转换关系：

```java
String ts = getDelegate().selectTriggerState(conn, key);
if (ts == null) {
    return TriggerState.NONE;
}
if (ts.equals(STATE_DELETED)) {
    return TriggerState.NONE;
}
if (ts.equals(STATE_COMPLETE)) {
    return TriggerState.COMPLETE;
}
if (ts.equals(STATE_PAUSED)) {
    return TriggerState.PAUSED;
}
if (ts.equals(STATE_PAUSED_BLOCKED)) {
    return TriggerState.PAUSED;
}
if (ts.equals(STATE_ERROR)) {
    return TriggerState.ERROR;
}
if (ts.equals(STATE_BLOCKED)) {
    return TriggerState.BLOCKED;
}
return TriggerState.NORMAL; 
```

那么暂停一个任务只需要更新这个值就可以吗？

直接试验一下，

修改字段 `trigger_state`，将 `WAITING` 为 `PAUSED`，任务停止了。

但是重新启动不等同于把 PAUSED 再恢复 WAITING。

查看 `/jobinfo/pause` 的实现

```java
StdScheduler#pauseTrigger
QuartzScheduler#pauseTrigger
JobStoreSupport#pauseTrigger
```

在第三步中，更新了数据库的字段值。

```java
// JobStoreSupport#pauseTrigger
String oldState = getDelegate().selectTriggerState(conn,
        triggerKey);
if (oldState.equals(STATE_WAITING)
        || oldState.equals(STATE_ACQUIRED)) {
    getDelegate().updateTriggerState(conn, triggerKey,
            STATE_PAUSED);
} else if (oldState.equals(STATE_BLOCKED)) {
    getDelegate().updateTriggerState(conn, triggerKey,
            STATE_PAUSED_BLOCKED);
} 
```

第二步中，除了调用更新值的方法，还通知了两个线程。

```java
// QuartzScheduler#pauseTrigger
public void pauseTrigger(TriggerKey triggerKey) throws SchedulerException {
    validateState();

    resources.getJobStore().pauseTrigger(triggerKey);
    notifySchedulerThread(0L);
    notifySchedulerListenersPausedTrigger(triggerKey);
}
```

查看 `notifySchedulerListenersPausedTrigger`，这个方法，啥也没做。

```java
// SchedulerListener#triggerPaused
// SchedulerListenerSupport#triggerPaused

public void triggerPaused(TriggerKey triggerKey) {
}
```

查看 `notifySchedulerThread`，注意到 `QuartzSchedulerThread`。

```java
QuartzScheduler#notifySchedulerThread
SchedulerSignalerImpl#signalSchedulingChange
QuartzSchedulerThread#signalSchedulingChange
```

`QuartzSchedulerThread` 是一个触发任务执行的线程。

```java
// QuartzSchedulerThread#signalSchedulingChange
/**
 * <p>
 * Signals the main processing loop that a change in scheduling has been
 * made - in order to interrupt any sleeping that may be occuring while
 * waiting for the fire time to arrive.
 * </p>
 *
 * @param candidateNewNextFireTime the time (in millis) when the newly scheduled trigger
 * will fire.  If this method is being called do to some other even (rather
 * than scheduling a trigger), the caller should pass zero (0).
 */
public void signalSchedulingChange(long candidateNewNextFireTime) {
    synchronized(sigLock) {
        signaled = true;
        signaledNextFireTime = candidateNewNextFireTime;
        sigLock.notifyAll();
    }
}
```

`signaledNextFireTime` 设为 0，代表该触发器不再触发。

试想一个运行中的任务，此时它的下次执行时间是明天 1 点钟。

现在我们把 `trigger_state` 设为 `PAUSED`，却没有管任何正在运行的 `QuartzSchedulerThread`。

那么下次执行的时候，该触发器会执行吗？

直接查看 `QuartzSchedulerThread.run` 的实现。

```java
QuartzSchedulerThread#run
QuartzSchedulerThread#isCandidateNewTimeEarlierWithinReason
JobStoreSupport#triggerFired
```

注意到 `triggerFired`，查看实现。

```java
protected TriggerFiredBundle triggerFired(Connection conn,
        OperableTrigger trigger)
    throws JobPersistenceException {
    ... ...

    // Make sure trigger wasn't deleted, paused, or completed...
    try { // if trigger was deleted, state will be STATE_DELETED
        String state = getDelegate().selectTriggerState(conn,
                trigger.getKey());
        if (!state.equals(STATE_ACQUIRED)) {
            return null;
        }
    } catch (SQLException e) {
        throw new JobPersistenceException("Couldn't select trigger state: "
                + e.getMessage(), e);
    }
    ... ...
}
```

到这里可以安心了，执行前会根据数据库中 `trigger_state` 值拦截触发器。

所以 v1.8.2 可以通过修改 `xxl_job_qrtz_triggers.trigger_state` 的值批量暂停 task。
