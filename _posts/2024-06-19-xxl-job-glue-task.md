---
layout: post
title: XXL-JOB > GLUE Task Execution
---

# Trigger

从 /jobinfo/trigger?id= 接口入手查看源码。

| method | comment |
| ------ | ------- |
| XxlJobServiceImpl#trigger | |
| JobTriggerPoolHelper#trigger | |
| JobTriggerPoolHelper#addTrigger | 将请求中指定的任务加入 trigger 队列 |
| XxlJobTrigger#trigger | |
| XxlJobTrigger#processTrigger | 执行 trigger |
| XxlJobTrigger#runExecutor | |
| XxlJobScheduler.getExecutorBiz | 根据执行器地址查找 executor |
| ExecutorBiz#run | |
| ExecutorBizImpl#run | 根据 GLUE 类型执行不同任务，注意到 TriggerParam 和 ScriptJobHandler |
| ScriptJobHandler#execute | 不同任务类型各自实现了 IJobHandler |
| ScriptUtil#markScriptFile | 将脚本内容在 executor 所在机器生成脚本文件 |
| ScriptUtil#execToFile | 执行脚本文件 |
| Runtime.getRuntime().exec | JDK 库自带的方法，执行 command |

# Conclusion

脚本内容由 job-admin 通过 rpc 传给 job-executor, 在 executor 所在服务器执行。

脚本执行会先生成脚本文件，再通过命令行执行脚本文件。
