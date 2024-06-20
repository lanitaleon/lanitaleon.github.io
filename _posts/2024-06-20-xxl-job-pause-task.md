---
layout: post
title: xxl-job > Task start and stop
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

查看发现，job-admin 注册该 bean 时，仅启动动态扫描注册器和异常报警的线程，和 2.4.1 的架构大不相同。

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

任务活动周期的管理全部交由 quartzScheduler 管理。

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

也就是完全依赖 quartz 框架，不能通过数据库刷新数据重载任务。