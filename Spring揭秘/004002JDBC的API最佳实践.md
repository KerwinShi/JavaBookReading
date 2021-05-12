JDBC的API最佳实践  
### 基于Template的JDBC使用方式
以JdbcTemplate为核心的基于Template的JDBC使用方式  

首先，需要肯定的是JDBC是Java平台访问关系数据库的标准API，是整个Java平台面向关系数据库进行数据访问的基石。但是，JDBC的API过于面向较为底层的数据库操作，开发者需要写一堆按照API规矩规定的雷同的代码，不好好封装API就使用它访问数据库让人抓狂(1.每个开发者都要自己按着规矩重复写代码；2.每个人的开发风格和水平也是不一样的，代码有好有坏)；并且，数据访问异常也没有“将革命进行到底”，SQLException包括了所有异常，没有细分异常子类（虽然现在改了），此外，SQLException异常采用的ErrorCode是各个数据库提供商自己定义的，为了确定一个ErrorCode对应的异常，需要先确定数据库是什么数据库，才可以找到ErrorCode对应的含义。 

由于上述JDBC存在的不足之处，Spring出马了，定义了JdbcTemplate作为数据访问的helper类：  
1.用统一的格式和规范使用JDBC API，避免繁琐的易错的基于JDBC API的数据访问代码存在代码的各个角落，减少重复代码。    
2.对SQLException进行统一转译，将异常纳入Spring的异常层次体系，统一数据接口定义，简化客户端对数据异常的处理。  

JdbcTemplate是通过[模板方法模式](../Common/设计模式/模板方法模式.md)对基于JDBC的数据访问代码进行统一封装。  

JDBC访问数据库的时候，就可以提取出这样一套流程
```java
// 取得数据库连接
con = getDaraSource().getConnection();
// 根据Connection创建Satetment或者PreparedSatetment
stmt = con.creatSatetment();
ps = con.prepareSatetment();
// 根据传入的sql语句或者参数，借助Satetment或者PreparedSatetment进行数据库操作，修改，查询等
ResultSet rs = stmt.executeQuery(sql);
// 关闭Satetment或者PreparedSatetment
stmt.close();
stmt=null;
// 捕获处理异常
catch(SQLException e)
// 关闭数据库连接
finally{con.close()}
```

根据模板方法模式，定义一个抽象类，将使用JDBC访问数据库的公共部分：异常处理和连接释放等进行统一管理，但是抽象类意味着我们使用的时候需要实现一个子类，这又是反人类的，所以Spring还引入了相应的CallBack接口`StatementCallBack`，这样，我们就不需要关心JDBC底层的东西了，只需要关心与数据库访问逻辑相关的东西就可以了。
```java
public interface StatementCallBack{
    Object doInStatement(Statement stmt);
}

// JdbcTemplate简单版源码
// JdbcTemplate根据相应的CallBack接口公开的API自由度的大小进行划分：  
// 面向Connection的模板方法：ConnectionCallBack  
// 面向Statement的模板方法：StatementCallBack  
// 面向PreparedStatement的模板方法：PreparedStatementCallback  
// 面向CallableStatement的模板方法：CallableStatementCallback 
public class JdbcTemplate extends JdbcAccessor implements JdbcOperations {
	// 面向Statement的模板方法：StatementCallBack
	public <T> T execute(StatementCallback<T> action) throws DataAccessException {
		Assert.notNull(action, "Callback object must not be null");
		Connection con = DataSourceUtils.getConnection(obtainDataSource());
		Statement stmt = null;
		try {
			stmt = con.createStatement();
			applyStatementSettings(stmt);
			T result = action.doInStatement(stmt);
			handleWarnings(stmt);
			return result;
		}
		catch (SQLException ex) {
			// Release Connection early, to avoid potential connection pool deadlock
			// in the case when the exception translator hasn't been initialized yet.
			String sql = getSql(action);
			JdbcUtils.closeStatement(stmt);
			stmt = null;
			DataSourceUtils.releaseConnection(con, getDataSource());
			con = null;
			throw translateException("StatementCallback", sql, ex);
		}
		finally {
			JdbcUtils.closeStatement(stmt);
			DataSourceUtils.releaseConnection(con, getDataSource());
		}
	}
}

// 使用示例
JdbcTemplate jdbcTemplate = ...;
// 数据库访问逻辑是我们要关心的
final String sql = "update ...";
StatementCallBack callBack = new StatementCallBack(){
    public Object doInStatement(Statement stmt){
        return new Integer(stmt.executeUpdate(sql));
    }
}
jdbcTemplate.execute(callBack);
```
![JdbcTemplate粗](./Image/004/JdbcTemplate粗.png) 
![JdbcTemplate](./Image/004/JdbcTemplate.png)   

JdbcOperations：定义了JDBC数据库操作集合  
JdbcAccessor：定义了一些公用的属性值  
    DataSource:JDBC的连接工厂  
    SQLExceptionTranslator：SQLException的转译  

 











### 基于操作对象的JDBC使用方式  
在JdbcTemplate基础上构建的基于操作对象的JDBC使用方式













