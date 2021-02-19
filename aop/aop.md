# Aop

面向切面编程的几大关键词：通知、连接点、切点、切面

通知分为5种:

- 前置通知（Before）：在目标方法被调用之前调用通知功能；
- 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
- 返回通知（After-returning）：在目标方法成功执行之后调用通知；
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
- 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

连接点是指，在某个方法执行时，执行切面的程序代码，该方法被称为连接点，例如：* club.codeqi.bean..*Controller*.*(..)

切点是指使用execution()修饰的方法，例如：execution(* club.codeqi.bean..*Controller*.*(..))

切面是通知和切点的结合，例如：@Before("webLogProject()")

## 在springboot中使用Aop

1. 修改pom文件
2. 创建切面类
3. 编写切面类

### 修改pom文件

添加aop在springboot中的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 创建切面类

```java
@Aspect
@Component
public class ControllerAspect {
}
```

在springboot中的切面类必须使用@Aspect和@Component注解，否则切面没有效果

### 创建切点

``` java
@Pointcut("execution(* club.codeqi.bean..*Controller*.*(..))")
    public void webLogProject(){}
```

### 创建前置通知

```java
 @Before("webLogProject()")
    public void beginRequest(JoinPoint joinPoint){
        LOGGER.info("-------------------请求开始--------------------------");
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        StringBuilder sb = new StringBuilder();
        sb.append("请求路径:"  + " " + request.getRequestURL().toString() + "  " + request.getMethod() + "\n");
        if (request.getMethod().equalsIgnoreCase(RequestMethod.GET.name())) {
            Map<String, String[]> parameterMap = request.getParameterMap();
            Map<String, String> paramMap = new HashMap<>();
            parameterMap.forEach((key, value) -> paramMap.put(key, Arrays.stream(value).collect(joining(","))));
            sb.append("请求内容:" + paramMap);
        } else if (request.getMethod().equalsIgnoreCase(RequestMethod.POST.name())) {
            Object[] args = joinPoint.getArgs();
            StringBuilder stringBuilder = new StringBuilder();
            Arrays.stream(args).forEach(object -> stringBuilder.append(object.toString().replace("=", ":")));
            if (stringBuilder.length() == 0) {
                stringBuilder.append("{}");
            }
            sb.append("请求内容:" + stringBuilder.toString());
        }
        else if (request.getMethod().equalsIgnoreCase(RequestMethod.PUT.name())) {
            Object[] args = joinPoint.getArgs();
            StringBuilder stringBuilder = new StringBuilder();
            Arrays.stream(args).forEach(object -> stringBuilder.append(object.toString().replace("=", ":")));
            if (stringBuilder.length() == 0) {
                stringBuilder.append("{}");
            }
            sb.append("请求内容:" + stringBuilder.toString());
        }
        else if (request.getMethod().equalsIgnoreCase(RequestMethod.DELETE.name())) {
            Object[] args = joinPoint.getArgs();
            StringBuilder stringBuilder = new StringBuilder();
            Arrays.stream(args).forEach(object -> stringBuilder.append(object.toString().replace("=", ":")));
            if (stringBuilder.length() == 0) {
                stringBuilder.append("{}");
            }
            sb.append("请求内容:" + stringBuilder.toString());
        }
        LOGGER.info(String.valueOf(sb));
    }
```

### 创建有返回值的后置通知

```java
@AfterReturning(pointcut = "webLogProject()", returning = "ret")
public void endRequest(Object ret) {
    LOGGER.info("请求结果：" + ret.toString());
    LOGGER.info("--------------------请求结束-------------------------");
}
```

### 创建环绕通知

```java
@Around("webLogProject()")
    public Object serviceAOP(ProceedingJoinPoint pjp) throws Throwable {
        //1.开始
        try {
            // 2、执行时
            Object obj =  pjp.proceed();
//            LOGGER.info("请求结果：" + obj.toString());
//            LOGGER.info("--------------------请求结束-------------------------");
            return obj;
        } catch (Throwable e) {
            LOGGER.error(e.toString());// 3、发生异常时
            LOGGER.error("--------------------请求结束-------------------------");
            throw e;
        }
    }
```

---

## 完整代码

```java
package club.codeqi.Aspect.user;

import club.codeqi.utils.JsonUtils;
import net.minidev.json.JSONUtil;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.assertj.core.util.DateUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

import static java.util.stream.Collectors.joining;

@Aspect
@Component
public class ControllerAspect {

    public static final Logger LOGGER = LoggerFactory.getLogger("Request请求记录");

    @Pointcut("execution(* club.codeqi.bean..*Controller*.*(..))")
    public void webLogProject(){}

    @Before("webLogProject()")
    public void beginRequest(JoinPoint joinPoint){
        LOGGER.info("-------------------请求开始--------------------------");
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        StringBuilder sb = new StringBuilder();
        sb.append("请求路径:"  + " " + request.getRequestURL().toString() + "  " + request.getMethod() + "\n");
        if (request.getMethod().equalsIgnoreCase(RequestMethod.GET.name())) {
            Map<String, String[]> parameterMap = request.getParameterMap();
            Map<String, String> paramMap = new HashMap<>();
            parameterMap.forEach((key, value) -> paramMap.put(key, Arrays.stream(value).collect(joining(","))));
            sb.append("请求内容:" + paramMap);
        } else if (request.getMethod().equalsIgnoreCase(RequestMethod.POST.name())) {
            Object[] args = joinPoint.getArgs();
            StringBuilder stringBuilder = new StringBuilder();
            Arrays.stream(args).forEach(object -> stringBuilder.append(object.toString().replace("=", ":")));
            if (stringBuilder.length() == 0) {
                stringBuilder.append("{}");
            }
            sb.append("请求内容:" + stringBuilder.toString());
        }
        else if (request.getMethod().equalsIgnoreCase(RequestMethod.PUT.name())) {
            Object[] args = joinPoint.getArgs();
            StringBuilder stringBuilder = new StringBuilder();
            Arrays.stream(args).forEach(object -> stringBuilder.append(object.toString().replace("=", ":")));
            if (stringBuilder.length() == 0) {
                stringBuilder.append("{}");
            }
            sb.append("请求内容:" + stringBuilder.toString());
        }
        else if (request.getMethod().equalsIgnoreCase(RequestMethod.DELETE.name())) {
            Object[] args = joinPoint.getArgs();
            StringBuilder stringBuilder = new StringBuilder();
            Arrays.stream(args).forEach(object -> stringBuilder.append(object.toString().replace("=", ":")));
            if (stringBuilder.length() == 0) {
                stringBuilder.append("{}");
            }
            sb.append("请求内容:" + stringBuilder.toString());
        }
        LOGGER.info(String.valueOf(sb));
    }

    @AfterReturning(pointcut = "webLogProject()", returning = "ret")
    public void endRequest(Object ret) {
        LOGGER.info("请求结果：" + ret.toString());
        LOGGER.info("--------------------请求结束-------------------------");
    }

    @Around("webLogProject()")
    public Object serviceAOP(ProceedingJoinPoint pjp) throws Throwable {
        //1.开始
        try {
            // 2、执行时
            Object obj =  pjp.proceed();
//            LOGGER.info("请求结果：" + obj.toString());
//            LOGGER.info("--------------------请求结束-------------------------");
            return obj;
        } catch (Throwable e) {
            LOGGER.error(e.toString());// 3、发生异常时
            LOGGER.error("--------------------请求结束-------------------------");
            throw e;
        }
    }
}
```