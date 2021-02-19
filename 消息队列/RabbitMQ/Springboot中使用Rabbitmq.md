修改pom文件，添加amqp依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



# 生产者

## 配置文件

```yaml
  #RabbitMQ配置
spring: 
  rabbitmq:
    host: 192.168.174.130
    port: 5672
    username: admin
    password: admin
    virtual-host: /admin
```

## Config文件定义队列

```java
@Configuration
public class RabbitConfig {
    @Bean
    public Queue queue(){
        return new Queue("hello");
    }
}
```

## 通过@Autowired AmqpTemplate使用消息队列

1. 通过@Autowired AmqpTemplate amqpTemplate;注入消息队列
2. 通过amqpTemplate.convertAndSend("hello",msg);发送消息，其中hello为Config文件中定义的队列

```java
@Aspect
@Component
public class JwtLoginAspect {

    public static final Logger LOGGER = LoggerFactory.getLogger("Token发放记录");
    
    @Autowired
    AmqpTemplate amqpTemplate;
    
    @Pointcut("execution(* club.codeqi.bean.user.userController.loginuserinfo(..))")
    public void successlogin(){}
    
    @Around("successlogin()")
    public Object aroundSendSMS(ProceedingJoinPoint pjp) throws Throwable {
        SMS sms = new SMS();
        sms.setPhone("17721330304");
        sms.setCode("1011");
        LOGGER.info("===============AOP向消息队列发送短信开始===============");
        LOGGER.info("向消息队列中发送消息：消息队列: "+"hello"+", 消息内容: "+sms.toString());
        amqpTemplate.convertAndSend("hello", JsonUtils.toString(sms));
        try {
            // 2、执行时
            Object obj =  pjp.proceed();
            LOGGER.info("===============AOP向消息队列发送短信结束===============");
            return obj;
        } catch (Throwable e) {
            LOGGER.error(e.toString());// 3、发生异常时
            LOGGER.error("===============AOP向消息队列发送短信结束===============");
            throw e;
        }

    }
}
```

---

# 消费者

## 配置文件

```yaml
  #RabbitMQ配置
spring: 
  rabbitmq:
    host: 192.168.174.130
    port: 5672
    username: admin
    password: admin
    virtual-host: /admin
```

## Config文件定义队列

```java
@Configuration
public class RabbitConfig {
    @Bean
    public Queue queue(){
        return new Queue("hello");
    }
}
```

## 通过@RabbitListener使用监听消息队列

@RabbitListener(queues = "hello")中的hello为定义在configration文件中的队列名称

```java
@Component
public class GetMsg {

    public static final Logger LOGGER = LoggerFactory.getLogger("消息发送记录");
    @RabbitListener(queues = "hello")
    public void myListener(String message){
        Map map = JsonUtils.toMap(message,String.class,String.class);
        LOGGER.info("接受到的消息是："+ map);
    }
}
```

