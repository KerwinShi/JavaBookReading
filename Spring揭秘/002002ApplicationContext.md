ApplicationContext  
![ApplicationContext与BeanFactory](./Image/002/ApplicationContext.png)  
“青出于蓝而胜于蓝”：BeanFactory支持的所有功能之外，还进一步扩展了基本容器的功能，包括BeanFactoryPostProcessor、BeanPostProcessor以及其他特殊类型bean的自动识别、容器启动后bean实例的自动初始化、国际化的信息支持、容器内事件发布等  

![ApplicationContext的默认实现类](./Image/002/ApplicationContext的实现类.png)  


##　一、统一资源加载策略　　
>URL全名是Uniform Resource Locator（统一资源定位器）　　
>But，说是同统一资源定义，其实只限于网络形式发布的资源的查找与定位，实际上，资源可以以任何形式存在，也可以存在任何场所；资源查找后返回的的结果没有统一的抽象。  
Spring提出了一套基于Resource和ResourceLoader接口的资源抽象和加载策略。  

![资源resource与资源定位resourceloader](./Image/002/资源resource与资源定位resourceloader.png)

### Resource
资源抽象，Resource接口可以根据资源的不同类型，或者资源所处的不同场合，给出相应的具体实现。　　
![Resource](./Image/002/Resource.png)
![Resource](./Image/002/Resource的实现类.png)

###　ResourceLoader
查找和定位资源，ResourceLoader接口是资源查找定位策略的统一抽象。  
![ResourceLoader](./Image/002/ResourceLoader-1.png)
![ResourceLoader](./Image/002/ResourceLoader-2.png)  

默认的实现类是`DefaultResourceLoader`，但是这个默认实现的处理逻辑最后`getResourceByPath(String)`返回的资源类型`Resource`的处理不统一，所以一般使用`FileSystemResourceLoader`，覆盖方法`getResourceByPath(String)`保证最后返回`Resource`为`FileSystemResource`。
```java
//DefaultResourceLoader的getResource()
    public Resource getResource(String location) {
        Assert.notNull(location, "Location must not be null");
        Iterator var2 = this.getProtocolResolvers().iterator();

        Resource resource;
        do {
            if (!var2.hasNext()) {
                if (location.startsWith("/")) {
                    return this.getResourceByPath(location);
                }

                if (location.startsWith("classpath:")) {
                    return new ClassPathResource(location.substring("classpath:".length()), this.getClassLoader());//classpath:为前缀的时候，返回ClassPathResource
                }

                try {
                    URL url = new URL(location);
                    return (Resource)(ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));//通过URL加载
                } catch (MalformedURLException var5) {
                    return this.getResourceByPath(location);//通过URL加载也失败了，就只能用最后的手段了
                }
            }

            ProtocolResolver protocolResolver = (ProtocolResolver)var2.next();
            resource = protocolResolver.resolve(location, this);
        } while(resource == null);

        return resource;
    }

    //构造一个不存在的Resource返回
    protected Resource getResourceByPath(String path) {
        return new DefaultResourceLoader.ClassPathContextResource(path, this.getClassLoader());
    }

    protected static class ClassPathContextResource extends ClassPathResource implements ContextResource {
        public ClassPathContextResource(String path, @Nullable ClassLoader classLoader) {
            super(path, classLoader);
        }

        public String getPathWithinContext() {
            return this.getPath();
        }

        public Resource createRelative(String relativePath) {
            String pathToUse = StringUtils.applyRelativePath(this.getPath(), relativePath);
            return new DefaultResourceLoader.ClassPathContextResource(pathToUse, this.getClassLoader());
        }
    }
```
为了可以批量的加载资源，而不是只能一个一个的加载，就有了继承的接口`ResourcePatternResolver`和对应的实现类`PathMatchingResourcePatternResolver`  
```java
public interface ResourcePatternResolver extends ResourceLoader {
    String CLASSPATH_ALL_URL_PREFIX = "classpath*:";//新的协议前缀
    Resource[] getResources(String var1) throws IOException;//获取多个Resource
}

public class PathMatchingResourcePatternResolver implements ResourcePatternResolver {
    private final ResourceLoader resourceLoader;
    public PathMatchingResourcePatternResolver() {
        this.resourceLoader = new DefaultResourceLoader();//默认内部有一个DefaultResourceLoader
    }
    public PathMatchingResourcePatternResolver(ResourceLoader resourceLoader) {//构造函数，可以传入指定的ResourceLoader，用它完成资源定位
        Assert.notNull(resourceLoader, "ResourceLoader must not be null");
        this.resourceLoader = resourceLoader;
    }
    //其他暂时忽略
}
```

### ApplicationContext与ResourceLoader  
ApplicationContext继承了ResourceLoader接口，本身就是一个ResourceLoader（ResourcePatternResolver），由于AbstractApplicationContext继承了DefaultResourceLoader，同时又有一个PathMatchingResourcePatternResolver（单独加载一个资源和同时加载多个资源的能力都有了，就问你强不强），因此，spring的ApplicationContext的实现类都可以做到资源统一加载。  
![AbstractApplicationContext](./Image/002/AbstractApplicationContext.png)  

1. 扮演ResourceLoader  

2. ResourceLoader类型的注入（bean需要依赖于ResourceLoader查找定位资源的时候，注入：还记得bean实例化过程的[Aware接口](./MindMap/002/IoC容器工作过程.drawio)吗？）  

3. Resource类型的注入
ApplicationContext启动伊始，会通过一个ResourceEditorRegistrar来注册Spring提供的针对Resource类型的PropertyEditor实现到容器中,即ResourceEditor（如果应用对象需要依赖一组Resource，通过CustomEditorConfigurar告知容器ResourceArrayPropertyEditor就可以）。  


4. 在特定情况下，ApplicationContext的Resource加载行为  

协议前缀，比如`file:`、`http:`、`ftp:`等，ResourceLoader中增加`classpath:`，ResourcePatternResolver增加`classpath*`(classpath*:如果能够在classpath中找到多个指定的资源,返回多个)

>ClassPathXmlApplicationContext:默认从classpath中加载bean定义配置文件  
>FileSystemXmlApplicationContext:默认会尝试从文件系统中加载  
>当容器实例化并启动完毕，我们要用相应容器作为ResourceLoader来加载其他资源时，各种ApplicationContext容器的实现类依然会有不同的表现。

FileSystemXmlApplicationContext都可以通过前缀强制指定，改变加载行为

##　二、国际化信息支持  
internationalization，取头取尾中间有18个字母，so，i18n就是这么来的  

###　Locale　　
不同的Locale代表不同的国家和地区，每个国家和地区在Locale这里都有相应的简写代码表示，包括语言代码以及国家代码（ISO标准代码）；
>Locale.CHINA代表中国地区  代码：zh_CN
>Locale.US代表美国地区 代码：en_US
>Locale.ENGLISH代表英语地区，代码只有语言代码：en
常用的Locale都提供有静态常量，不用我们自己重新构造。  
参考：https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html  

###　ResourceBundle　　
ResourceBundle用来保存特定于某个Locale的信息。
>通常，ResourceBundle管理一组信息序列，所有的信息序列有统一的一个basename，然后特定的Locale的信息，可以根据basename后追加的语言或者地区代码来区分　　
```properties
messages.properties 
messages_zh.properties 
messages_zh_CN.properties 
messages_en.properties 
messages_en_US.properties 
```
basename就是`messages`。  

>另外一种场景就是，每个资源文件中都有相同的键来标志具体资源条目，但每个资源内部对应相同键的资源条目内容，则根据Locale的不同而不同。  
```properties
# messages_zh_CN.properties文件中
menu.file=文件({0}) 
menu.edit=编辑
... 

# messages_en_US.properties文件中
menu.file=File({0}) 
menu.edit=Edit 
... 
```
这样，通过ResourceBundle的getBundle(String baseName, Locale locale)方法取得不同Locale对应的ResourceBundle，然后根据资源的键取得相应Locale的资源条目内容。

###　ApplicationContext与MessageSource  
Spring进一步抽象了国际化信息的访问接口，得到了MessageSource接口，于是ApplicationContext可以当做MessageSource使用（在默认情况下，ApplicationContext将委派容器中一个名称为messageSource，找不到实现，就默认实例化一个不含任何内容的StaticMessageSource实例）
```java
public interface MessageSource {
    String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);//根据传入的资源条目的键code,参数和Locale查找信息，没找到就返回传入的默认值
    String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;//相比上面的，没有默认值，抛出异常
    String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;//MessageSourceResolvable对资源条目的键，参数进行封装，没有找到就抛出异常
```
![MessageSource的实现类](./Image/002/MessageSource的实现类.png)   

1. StaticMessageSource   
简单实现，用于测试，不能用于生产

2. ResourceBundleMessageSource  
基于标准的ResourceBundle实现，最常用的、用于正式生产环境下的MessageSource实现。


3. ReloadableResourceBundleMessageSource  
cacheSeconds属性可以指定时间段，以定期刷新并检查底层的properties资源文件是否有变更，可以通过
ResourceLoader来加载信息资源文件（避免将信息资源文件放到classpath中）。  

以上三个实现类是可以独立于ApplicationContext运行的；
ApplicationContext启动的时候，自动识别容器中类型为MessageSourceAwaree的bean，并将自身作为MessageSource注入相应对象实例（实际上，可以通过将ApplicationContext中的messageSource注入到相应对象实例，和普通依赖注入就没有区别了）。

##　三、容器事件发布  

### 自定义事件发布  
EventObject和EventListener实现自定义事件发布    
![自定义事件发布组成](./Image/002/自定义事件发布组成.png)   
```java
//1.自定义事件类型
//针对方法执行事件的自定义事件类型定义，在合适的事件发布事件
public class MethodExecutionEvent extends EventObject { 
    private static final long serialVersionUID = -71960369269303337L; 
    private String methodName; 

    public MethodExecutionEvent(Object source) { 
        super(source); 
    } 
    public MethodExecutionEvent(Object source,String methodName) 
    { 
        super(source); 
        this.methodName = methodName; 
    } 
    public String getMethodName() { 
        return methodName; 
    } 
    public void setMethodName(String methodName) { 
        this.methodName = methodName; 
    } 
} 
//2.现针对自定义事件类的事件监听器接口，监听事件（我们的自定义事件监听器类只负责监听其对应的自定义事件并进行处理）
public interface MethodExecutionEventListener extends EventListener { 
    /** 
    * 处理方法开始执行的时候发布的MethodExecutionEvent事件
    */ 
    void onMethodBegin(MethodExecutionEvent evt); 
    /** 
    * 处理方法执行将结束时候发布的MethodExecutionEvent事件
    */ 
    void onMethodEnd(MethodExecutionEvent evt); 
}
//这里省略事件监听接口实现类，接口实际不干事的

//3.组合事件类和监听器，发布事件。
//事件发布者（EventPublisher），它本身作为事件源，会在合适的时点，将相应事件发布给对应的事件监听器
public class MethodExeuctionEventPublisher {
    private List<MethodExecutionEventListener> listeners = new ArrayList<MethodExecutionEventListener>();
    public void methodToMonitor(){
        MethodExecutionEvent event2Publish = new MethodExecutionEvent(this,"methodToMonitor");
        //具体时点上自定义事件的发布，方法开始
        publishEvent(MethodExecutionStatus.BEGIN,event2Publish);
        // 执行实际的方法逻辑
        // ...
        //具体时点上自定义事件的发布，方法即将结束
        publishEvent(MethodExecutionStatus.END,event2Publish);
    }

    protected void publishEvent(MethodExecutionStatus status, MethodExecutionEvent methodExecutionEvent) {
        List<MethodExecutionEventListener> copyListeners = new ArrayList<MethodExecutionEventListener>(listeners);
        for(MethodExecutionEventListener listener:copyListeners)
        {
            if(MethodExecutionStatus.BEGIN.equals(status))
                listener.onMethodBegin(methodExecutionEvent);
            else
                listener.onMethodEnd(methodExecutionEvent);
        }
    }

    //客户端可以根据情况决定是否需要注册或者移除某个事件监听器,避免一直被引用
    public void addMethodExecutionEventListener(MethodExecutionEventListener listener)
    {
        this.listeners.add(listener);
    }
    public void removeListener(MethodExecutionEventListener listener)
    {
        if(this.listeners.contains(listener))
            this.listeners.remove(listener);
    }
    public void removeAllListeners()
    {
        this.listeners.clear();
    }

    //psvm......
}
```

### ApplicationContext与事件   
对应上面的自定义事件发布过程，  ApplicationEvent为自定义事件类型，实现类有：默认有ContextClosedEvent（容器关闭）、ContextRefreshedEvent（容器初始化或者刷新）和RequestHandledEvent（Web请求处理后）三个实现；ApplicationListener为自定义事件监听器接口；ApplicationContext继承了ApplicationEventPublisher接口，扮演事件发布者的角色，提供了void publishEvent(ApplicationEvent event)方法定义。  

>ApplicationContext的事件的发布和事件监听器的注册没有自己做，而是交给了ApplicationEventMulticaster接口。容器启动的时候检查容器内是否存在名称为applicationEventMulticaster的实例对象，有就使用，否则实例化一个默认的SimpleApplicationEventMulticaster。
![ApplicationContext的事件发布](./Image/002/ApplicationContext的事件发布.png)  

```java
//ApplicationContext版本的事件发布
//自定义事件类型
public class MethodExecutionEvent extends ApplicationEvent {
    private static final long serialVersionUID = -71960369269303337L;
    private String methodName;
    private MethodExecutionStatus methodExecutionStatus;
    public MethodExecutionEvent(Object source) {
        super(source);
    }
    public MethodExecutionEvent(Object source,String methodName,MethodExecutionStatus methodExecutionStatus)
    {
        super(source);
        this.methodName = methodName;
        this.methodExecutionStatus = methodExecutionStatus;
    }
//get set方法
}

//事件监听器
public class MethodExecutionEventListener implements ApplicationListener {
    public void onApplicationEvent(ApplicationEvent evt) {
        if(evt instanceof MethodExecutionEvent)
        {
            // 执行处理逻辑
        }
    }
}

//发布事件
public class MethodExeuctionEventPublisher implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher eventPublisher;//ApplicationEventPublisherAware会自动注入一个eventPublisher，即本身
    public void methodToMonitor()
    {
        MethodExecutionEvent beginEvt = new MethodExecutionEvent(this,"methodToMonitor",MethodExecutionStatus.BEGIN);
        this.eventPublisher.publishEvent(beginEvt);//具体的发布逻辑不用自己再写了，交给注入的eventPublisher吧
        // 执行实际方法逻辑
        // ...
        MethodExecutionEvent endEvt = new  MethodExecutionEvent(this,"methodToMonitor",MethodExecutionStatus.END);
        this.eventPublisher.publishEvent(endEvt);
    }
    public void setApplicationEventPublisher(ApplicationEventPublisher appCtx) {
        this.eventPublisher = appCtx;
    }
}
```

##　四、多配置模块加载（更方便的加载多个配置文件）  
让容器同时读入划分到不同配置文件的信息，ApplicationContext更方便  
```java
String[] locations = new String[]{ "conf/dao-tier.springxml","conf/view-tier.springxml", "conf/business-tier.springxml"};
ApplicationContext container = new FileSystemXmlApplicationContext(locations);
// 或者
ApplicationContext container = new ClassPathXmlApplicationContext(locations);
//...
//甚至于使用通配符
ApplicationContext container = new FileSystemXmlApplicationContext("conf/**/*.springxml");


//ClassPathXmlApplicationContext可以通过某一个类在Classpath中的位置定位配置文件，而不用指定每个配置文件的完整路径名
//com/
//  foo/
//      services.xml
//      daos.xml
//      MessengerService.class
ApplicationContext ctx = new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```  
BeanFactory需要分多次加载，或者通过主配置文件`<import>`其他配置文件

































