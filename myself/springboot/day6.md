## AOP

Spring框架的第二大核心,面向切面编程,即面向特定方法的编程

### 使用

@Aspect:类注解,标记当前类为切面类

@Around:方法注解,标明切入点

```java
@Component
@Aspect //当前类为切面类
@Slf4j
public class TimeAspect {

    @Around("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
    public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {
        //记录方法执行开始时间
        long begin = System.currentTimeMillis();
        //执行原始方法
        Object result = pjp.proceed();
        //记录方法执行结束时间
        long end = System.currentTimeMillis();
        //计算方法执行耗时
        log.info("方法执行耗时: {}毫秒",end-begin);
        return result;
    }
}
```



### 核心概念:

- 连接点JoinPoint:指可以被AOP控制的方法,其中封装了连接点方法在执行时的相关信息

- 通知Advice:指重复的逻辑,即方法间的共性功能


- 切入点PointCut:匹配连接的条件,通知仅会在切入点方法执行时被应用


- 切面Aspect:描述通知与切入点的对应关系(通知+切入点),可以描述当前切面针对哪个原始方法,执行什么操作,切面所在的类称为切面类


- 目标对象Target:通知所应用的对象

底层基于动态代理技术实现,即为目标对象生成一个代理对象,代理对象中会有对应的增强

### 通知类型

- @Around:环绕通知
- @Before:前置通知
- @After:后置通知
- @AfterReturing:返回后通知,目标方法出现异常后不执行
- @AfterThrowing:异常后通知,目标方法出现异常后执行

### 通知顺序

- 不同切面类中,其通知顺序与类名的字母排序顺序有关
- @Order注解可控制不同切面类的通知顺序,其参数值的**相对大小**越小越先执行(即执行越靠近外层)

### 切入点表达式

#### execution

```java
execution(访问修饰符?  返回值  包名.类名.?方法名(方法参数) throws 异常?)
    //?标记可省略,包名不建议省略
    //异常指声明方法上抛出的异常
```

通配符:

- *:可用于标注**返回值,包名,类名,方法名,参数类型**表示任意
- ..:表示任意个数,用于配置参数,或包名,表示任意一级

逻辑符:

- &&,||,!:用于复杂的切入点

#### @annotation

- 自定义注解:

  ```java
  @Target(ElementType.METHOD) // 修饰注解的注解,参数指明该注解用于修饰方法
  @Retention(RetentionPolicy.RUNTIME) // 修饰注解的注解,用于指明目标注解何时生效,该参数指明运行时生效
  public @interface LogOperation{
  }
  // 该注解不需要任何属性,仅起到标识作用
  //此后@LogOperation标注目标方法
  //@Before("@annotation(com.itheima.anno.LogOperation)")标注通知
  ```

  