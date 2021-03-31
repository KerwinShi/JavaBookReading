# ProxyFactory背后的原理  
知其然，也要知其所以然  

```java
public interface AopProxy{
    Object getProxy();
    Object getProxy(ClassLoader calssLoader);
}
```
AopProxy对不同的代理实现提供了相应的子类实现
![AopProxy](./Image/003/AopProxy.png)  

AopProxy实例通过抽象工厂模式创建，通过AopProxyFactory接口完成  
```java
public interface AopProxyFactory{
    AopProxy creatAopProxy(AdvisedSupport config) throws AopConfigException;//根据AdvisedSupport确定生成什么类型的AopProxy，实际交给DefaultAopProxyFactory实现
}
```
![ ](./Image/003/AopProxyFactory.png)

那么现在，关注点落在了AdvisedSupport上，他是何方神圣呢？  
接下来，就让我们揭开她的神秘面纱  
![AdvisedSupport](./Image/003/AdvisedSupport.png)
AdvisedSupport其实包含了两部分内容：  
一部分就是用来记载生成代理对象的控制信息（ProxyConfig分支）；另一部分就是承载生成代理对象所需的必要信息（Advised分支，其实就是目标类、Advice、Advisor等）。  

- ProxyConfig（五个boolean属性）  
```java
//true则强制使用CGLIB代理
private boolean proxyTargetClass = false;
//告知代理对象是否需要采取进一步优化措施（即使为代理对象添加或移除了相应的Advice也会被忽略），true则强制使用CGLIB代理
private boolean optimize = false;
//是否可以强制转换为Advised，可以通过Advised查询代理对象的一些信息
boolean opaque = false;
//生成代理对象的时候，绑定到ThreadLocal，目标对象需要访问代理对象可以通过AopContext.currentProxy取得
boolean exposeProxy = false;
//true表示一旦代理对象的相关配置完成，不容许再修改
private boolean frozen = false;
```
- Advised  
要针对哪些目标类生成代理对象，要为代理对象加入何种横切逻辑，都可以通过Advised社设置查询，比如可以使用Advised访问代理对象持有的Advisor，实现添加、删除等动作。

ProxyFactory同时集成了AdvisedSupport（用来设置生成的代理对象的相关信息）和AopProxy（用来取得最终的代理对象）  
![ProxyFactory](./Image/003/ProxyFactory.png)









