## java平台web开发历程  
1. Servlet  
是java平台上第一个用于web开发的技术，运行于web容器内，提供了session和对象生命周期管理等功能，最主要的是他是java类，可以直接访问Java平台的各种服务，使用相应的api。  
>这东西是好，但是总是要说但是。。。  
但是，Magic Servlet的存在，并不意味着一个web应用程序只存在一个Servlet实现，实际上，更多时候是一个Servlet处理一个web请求；并且html这些渲染工作也都混在java代码里面，不好找，美工，前台开发后台开发都要维护这一份写死在java里面的代码。  

2. JSP  
>为了把Servlet中的视图渲染的逻辑抽取出来，使用模板化的方法，jsp的提出，成为java平台开发web应用程序事实上的模板化视图标准  

jsp把Servlet中通过out.println语句输出的视图渲染逻辑抽取到.jsp模板文件  

jsp最终是编译为Servlet来运行的，可以在jsp中编写java代码；而且，使用Servlet处理web请求，需要在web.xml文件中配置URL与Servlet之间的映射关系。  

>但是，又是但是  

但是，由于jsp可以直接当做servlet使用，开始不满足于作为视图模板使用，野心膨胀了，抢走了原本属于Servlet的活，代替Servlet。这，这不回到起点了么。  

3. Servlet与JSP的联盟  
回到JSP设计的最初起点，Servlet负责请求处理流程的控制以及与业务层的交互，JSP负责页面开发或者表现层，并且引入JavaBean，得到JSP Model2  

4. MVC(Model-View-Controller)  
Controller：负责接收视图发送的请求并处理，根据请求条件通知模型进行应用程序状态更新，之后选择合适的视图显示给用户。  
Model：封装应用的逻辑以及数据状态，收到控制器通知执行相应的逻辑，完成后，通过事件机制返回给视图，显示最新数据状态。  
View：面向用户的接口，通过视图发起请求，给到控制器进行处理，最终接受模型的状态更新通知，更新自身的展示  
![MVC结构示意图](./Image/006/MVC结构示意图.png)  























