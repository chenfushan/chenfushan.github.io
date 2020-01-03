# spring-boot 日志配置

spring-boot 默认使用logback来进行日志打印. logback是一个很好的日志系统. 但是我们记录日志的时候, 使用的是日志框架而不是日志系统. 为什么?

这是由于日志框架仅仅是日志的一套接口, 并无实现. 在日志接口的实现中, 可以使用任意喜欢的日志系统. 这样在以后的系统日志更换的话, 无需修改代码, 那么为什么会有这种需求呢? 因为在系统的集成过程中, 有一些系统使用的是不同的日志系统. 那么就需要统一日志系统. 如果一个系统有多个日志系统, 那么会导致系统日志无法打印.

## 日志系统

常见的日志系统有: `logback, JUL(java.util.logging), log4j, Log4j2`等等.

最开始的时候大家使用`log4j`, 然后出现了`JUL`. 以及后面的其他日志系统, 导致打印日志的系统混乱, 不同的日志系统不兼容, 导致了后面的日志框架的出现. 为了能够让系统的开发日志能够更好的兼容.

## 日志框架

常用的日志框架有: `JCL(Jakarta Commons Logging)` 和 `slf4j`.

### 引入slf4j



## log4j 日志系统

spring-boot已经使用了logback日志, 无需继续使用引入其他日志实现, 但是我在做项目的过程中, 已经习惯了使用log4j. 所以还是决定了排除了`logback`.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j</artifactId>
    <version>1.3.8.RELEASE</version>
</dependency>
```

## 日志打印规范

在使用`slf4j`的时候, 打印日志也需要有规范? 这个是约定的规范, 来源于阿里巴巴开源出来的JAVA研发规定.

对于`info / debug`级别的日志在打印的时候, 需要注意尽量使用占位符.

```java

//正确打印方式 1
LOGGER.debug("log id: {} and symbol : {} ", id, symbol);

//正确打印方式 2
if (LOGGER.isInfoEnabled()) {
    LOGGER.info("log id: "+id+" and symbol : "+symbol);
}

//错误示范 3
LOGGER.info("log id: "+id+" and symbol : "+symbol);

```
上面这三种打印方式, 前两种都是正确的打印方式, 直接说为什么第三种打印方式是错误的, 是因为线上的日志级别一般在`warn`上. 有的时候开`info`日志. 但是绝对别开`debug`. 为啥 ? 约定! 然后在`info`的时候, 即使日志级别是`warn`. 但是字符串还是会拼接. 会导致无效的性能消耗.





