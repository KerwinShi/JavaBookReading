# Spring AOP
Spring AOP采用java作为AOP的实现语言AOL，对AOP的概念进行适当的抽象和实现，正式因为如此，学习起来比AspectJ要简单一些。  
### 实现机制
Spring AOP在2.0引入了AspectJ支持，但是实现还是基于动态代理机制和字节码生成技术实现，在运行期间为目标对象生成一个代理对象，实现将横切逻辑织入到这个代理对象。  
[动态代理](../Common/设计模式/代理模式.md)  

### Spring AOP的前世今生  
SpringAOP框架第一次发布-->  [一代目](./003002001SpringAOP1st.md)  -->  Spring 2.0(分解线)     -->  [二代目](./003002002SpringAOP2ed.md)  

### 最佳实践  
1. 异常处理  
2. 安全检查
3. 缓存

### 特殊问题  
目标对象中方法之间存在调用
```java
public class testObj{
    // 方法1
    public void method1(){
        method2();//调用方法2
    }
    //方法2
    public void method2(){
        
    }
}

public class testAspect{
    @Poincut("execution(public void *.method1())")
    public void method1(){}
    @Poincut("execution(public void *.method2())")
    public void method2(){}

    @Poincut("method1()||method2()")
    public void compositePointcut(){}

    @Around("compositePointcut()")
    public Object performanceTrace(ProceedingJointPoint pjp)throws Throwable{
        // 。。。
    }
}
```
正常情况下，当我们调用method1(),method2()的时候，都会被拦截。当我们直接调用method2()的时候，会被拦截；但是当我们调用method1的时候，会发现method1()调用的method2()并没有被拦截，这是为什么呢？  
>原因：Spring AOP是采用代理实现AOP的，横切逻辑是被添加到动态生成的代理对象上的，只要调用的是代理对象的方法就可以保证目标对象上的方法是被拦截的。上面，代理对象的method1()方法最终会调用目标对象的mehtod1()，但是method1()里面调用的是目标对象的method2()，而不是代理对象的method2()，所以就不会被拦截。
```java
// 解决之道，方法还有更多，但是基本的思路都是这
public void method1(){
    (testObjAopContext.getCurrentProxy()).method2();//获取当前目标对象的代理对象，调用代理对象的方法2
}
```








