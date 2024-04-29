---
layout: post
title: XXL-JOB 中的报警通知
---

1.执行器需要远程更新任务cron，且每个OEM拥有自己的task

需要在调度中心加个接口，允许给需要发送加工日报的OEM创建任务，

将oemId作为任务区别标识，创建成功后返回任务ID；

在执行器维护一个JOB ID配置，通过调度中心开放的HTTP接口修改每个OEM的任务执行时间；

2.报警通知扩展到企业微信

官方文档5.17

新增一种报警通知方式，只需要在调度中心新增一个组件实现JobAlarm接口即可。

```java
@Component
public class WxJobAlarm implements JobAlarm {
    private static Logger logger = LoggerFactory.getLogger(WxJobAlarm.class);

    @Override
    public boolean doAlarm(XxlJobInfo info, XxlJobLog jobLog) {
        String template = "定时任务%s执行失败，任务ID：%d，调度信息：%s，调度结果：%d";
        String content = String.format(template, info.getJobDesc(), info.getId(), jobLog.getTriggerMsg(), jobLog.getHandleCode());
        logger.info(content);
        return false;
    }
}
```

报警触发位于JobFailMonitorHelper中，可以看到调度或执行失败时将触发JobAlarmer加载的所有Alarm组件。

```java
// 2、fail alarm monitor
int newAlarmStatus = 0;     
// 告警状态：0-默认、-1=锁定状态、1-无需告警、2-告警成功、3-告警失败
if (info!=null && info.getAlarmEmail()!=null && info.getAlarmEmail().trim().length()>0) {
    boolean alarmResult = XxlJobAdminConfig.getAdminConfig().getJobAlarmer().alarm(info, log);
    newAlarmStatus = alarmResult?2:3;
} else {
    newAlarmStatus = 1;
}
```

而JobAlarmer在项目启动完成后加载了所有实现了JobAlarm接口的组件。

```java
@Component
public class JobAlarmer implements ApplicationContextAware, InitializingBean {

    private ApplicationContext applicationContext;
    private List<JobAlarm> jobAlarmList;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        Map<String, JobAlarm> serviceBeanMap = applicationContext.getBeansOfType(JobAlarm.class);
        if (serviceBeanMap != null && serviceBeanMap.size() > 0) {
            jobAlarmList = new ArrayList<JobAlarm>(serviceBeanMap.values());
        }
    }
}    
```
