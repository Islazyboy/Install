### Quartz

### 1.什么是Quartz?

> Quartz是OpenSymphony开源组织在Job scheduling领域又一个开源项目，完全由Java开发，可以用来执行定时任务，类似于java.util.Timer。但是相较于Timer， Quartz增加了很多功能：
>
> - 持久性作业 - 就是保持调度定时的状态;
> - 作业管理 - 对调度作业进行有效的管理;

### 2.Quartz的应用场景

拿网上购物来说，当我们在购物车中完成了选择进行订单提交时，为了避免大量用户不及时付钱而导致大量的订单搁置在数据库，我们就需要对订单进行定时清理，这个时候我们就可以用到Quartz，当你提交订单之后，后台就会插入一条任务，时间长度是30Min，超过了30分钟就会执行这个任务，判断你的订单是否支付，未支付就会取消此次订单。

[![Dgiva6.png](https://s3.ax1x.com/2020/11/29/Dgiva6.png)](https://imgchr.com/i/Dgiva6)

### 3.Quartz的基本组成部分

- 调度器：Scheduler
- 任务：JobDetail
- 触发器：Trigger，包括SimpleTrigger和CronTrigger

[![DgFmi8.png](https://s3.ax1x.com/2020/11/29/DgFmi8.png)](https://imgchr.com/i/DgFmi8)

### 4.Quartz的核心详解

- Job和JobDetail
- JobExecutionContext
- JobDataMap
- Trigger、SimpleTrigger、CronTrigger

### 5.Quartz的代码实现

```
@Configuration      //1.主要用于标记配置类，兼备Component的效果。
@EnableScheduling   // 2.开启定时任务
public class SaticScheduleTask {
    //3.添加定时任务
    @Scheduled(cron = "0/5 * * * * ?")
    //或直接指定时间间隔，例如：5秒
    //@Scheduled(fixedRate=5000)
    private void configureTasks() {
        System.err.println("执行静态定时任务时间: " + LocalDateTime.now());
    }
}
```

### 6.Cron表达式

```
Cron表达式被用来配置CronTrigger实例。Cron表达式是一个由6，7个域（子表达式）和空格组成的字符串。每个子表达式都描述了一个单独的日程细节
```

| 域             | 是否强制 | 允许值             | 允许特殊字符    |
| -------------- | -------- | ------------------ | --------------- |
| `Seconds`      | `YES`    | `0-59`             | `, - * /`       |
| `Minutes`      | `YES`    | `0-59`             | `, - * /`       |
| `Hours`        | `YES`    | `0-23`             | `, - * /`       |
| `Day of month` | `YES`    | `1-31`             | `, - * ? / L W` |
| `Month`        | `YES`    | `1-12 or JAN-DEC`  | `, - * /`       |
| `Day of week`  | `YES`    | `1-7 or SUN-SAT`   | `, - * ? / L #` |
| `Year`         | `NO`     | `empty, 1970-2099` | `, - * /`       |