# java日志

## jdk中的日志

### 初步使用

```java
// 1.获取日志对象
Logger logger = Logger.getLogger("club.codeqi.log.QuickTest");
// 2.日志记录输出
logger.severe("severe");
logger.warning("warning");
logger.info("info");
logger.config("cofnig");
logger.fine("fine");
logger.finer("finer");
logger.finest("finest");
```

默认logger只会输出info级别以上的日志信息

### 自定义日志级别和formatter对象

```java
// 1.创建日志记录器对象
Logger logger = Logger.getLogger("club.codeqi.log.JULTest");
// 一、自定义日志级别
// a.关闭系统默认配置
logger.setUseParentHandlers(false);
// b.创建handler对象
ConsoleHandler consoleHandler = new ConsoleHandler();
// c.创建formatter对象
SimpleFormatter simpleFormatter = new SimpleFormatter();
// d.进行关联
consoleHandler.setFormatter(simpleFormatter);
logger.addHandler(consoleHandler);
// e.设置日志级别
logger.setLevel(Level.ALL);
consoleHandler.setLevel(Level.ALL);
// 二、输出到日志文件
FileHandler fileHandler = new FileHandler("d:/logs/jul1.log");//需要先在d盘下创建logs文件夹
fileHandler.setFormatter(simpleFormatter);
logger.addHandler(fileHandler);
fileHandler.setLevel(Level.INFO);
// 2.日志记录输出
logger.severe("severe");
logger.warning("warning");
logger.info("info");
logger.config("config");
logger.fine("fine");
logger.finer("finer");
logger.finest("finest");
```

自定义的步骤：

1. 禁用默认配置：logger.setUseParentHandlers(false);
2. 设置handler：new ConsoleHandler();创建控制台输出handler；new FileHandler("d:/logs/jul1.log");创建文件输出handler；
3. 设置format：consoleHandler.setFormatter(simpleFormatter);
4. 设置logger的handler：logger.addHandler(consoleHandler);
5. 设置logger的等级：logger.setLevel(Level.ALL);
6. 设置handler的等级：consoleHandler.setLevel(Level.ALL);

## springboot中使用日志

1. springboot 底层默认使用**logback**作为日志实现。
2. 使用了SLF4J作为日志门面（SLF4J中日志有四个等级，分别是error; warn; info; debug; trace）
3. 将JUL也转换成slf4j
4. 也可以使用log4j2作为日志门面，但是最终也是通过slf4j调用logback

### 配置文件

```properties
logging.level.club.codeqi=trace #设置哪个包输出日志，以及日志等级
# 在控制台输出的日志的格式 同logback
logging.pattern.console=%d{yyyy-MM-dd} [%thread] [%-5level] %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.file=D:/logs/springboot.log
logging.pattern.file=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
```

