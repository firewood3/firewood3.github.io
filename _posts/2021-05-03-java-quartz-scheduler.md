---
title: 'JAVA Quartz Scheduler 사용 기록'
date: 2021-03-24 14:06:12 -0400
categories: java quartz scheduler
---

자바로 스케줄링 프로그램을 만들어 사용해 볼 기회가 있어 기록을 남긴다.

## quartz 라이브러리 등록

```code
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.2</version>
</dependency>
```

## SchedulerHelper 클래스 제작

1. SchedulerHepler 클래스는 생성자를 통해 Quartz 라이브러리에 있는 Scheduler 클래스를 생성
2. addScheduleJob 메소드를 통해 수행할 Job 클래스와 Job 클래스를 실행할 Cron 시간, Job 클래스에 전달할 매개변수를 Scheduler에 추가
3. startScheduler 메소드를 사용하여 추가된 Job을 실행

```java
package fd27.quartz.helper;

import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;

public class ScheduleHelper {
    private final Scheduler scheduler;

    public ScheduleHelper() throws SchedulerException {
        scheduler = StdSchedulerFactory.getDefaultScheduler();
    }

    public void addScheduleJob(String cron, Class<? extends Job> jobClass, String data) throws SchedulerException {
        JobDetail jobDetail = JobBuilder
                .newJob(jobClass)
                .usingJobData("data", data)
                .build();

        Trigger trigger = TriggerBuilder.newTrigger()
                .withSchedule(CronScheduleBuilder.cronSchedule(cron))
                .build();

        scheduler.scheduleJob(jobDetail, trigger);
    }

    public void startScheduler() throws SchedulerException {
        scheduler.start();
    }
}
```

## 실행될 두 개의 Job 클래스 제작

Hello를 출력하는 Job 클래스와 World를 출력하는 Job 클래스

```java
package fd27.quartz.job;

import org.quartz.Job;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

import java.sql.Timestamp;

public class PrintHelloJob implements Job {

    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        JobDataMap jobDataMap = jobExecutionContext.getJobDetail().getJobDataMap();
        Timestamp timestamp = new Timestamp(System.currentTimeMillis());
        String data = jobDataMap.getString("data");
        String message = timestamp + ", Hello Job, " + data + ", 스케줄링이 실행됩니다.";
        System.out.println(message);
    }
}
```

```java
package fd27.quartz.job;

import org.quartz.Job;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

import java.sql.Timestamp;

public class PrintWorldJob implements Job {

    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        JobDataMap jobDataMap = jobExecutionContext.getJobDetail().getJobDataMap();
        Timestamp timestamp = new Timestamp(System.currentTimeMillis());
        String data = jobDataMap.getString("data");
        String message = timestamp.toString() + ", World Job, " + data + ", 스케줄링이 실행됩니다.";
        System.out.println(message);
    }
}
```

## 스캐줄링 실행

SchedulerHelper 클래스를 통해 PrintHelloJob과 PrintWorldJob를 실행

```java
package fd27;

import fd27.quartz.helper.ScheduleHelper;
import fd27.quartz.job.PrintHelloJob;
import fd27.quartz.job.PrintWorldJob;
import org.quartz.SchedulerException;

public class Main {
    public static void main(String[] args) throws SchedulerException {
        ScheduleHelper scheduleHelper = new ScheduleHelper();
        scheduleHelper.addScheduleJob("0/5 * * * * ?", PrintHelloJob.class, "전달 데이터1");
        scheduleHelper.addScheduleJob("0/10 * * * * ?", PrintHelloJob.class, "전달 데이터2");
        scheduleHelper.addScheduleJob("0/15 * * * * ?", PrintWorldJob.class, "전달 데이터1");
        scheduleHelper.addScheduleJob("0/20 * * * * ?", PrintWorldJob.class, "전달 데이터2");
        scheduleHelper.startScheduler();
    }
}
```
