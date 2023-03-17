## logback输出日志到es

### 项目配置

+ 添加依赖

```xml
<dependency>
    <groupId>com.internetitem</groupId>
    <artifactId>logback-elasticsearch-appender</artifactId>
    <version>1.6</version>
</dependency>
```

+ 配置logback.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
    <property name="LOG_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{trace_id}]%-5level %logger{50} - %msg%n"/>
    
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <appender name="ELASTIC" class="com.internetitem.logback.elasticsearch.ElasticsearchAppender">
        
        <url>http://user:password@8.143.197.105:9200/_bulk</url>
        <index>flaginfo-logs-%date{yyyy-MM-dd}</index>
        <type>sms-api</type>
        <loggerName>flaginfo-logger</loggerName> 
		<errorLoggerName>flaginfo-error-logger</errorLoggerName>
        <connectTimeout>30000</connectTimeout>
        <errorsToStderr>false</errorsToStderr> 
        <includeCallerData>false</includeCallerData>
        <logsToStderr>false</logsToStderr> 
        <maxQueueSize>104857600</maxQueueSize> 
        <maxRetries>3</maxRetries> 
        <readTimeout>30000</readTimeout>
        <sleepTime>250</sleepTime>
        <rawJsonMessage>false</rawJsonMessage> 
        <includeMdc>false</includeMdc> 
        <maxMessageSize>100</maxMessageSize> 
		<authentication class="com.internetitem.logback.elasticsearch.config.BasicAuthentication"/> 
        <properties>
            <property>
                <name>host</name>
                <value>${HOSTNAME}</value>
                <allowEmpty>false</allowEmpty>
            </property>
            <property>
                <name>severity</name>
                <value>%level</value>
            </property>
            <property>
                <name>thread</name>
                <value>%thread</value>
            </property>
            <property>
                <name>stacktrace</name>
                <value>%ex</value>
            </property>
            <property>
                <name>logger</name>
                <value>%logger</value>
            </property>
            <property>
                <name>trace_id</name>
                <value>%X{trace_id}</value>
            </property>
        </properties>
        <headers>
            <header>
                <name>Content-Type</name>
                <value>application/json</value>
            </header>
        </headers>
    </appender>

    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="ELASTIC"/>
    </root>

</configuration>
```

+ 日志配置说明

ElasticsearchAppender：负责将logback日志输出写入es服务器；

com.internetitem.logback.elasticsearch.config.Settings：记录了写入es服务器时的配置信息

BasicAuthentication：指定连接es服务器的认证方式，没有可以不配置

properties：指定了记录到es中的字段信息，默认会记录时间戳和消息体

loggerName：指定将写入es的信息在控制台输出

errorLoggerName：将错误信息在控制台输出

+ 启动项目，将日志写入es

![](.\图片\Snipaste_2021-09-24_16-55-25.png)

看到如图所示的输出，即表示成功



### ES配置

+ 登录es管理界面，进入Stack Management，可以看到日志记录

![](.\图片\Snipaste_2021-09-24_17-00-45.png)

出现先上图的记录，就表示日志成功写入es服务器。

+ 配置索引模式，方便查看日志

进入索引模式界面，创建索引模式，索引模式名称 flaginfo-logs-*。

+ 进入日志页面，查看日志

![](.\图片\Snipaste_2021-09-24_17-05-27.png)



![](.\图片\Snipaste_2021-09-24_17-07-39.png)

### 提示

+ 搜索日志时，非keyword类型的字段需要在条件上加引号才能实现精确搜索