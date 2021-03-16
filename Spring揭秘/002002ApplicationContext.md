ApplicationContext  
![ApplicationContext与BeanFactory](./Image/002/ApplicationContext.png)  
“青出于蓝而胜于蓝”：BeanFactory支持的所有功能之外，还进一步扩展了基本容器的功能，包括BeanFactoryPostProcessor、BeanPostProcessor以及其他特殊类型bean的自动识别、容器启动后bean实例的自动初始化、国际化的信息支持、容器内事件发布等  

![ApplicationContext的默认实现类](./Image/002/ApplicationContext的实现类.png)  


##　统一资源加载策略　　
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





















