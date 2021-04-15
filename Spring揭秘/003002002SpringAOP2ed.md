Spring AOP 二代  
一代目的重点的是了解各种概念和实现原理，这些都是框架的基础，无论如何升级框架，这些东西都是不会变的。  

###　AspectJ形式的Spring AOP
主要升级：  
- 使用POJO声明AspectJ和相关的Advice  
- 获得了新的Pointcut表述方式  

>Spring AOP还是那个Spring AOP，集成AspectJ只是穿了它的大衣。  

使用实例：   
#### 1.Aspect定义 与 目标对象
```java
//Aspect定义
@Aspect
public class PerformanceTraceAspect{
    //Pointcut定义
    @Pointcut("excution(public void *.method1())")
    public void pointcutName(){}

    //Advice定义
    @Around("pointcutName()")
    public Object performanceTrace(ProceedingJointPoint jiontPoint) throws Throwable{
        StopWatch watch = new StopWatch();
        try{
            watch.start();
            return jiontPoint.proceed();
        }finally{
            watch.stop();
        }
    }
}

public class Foo{
    public void method1（）{
        System.out.prinln("lalalal");
    }
}
```
2.将Aspect织入目标对象  
```java
//ProxyFactory
AspectJProxyFactory weaver  = new AspectJProxyFactory();
weaver.setProxyTargetClass(true);
weaver.setTarget(new Foo());
weaver.addAspect(PerformanceTraceAspect.class);
Object proxy = weaver.getProxy();
proxy.method1();


//AutoProxyCreator
//IoC注册AnnotationAwareAspectJAutoProxyCreator，然后自动搜索IoC容器中的Aspect，并用到Pointcut定义的目标对象上。
```

#### 2.@AspectJ形式的Pointcut
2.0之前，指定方法名和正则表达式两种方式就可以达到目的  
2.0之后，集成了AspectJ的Pointcut描述语言支持  
- 声明方式
依附在@Aspect标注的Aspect定义类之内，通过@Pointcut注解指定AspectJ形式的Pointcut表达式，然后注解标注到Aspect定义类的某个方法上即可。  
```java
//Aspect定义
@Aspect
public class PerformanceTraceAspect{
    //Pointcut Expression：规定Pointcut匹配规则的地方，包括Pontcut标志符和表达式匹配模式  
    @Pointcut("excution(public void *.method1())")
    public void pointcutName(){}//这个方法就是Pointcut Signature，返回值必须是void，public类型的可以在其他Aspect定义中引用，private则只能在当前的Aspect使用，避免重复Pointcut Expression的定义
    @Pointcut("pointcutName()")//Pointcut Signature的作用，当做Pointcut Expression标志符，引用第一个Pointcut的定义，还可以进一步的做逻辑运算&& || ！
    public void pointcutName1() {}

    //技巧：可以在一个Aspect中定义，管理所有的Pointcut
    //在其他的Aspect定义时，使用这些Pointcut
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)//方法级注解，不能脱离方法存在
public @interface Pointcut {
    String value() default "";
    String argNames() default "";
}
```
- @AspectJ形式的Pointcut表达式的标志符  
相比于@AspectJ，有所限制，被限制在方法级别的  
    - execution  
    匹配拥有指定方法签名的Jointpoint，规定格式如下：
    `execution(modifiers? ret-type-pattern declaring-type-pattern? name-parttern(param-pattern) throws-pattern?)`，方法的返回类型，方法名称以及参数部分必须指定，其他可以省略。`execution(public void Foo.doSomething(String))`和`execution(void doSomething(String))`都是可以的。  
    `*`可以用于任何部分的匹配模式中，可以匹配相邻的多个字符`execution(* *(*))`代表匹配一个参数的方法  
    `..`可以在declaring-type-pattern和参数位置使用`execution(void cn.springtest..*.doSomething(String))`匹配cn.springtest包下的所有类型，以及cn.springtest的子包中的所有类型；`void doSomething(..)`匹配有0到多个参数的方法，参数类型不限
    `*`和`..`还可以组合使用，更加强大。
    - within










