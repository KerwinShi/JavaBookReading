JDBC的API最佳实践  
### 基于Template的JDBC使用方式
以JdbcTemplate为核心的基于Template的JDBC使用方式  

首先，需要肯定的是JDBC是Java平台访问关系数据库的标准API，是整个Java平台面向关系数据库进行数据访问的基石。但是，JDBC的API过于面向较为底层的数据库操作，开发者需要写一堆按照API规矩规定的雷同的代码，不好好封装API就使用它访问数据库让人抓狂(1.每个开发者都要自己按着规矩重复写代码；2.每个人的开发风格和水平也是不一样的，代码有好有坏)；并且，数据访问异常也没有“将革命进行到底”，SQLException包括了所有异常，没有细分异常子类（虽然现在改了），此外，SQLException异常采用的ErrorCode是各个数据库提供商自己定义的，为了确定一个ErrorCode对应的异常，需要先确定数据库是什么数据库，才可以找到ErrorCode对应的含义。 

由于上述JDBC存在的不足之处，Spring出马了，定义了JdbcTemplate作为数据访问的helper类：  
1.用统一的格式和规范使用JDBC API，避免繁琐的易错的基于JDBC API的数据访问代码存在代码的各个角落，减少重复代码。    
2.对SQLException进行统一转译，将异常纳入Spring的异常层次体系，统一数据接口定义，简化客户端对数据异常的处理。  















### 基于操作对象的JDBC使用方式  
在JdbcTemplate基础上构建的基于操作对象的JDBC使用方式













