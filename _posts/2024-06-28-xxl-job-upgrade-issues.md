---
layout: post
title: XXL-JOB > Version upgrade (1.8.2 >> 2.4.1)
---

# Story

为了修复三方库的漏洞，需要升级 xxl-job-core，从 1.8.2 升到目前的最新版本 2.4.1。

对比后发现，数据库结构发生了较大变化，框架设计上也做了很大改动。

去掉了 quartz 的依赖，升级了 springboot 2.7，页面模板也进行了调整。

# Migration

由于关联项目和任务数量比较多，升级需要分步进行。

## 1. 部署新版本调度中心

新旧版本调度中心同时运行。

## 2. 迁移所有数据

包括 task，glue 脚本，log 等，参考脚本如下。

## 3. 批量替换 GLUE 类型的任务

只需要更新对应表的状态即可。

## 4. 替换 BEAN 类型任务

相关执行器项目逐步替换，全部替换完成后，再停掉旧版本调度中心。

# Issues

## 1. v2.4.1 调度中心增加多环境配置文件

`/src/main/resources` 同级创建 `/src/main/profiles`

pom 文件增加如下配置。

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/profiles</directory>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
</build>
```

## 2. Database Diff

总的来说，可以分成三种类型：

需要迁移数据的表、新增的表、已经移除的表。

| table name                       | remark    |
| -------------------------------- | --------- |
| xxl_job_info                     | 需要迁移数据    |
| xxl_job_log                      | 需要迁移数据    |
| xxl_job_logglue                  | 需要迁移数据    |
| xxl_job_registry                 | 需要迁移数据    |
| xxl_job_group                    | 需要迁移数据    |
| xxl_job_lock                     | v2.4.1 新增 |
| xxl_job_log_report               | v2.4.1 新增 |
| xxl_job_user                     | v2.4.1 新增 |
| XXL_JOB_QRTZ_JOB_DETAILS         | v2.4.1 移除 |
| XXL_JOB_QRTZ_TRIGGERS            | v2.4.1 移除 |
| XXL_JOB_QRTZ_SIMPLE_TRIGGERS     | v2.4.1 移除 |
| XXL_JOB_QRTZ_CRON_TRIGGERS       | v2.4.1 移除 |
| XXL_JOB_QRTZ_SIMPROP_TRIGGERS    | v2.4.1 移除 |
| XXL_JOB_QRTZ_BLOB_TRIGGERS       | v2.4.1 移除 |
| XXL_JOB_QRTZ_CALENDARS           | v2.4.1 移除 |
| XXL_JOB_QRTZ_PAUSED_TRIGGER_GRPS | v2.4.1 移除 |
| XXL_JOB_QRTZ_FIRED_TRIGGERS      | v2.4.1 移除 |
| XXL_JOB_QRTZ_SCHEDULER_STATE     | v2.4.1 移除 |

需要迁移数据的表详细对比如下：

`xxl_job_info`

| old field               | new field                 | new field comment | diff remark                                                    |
| ----------------------- | ------------------------- | ----------------- | -------------------------------------------------------------- |
| id                      | id                        |                   | -                                                              |
| job_group               | job_group                 |                   | -                                                              |
| job_desc                | job_desc                  |                   | -                                                              |
| add_time                | add_time                  |                   | -                                                              |
| update_time             | update_time               |                   | -                                                              |
| author                  | author                    |                   | -                                                              |
| alarm_email             | alarm_email               |                   | -                                                              |
| -                       | schedule_type             | 调度类型              | 1.8.2 默认 CRON<br/> 2.4.1 扩展为 NONE/CRON/FIX_RATE                |
| job_cron                | schedule_conf             |                   | 仅变更字段名称                                                        |
| -                       | misfire_strategy          | 调度过期策略            | 1.8.2 默认为 DO_NOTHING<br/>2.4.1 扩展为 DO_THING，FIRE_ONCE_NOW      |
| executor_route_strategy | executor_route_strategy   |                   | -                                                              |
| executor_handler        | executor_handler          |                   | -                                                              |
| executor_param          | executor_param            |                   | -                                                              |
| executor_block_strategy | executor_block_strategy   |                   | -                                                              |
| executor_fail_strategy  | executor_fail_retry_count |                   | 1.8.2 是邮件告警重试二选一<br/>2.4.1 同时支持重试和邮件告警                         |
| -                       | executor_timeout          | 任务执行超时时间，单位秒      | 2.4.1 支持配置超时秒数，大于0生效                                           |
| glue_type               | glue_type                 |                   | -                                                              |
| glue_source             | glue_source               |                   | -                                                              |
| glue_remark             | glue_remark               |                   | -                                                              |
| glue_updatetime         | glue_updatetime           |                   | -                                                              |
| child_jobkey            | child_jobid               | 子任务ID，多个逗号分割      | 1.8.2 {job_group_id}\_{job_id}，如："4_170"<br/>2.4.1 job_id，如：170 |
|                         | trigger_status            |                   | -                                                              |
|                         | trigger_last_time         |                   | -                                                              |
|                         | trigger_next_time         |                   | -                                                              |

迁移脚本参考：

```sql
insert into job_b.xxl_job_info (
    id,
    job_group,
    job_desc,
    add_time,
    update_time,
    author,
    alarm_email,
    schedule_type,
    schedule_conf,
    misfire_strategy,
    executor_route_strategy ,
    executor_handler ,
    executor_param ,
    executor_block_strategy ,
    glue_type ,
    glue_source ,
    glue_remark ,
    glue_updatetime ,
    child_jobid 
)
select
    id,
    job_group ,
    job_desc ,
    add_time ,
    update_time ,
    author ,
    alarm_email ,
    'CRON',
    job_cron ,
    'DO_NOTHING',
    executor_route_strategy ,
    executor_handler ,
    executor_param ,
    executor_block_strategy ,
    glue_type ,
    glue_source ,
    glue_remark ,
    glue_updatetime ,
    case
        when child_jobkey is not null then substring_index(child_jobkey, '_', -1)
        else child_jobkey
    end
from
    job_a.XXL_JOB_QRTZ_TRIGGER_INFO;
```

`xxl_job_log`

| old field        | new field                 | new field comment | diff remark              |
| ---------------- | ------------------------- | ----------------- | ------------------------ |
| id               | id                        |                   | -                        |
| job_group        | job_group                 |                   | -                        |
| job_id           | job_id                    |                   | -                        |
| executor_address | executor_address          |                   | -                        |
| executor_handler | executor_handler          |                   | -                        |
| executor_param   | executor_param            |                   | -                        |
| -                | executor_sharding_param   | 执行器任务分片参数         | 2.4.1 新增的分片广播，默认 NULL 即可 |
| -                | executor_fail_retry_count | 失败重试次数            | 2.4.1 默认0即可              |
| trigger_time     | trigger_time              |                   | -                        |
| trigger_code     | trigger_code              |                   | -                        |
| trigger_msg      | trigger_msg               |                   | -                        |
| handle_time      | handle_time               |                   | -                        |
| handle_code      | handle_code               |                   | -                        |
| handle_msg       | handle_msg                |                   | -                        |
| -                | alarm_status              | 告警状态              | 2.4.1 默认0即可              |
| glue_type        | -                         |                   | 2.4.1 移除，忽略即可            |

迁移脚本参考：

```sql
insert into job_b.xxl_job_log (
    id,
    job_group,
    job_id,
    executor_address,
    executor_handler ,
    executor_param ,
    trigger_time,
    trigger_code,
    trigger_msg,
    handle_time,
    handle_code,
    handle_msg
)
select
    id,
    job_group ,
    job_id,
    executor_address,
    executor_handler ,
    executor_param ,
    trigger_time,
    trigger_code,
    trigger_msg,
    handle_time,
    handle_code,
    handle_msg
from
    job_a.XXL_JOB_QRTZ_TRIGGER_LOG;   
```

`xxl_job_logglue`

| old field   | new field   | diff remark |
| ----------- | ----------- | ----------- |
| id          | id          | -           |
| job_id      | job_id      | -           |
| glue_type   | glue_type   | -           |
| glue_source | glue_source | -           |
| glue_remark | glue_remark | -           |
| add_time    | add_time    | -           |
| update_time | update_time | -           |

迁移脚本参考：

```sql
insert into job_b.xxl_job_logglue (
    id,
    job_id,
    glue_type,
    glue_source,
    glue_remark,
    add_time,
    update_time
)
select
    id,
    job_id,
    glue_type,
    glue_source,
    glue_remark,
    add_time,
    update_time
from
    job_a.XXL_JOB_QRTZ_TRIGGER_LOGGLUE;
```

`xxl_job_registry`

| old field      | new field      | diff remark |
| -------------- | -------------- | ----------- |
| id             | id             | -           |
| registry_group | registry_group | -           |
| registry_key   | registry_key   | -           |
| registry_value | registry_value | -           |
| update_time    | update_time    | -           |

迁移脚本参考：

```sql
insert into job_b.xxl_job_registry (
    id,
    registry_group,
    registry_key,
    registry_value,
    update_time
)
select
    id,
    registry_group,
    registry_key,
    registry_value,
    update_time
from
    job_a.XXL_JOB_QRTZ_TRIGGER_REGISTRY;    
```

`xxl_job_group`

| old field    | new field    | new field comment     | diff remark                                          |
| ------------ | ------------ | --------------------- | ---------------------------------------------------- |
| id           | id           |                       | -                                                    |
| app_name     | app_name     |                       | -                                                    |
| title        | title        |                       | -                                                    |
| order        | -            |                       | 2.4.1 移除，默认排序：ORDER BY t.app_name, t.title, t.id ASC |
| address_type | address_type | 执行器地址类型：0=自动注册、1=手动录入 | -                                                    |
| address_list | address_list | 执行器地址列表，多地址逗号分隔       | -                                                    |
| -            | update_time  |                       | 2.4.1 新增，默认为NULL                                     |

迁移脚本参考：

```sql
insert into job_b.xxl_job_group (
    id,
    app_name,
    title,
    address_type,
    address_list,
    update_time
)
select
    id,
    app_name,
    title,
    address_type,
    address_list,
    NOW()
from
    job_a.XXL_JOB_QRTZ_TRIGGER_GROUP;
```

## 3. Code diff

### 3.1 xxl-job config

```java
// old
XxlJobExecutor.setAppName();

// new 
XxlJobExecutor.setAppname();
```

### 3.2 Job handler

BEAN Task 的实现微调。

v1.8.2

```java
@JobHandler(value=”xxx”)
public class MyJobHandler extends IJobHandler {
    @Override
    public ReturnT<String> execute(String... params) {
        // ...
    }
}
```

v2.4.1

```java
@Component
public class MyJobHandler {
    @XxlJob(value = "xxx")
    public ReturnT<String> execute(String... params) {
        // ...
    }
}
```

## 4. Start and Stop tasks in batches

[XXL-JOB > Task start and stop](https://lanitaleon.github.io/2024/06/20/xxl-job-pause-task.html)

v2.4.1 批量启停任务只需要变更 `xxl_job_info.trigger_status`。

v1.8.2 批量暂停任务只需要变更 `xxl_job_qrtz_triggers.trigger_state`。

## 5. GLUE Task

[XXL-JOB > GLUE Task Execution](https://lanitaleon.github.io/2024/06/19/xxl-job-glue-task.html)
