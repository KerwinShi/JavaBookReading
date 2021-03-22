# Spring AOP
Spring AOP采用java作为AOP的实现语言AOL，对AOP的概念进行适当的抽象和实现，正式因为如此，学习起来比AspectJ要简单一些。  
### 实现机制
Spring AOP在2.0引入了AspectJ支持，但是实现还是基于动态代理机制和字节码生成技术实现，在运行期间为目标对象生成一个代理对象，实现将横切逻辑织入到这个代理对象。  
[动态代理](../Common/设计模式/代理模式.md)  

### Spring AOP的前世今生  
SpringAOP框架第一次发布-->  [一代目](./003002001SpringAOP1st.md)  -->  Spring 2.0(分解线)     -->  [二代目](./003002002SpringAOP2ed.md)  













