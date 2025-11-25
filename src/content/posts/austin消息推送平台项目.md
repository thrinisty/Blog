---
title: austin消息推送平台项目
published: 2025-11-17
updated: 2025-11-17
description: ''
image: ''
tags: [project]
category: 'project'
draft: false 
---

# Austin

## 依赖配置

Maven仓库

```xml
  <!-- aliyun yun -->
	<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
  </mirror>

  <!-- huawei yun -->
  <mirror>
      <id>huaweicloud</id>
      <mirrorOf>*</mirrorOf>
      <url>https://repo.huaweicloud.com/repository/maven/</url>
  </mirror>
  
   <!-- 中央仓库1 -->
  <mirror>
      <id>repo1</id>
      <mirrorOf>central</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://repo1.maven.org/maven2/</url>
  </mirror>

  <!-- 中央仓库2 -->
  <mirror>
      <id>repo2</id>
      <mirrorOf>central</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://repo2.maven.org/maven2/</url>
  </mirror>
```



## 项目模块化

我们创建项目的时候以业务作为区分将各个项目模块化，不同职责被分到对应的模块上而`austin`直属下的`pom`文件就一般只用来管理依赖

利用modules标签包含以下子模块

```xml
<modules>
    <module>common</module>
    <module>support</module>
    <module>stream</module>
    <module>handler</module>
    <module>web</module>
    <module>service</module>
    <module>service-impl</module>
    <module>cron</module>
</modules>
```

在父工程中引入SpringBoot版本，后续再子模块中导入依赖的时候就不需要指定版本

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.6</version>
    <relativePath/>
</parent>
```



### SpringBoot

在子模块web中接入SpringBoot

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

```java
@SpringBootApplication
public class AustinApplication {
    public static void main(String[] args) {
        SpringApplication.run(AustinApplication.class, args);
    }
}
```



### 常用工具

在日常开发中常用的一些开发工具，同样的在父工程中进行定义

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.7.15</version>
        </dependency>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>31.0.1-jre</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.83</version>
        </dependency>
        <dependency>
             <groupId>org.projectlombok</groupId>
             <artifactId>lombok</artifactId>
             <version>1.18.34</version>
         </dependency>
    </dependencies>
</dependencyManagement>
```



### 集成日志

在线上我们不常用sout进行日志的打印，因为需要对于时间、持久化的一些处理，这个时候日志框架是更好的选择，Slf4j是日志框架的接口

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

在resource资源下引入logback.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="30 seconds">

    <!-- 定义通用日志格式 -->
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>

    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <!-- 主日志文件（按天滚动，保留30天） -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <!-- ERROR级别日志单独输出 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/error.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <!-- 异步日志（提升性能） -->
    <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>512</queueSize>
        <discardingThreshold>0</discardingThreshold>
        <appender-ref ref="FILE"/>
    </appender>

    <!-- 根日志级别（可覆盖包路径级别） -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

    <!-- 特定包/类日志级别（示例） -->
    <logger name="com.example" level="DEBUG"/>
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.hibernate.SQL" level="DEBUG"/>

</configuration>
```

通过@Slf4j注解实现日志的集成

```java
/**
 * 测试接口
 */
@Slf4j
@RestController
@RequestMapping("test")
public class TestController {
    @GetMapping
    public String test() {
        log.info("test");
        return "test";
    }
}
```

```
2025-11-16 22:26:30.259 [http-nio-8080-exec-3] INFO  c.l.m.web.controller.TestController - test
```



## 短信发送

腾讯云SDK

```xml
<dependency>
    <groupId>com.tencentcloudapi</groupId>
    <artifactId>tencentcloud-sdk-java-sms</artifactId>
    <version>3.1.1357</version>
</dependency>
```

```java
@Slf4j
public class Main {
    public static void main(String[] args) {
        send("", "", "", "", "");
    }
    public static void send(String phone, String content,String secretId,String secretKey,String sdkAppId) {

        try {

            /**
             * 初始化
             */
            Credential cred = new Credential(secretId, secretKey);
            HttpProfile httpProfile = new HttpProfile();
            httpProfile.setEndpoint("sms.tencentcloudapi.com");
            ClientProfile clientProfile = new ClientProfile();
            clientProfile.setHttpProfile(httpProfile);
            SmsClient client = new SmsClient(cred, "ap-guangzhou", clientProfile);

            /**
             * 组装入参
             */
            SendSmsRequest req = new SendSmsRequest();
            String[] phoneNumberSet1 = new String[]{phone};
            req.setPhoneNumberSet(phoneNumberSet1);
            req.setSmsSdkAppId(sdkAppId);
            req.setSignName("Java3y公众号");
            req.setTemplateId("1182097");
            String[] templateParamSet1 = {content};
            req.setTemplateParamSet(templateParamSet1);
            req.setSessionContext(IdUtil.fastSimpleUUID());

            /**
             * 发送
             */
            SendSmsResponse response = client.SendSms(req);
            log.info(JSON.toJSONString(response));
        } catch (Exception e) {
            log.error(Throwables.getStackTraceAsString(e));
        }
    }
}
```



## 数据库

这个项目使用的是MySQL数据库，用于存储消息发送后短信的凭证，以及拉取的回执，并在后续提供一些发送消息的可视化业务

```


CREATE TABLE `message_template`
(
    `id`               bigint(20) NOT NULL AUTO_INCREMENT,
    `name`             varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '标题',
    `audit_status`     tinyint(4) NOT NULL DEFAULT '0' COMMENT '当前消息审核状态： 10.待审核 20.审核成功 30.被拒绝',
    `flow_id`          varchar(50) COLLATE utf8mb4_unicode_ci COMMENT '工单ID',
    `msg_status`       tinyint(4) NOT NULL DEFAULT '0' COMMENT '当前消息状态：10.新建 20.停用 30.启用 40.等待发送 50.发送中 60.发送成功 70.发送失败',
    `cron_task_id`     bigint(20) COMMENT '定时任务Id (xxl-job-admin返回)',
    `cron_crowd_path`  varchar(500) COMMENT '定时发送人群的文件路径',
    `expect_push_time` varchar(100) COLLATE utf8mb4_unicode_ci COMMENT '期望发送时间：0:立即发送 定时任务以及周期任务:cron表达式',
    `id_type`          tinyint(4) NOT NULL DEFAULT '0' COMMENT '消息的发送ID类型：10. userId 20.did 30.手机号 40.openId 50.email 60.企业微信userId',
    `send_channel`     tinyint(4) NOT NULL DEFAULT '0' COMMENT '消息发送渠道：10.IM 20.Push 30.短信 40.Email 50.公众号 60.小程序 70.企业微信 80.钉钉机器人 90.钉钉工作通知 100.企业微信机器人 110.飞书机器人 110. 飞书应用消息 ',
    `template_type`    tinyint(4) NOT NULL DEFAULT '0' COMMENT '10.运营类 20.技术类接口调用',
    `msg_type`         tinyint(4) NOT NULL DEFAULT '0' COMMENT '10.通知类消息 20.营销类消息 30.验证码类消息',
    `shield_type`      tinyint(4) NOT NULL DEFAULT '0' COMMENT '10.夜间不屏蔽 20.夜间屏蔽 30.夜间屏蔽(次日早上9点发送)',
    `msg_content`      varchar(600) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '消息内容 占位符用{$var}表示',
    `send_account`     tinyint(4) NOT NULL DEFAULT '0' COMMENT '发送账号 一个渠道下可存在多个账号',
    `creator`          varchar(45) COLLATE utf8mb4_unicode_ci  NOT NULL DEFAULT '' COMMENT '创建者',
    `updator`          varchar(45) COLLATE utf8mb4_unicode_ci  NOT NULL DEFAULT '' COMMENT '更新者',
    `auditor`          varchar(45) COLLATE utf8mb4_unicode_ci  NOT NULL DEFAULT '' COMMENT '审核人',
    `team`             varchar(45) COLLATE utf8mb4_unicode_ci  NOT NULL DEFAULT '' COMMENT '业务方团队',
    `proposer`         varchar(45) COLLATE utf8mb4_unicode_ci  NOT NULL DEFAULT '' COMMENT '业务方',
    `is_deleted`       tinyint(4) NOT NULL DEFAULT '0' COMMENT '是否删除：0.不删除 1.删除',
    `created`          int(11) NOT NULL DEFAULT '0' COMMENT '创建时间',
    `updated`          int(11) NOT NULL DEFAULT '0' COMMENT '更新时间',
    PRIMARY KEY (`id`),
    KEY                `idx_channel` (`send_channel`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci COMMENT ='消息模板信息';



CREATE TABLE `sms_record`
(
    `id`                  bigint(20) NOT NULL AUTO_INCREMENT,
    `message_template_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '消息模板ID',
    `phone`               bigint(20) NOT NULL DEFAULT '0' COMMENT '手机号',
    `supplier_id`         tinyint(4) NOT NULL DEFAULT '0' COMMENT '发送短信渠道商的ID',
    `supplier_name`       varchar(40) COLLATE utf8mb4_unicode_ci  NOT NULL DEFAULT '' COMMENT '发送短信渠道商的名称',
    `msg_content`         varchar(600) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '短信发送的内容',
    `series_id`           varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '下发批次的ID',
    `charging_num`        tinyint(4) NOT NULL DEFAULT '0' COMMENT '计费条数',
    `report_content`      varchar(50)                             NOT NULL DEFAULT '' COMMENT '回执内容',
    `status`              tinyint(4) NOT NULL DEFAULT '0' COMMENT '短信状态： 10.发送 20.成功 30.失败',
    `send_date`           int(11) NOT NULL DEFAULT '0' COMMENT '发送日期：20211112',
    `created`             int(11) NOT NULL DEFAULT '0' COMMENT '创建时间',
    `updated`             int(11) NOT NULL DEFAULT '0' COMMENT '更新时间',
    PRIMARY KEY (`id`),
    KEY                   `idx_send_date` (`send_date`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci COMMENT ='短信记录信息';
```



## 责任链模板

方法接口，其中实现责任链中每个节点调用的方法

```java
/**
 * 业务执行器
 */
public interface BusinessProcess {
    void process(ProcessContext context);
}
```

责任链的上下文，要求传入ProcessContext主要是责任链下游可能需要上游的一些信息

```java
/**
 * 责任链上下文
 */
public class ProcessContext {
    // 标识责任链的code
    private String code;
    // 存储上下文的真正载体
    private Model model;
    // 责任链中断的标识
    private Boolean needBreak = false;
}
```

通过ProcessTemplate，放入多个被实现的BusinessProcess接口组装成为链，提供获取链的方法

```java
/**
 * 业务执行模板（把责任链的逻辑串起来）
 */
public class ProcessTemplate {
    private List<BusinessProcess> processList;
    public List<BusinessProcess> getProcessList() {
        return processList;
    }
    public void setProcessList(List<BusinessProcess> processList) {
        this.processList = processList;
    }
}
```

执行器，这里还额外的用了一个责任链工厂存储不同的责任链，根据业务code选取不同的责任链

```java
/**
 * 责任链的流程控制器（整个责任链的执行流程通用控制）
 */
@Data
public class ProcessController {
    
    // 不同的code 对应不同的责任链
    private Map<String, ProcessTemplate> templateConfig = null;

    public void process(ProcessContext context) {
        //根据上下文的Code 执行不同的责任链
        String businessCode = context.getCode();
        ProcessTemplate processTemplate = templateConfig.get(businessCode);
        List<BusinessProcess> actionList = processTemplate.getProcessList();
        //遍历某个责任链的流程节点
        for (BusinessProcess action : actionList) {
            try {
                action.process(context);
                if (context.getNeedBreak()) {
                    break;
                }
            } catch (Exception e2) {
                //...
            }
        }
    }
}
```

