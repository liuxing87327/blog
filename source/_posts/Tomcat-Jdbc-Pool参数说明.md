title: "Tomcat-Jdbc-Pool参数说明"
date: 2016-01-05 11:36:00
category: Java
tags: [连接池, 数据源, Jdbc]

---

##介绍
Tomcat 在 7.0 以前的版本都是使用commons-dbcp做为连接池的实现，但是DBCP存在一些问题： 
- DBCP 是单线程的，为了保证线程安全会锁整个连接池 
- DBCP 性能不佳
- DBCP 太复杂，超过60个类，发展滞后。 

因此，通常J2EE中还会使用其它的高性能连接池，如C3P0，还有阿里系的druid等。
为此，Tomcat 从 7.0 开始引入一个新的模块： Tomcat Jdbc Pool 
Tomcat Jdbc Pool 近乎兼容 DBCP，性能更高 
- 异步方式获取连接 
- Tomcat Jdbc Pool是Tomcat的一个模块，基于Tomcat-Juli，使用Tomcat的日志框架 
- 使用 javax.sql.PooledConnection 接口获取连接 
- 支持高并发应用环境 
- 超简单，核心文件只有8个，比 c3p0 还少 
- 更好的空闲连接处理机制 
- 支持 JMX 
- 支持 XA Connection。 
- Tomcat Jdbc Pool 可在 Tomcat 中直接使用，也可以在独立的应用中使用。 


##属性

通用属性

| 属性名      | 描述(DBCP/Tomcat jdbc-pool 差别 ) |   DBCP默认值   | jdbc-pool默认值 |
| :----- | :--------| :------ | :------ |
|  username  |  用户名  |  -  |  -   |
|  password  |  密码  |  -  |   -  |
|  url  |  建立连接的URL  |  -  |  -   |
|  driverClassName  |  驱动的完整有效的java类名  |  -  |  -   |
|  connectionProperties  |  (String) 当建立新连接时被发送给JDBC 驱动的连接参数，格式必须是 [propertyName=property;]<br>*注意 ：参数user/password 将被明确传递，所以不需要包括在这里。  |  -  |   -  |
|  defaultAutoCommit  |  (boolean) 连接池创建的连接的默认的auto-commit 状态  |  true  |   driver default  |
|  defaultReadOnly  |  (boolean) 连接池创建的连接的默认的read-only 状态。 如果没有设置则setReadOnly 方法将不会被调用。 ( 某些驱动不支持只读模式， 比如：Informix)  |  driver default  |  driver default   |
|  defaultTransactionIsolation  |  (String) 连接池创建的连接的默认的TransactionIsolation 状态。 下面列表当中的某一个： ( 参考javadoc) <br>NONE <br>READ_COMMITTED <br>READ_UNCOMMITTED <br>REPEATABLE_READ <br>SERIALIZABLE  |  driver default  |  driver default   |
|  defaultCatalog  |  	(String) 连接池创建的连接的默认的catalog	  |  -  |  driver default   |
|  initialSize  |  (int) 初始化连接： 连接池启动时创建的初始化连接数量，1。2 版本后支持  |  0  |  10   |
|  maxActive  |  (int) 最大活动连接： 连接池在同一时间能够分配的最大活动连接的数量， 如果设置为非正数则表示不限制  |  8  |  100   |
|  maxIdle  |  (int) 最大空闲连接： 连接池中容许保持空闲状态的最大连接数量， 超过的空闲连接将被释放， 如果设置为负数表示不限制 <br>`如果启用，将定期检查限制连接，如果空闲时间超过minEvictableIdleTimeMillis 则释放连接 （ 参考testWhileIdle ）`  |  8  |  与maxActive相同   |
|  minIdle  |  (int) 最小空闲连接： 连接池中容许保持空闲状态的最小连接数量， 低于这个数量将创建新的连接， 如果设置为0 则不创建 <br>`如果连接验证失败将缩小这个值（ 参考testWhileIdle ）` |  0  |  与initialSize 相同   |
|  maxWait  |  (int) 最大等待时间： 当没有可用连接时， 连接池等待连接被归还的最大时间( 以毫秒计数)， 超过时间则抛出异常， 如果设置为-1 表示无限等待  |  无限  |  30000（30秒）   |
|  validationQuery  |  (String) SQL 查询， 用来验证从连接池取出的连接， 在将连接返回给调用者之前。 如果指定， 则查询必须是一个SQL SELECT 并且必须返回至少一行记录 <br> `查询不必返回记录，但这样将不能抛出SQL异常` |  -  |  -   |
|  testOnBorrow  |  (boolean) 指明是否在从池中取出连接前进行检验， 如果检验失败， 则从池中去除连接并尝试取出另一个。注意： 设置为true 后如果要生效，validationQuery 参数必须设置为非空字符串 <br> `参考validationInterval以获得更有效的验证`  |  true  |  false   |
|  testOnReturn  |  (boolean) 指明是否在归还到池中前进行检验 注意： 设置为true 后如果要生效，validationQuery 参数必须设置为非空字符串  |  false  |  false   |
|  testWhileIdle  | (boolean) 指明连接是否被空闲连接回收器( 如果有) 进行检验。 如果检测失败， 则连接将被从池中去除。注意： 设置为true 后如果要生效，validationQuery 参数必须设置为非空字符串   |  false  |  false   |
|  timeBetweenEvictionRunsMillis  |  (int) 在空闲连接回收器线程运行期间休眠的时间值， 以毫秒为单位。 如果设置为非正数， 则不运行空闲连接回收器线程 <br> `这个值不应该小于1秒，它决定线程多久验证连接或丢弃连接`  |  -1  |  5000（5秒）   |
|  numTestsPerEvictionRun  |  (int) 在每次空闲连接回收器线程( 如果有) 运行时检查的连接数量  |  false  |  未使用   |
|  minEvictableIdleTimeMillis  |  连接在池中保持空闲而不被空闲连接回收器线程( 如果有) 回收的最小时间值，单位毫秒  |  1000 * 60 * 30（30分钟）  |  60000（60秒）   |
|  poolPreparedStatements  |  (boolean) 开启池的prepared statement 池功能  |  false  |  未使用   |
|  maxOpenPreparedStatements  |  (int)statement 池能够同时分配的打开的statements 的最大数量， 如果设置为0 表示不限制  |  不限制  |  未使用  |
|  accessToUnderlyingConnectionAllowed  |  (boolean) 控制PoolGuard 是否容许获取底层连接 <br> `jdbc-pool中未使用此属性；可以通过调用连接的unwrap方法取得控制权。参考javax。sql。DataSource接口，通过反射调用getConnection方法，或强制转换为javax。sql。PooledConnection对象。`|  false  |  未使用   |
|  removeAbandoned  |  (boolean) 标记是否删除泄露的连接， 如果他们超过了removeAbandonedTimout 的限制。 如果设置为true， 连接被认为是被泄露并且可以被删除， 如果空闲时间超过removeAbandonedTimeout。 设置为true 可以为写法糟糕的没有关闭连接的程序修复数据库连接。<br> `参考logAbandoned`  |  false  |  false   |
|  removeAbandonedTimeout  |  (int) 泄露的连接可以被删除的超时值， 单位秒 <br> 应设置为应用中查询执行最长的时间  |  300  |  60   |
|  logAbandoned  |  (boolean) 标记当Statement 或连接被泄露时是否打印程序的stack traces 日志。被泄露的Statements 和连接的日志添加在每个连接打开或者生成新的Statement， 因为需要生成stack trace 。  |  false  |  false   |


----

Tomcat jdbc-pool 新增属性

| 属性名      | 描述     | 默认值 |
| :--------  | :------ | :------ |
| validatorClassName  |  (String)实现org.apache.tomcat.jdbc.pool.Validator接口的类名，必须存在默认或明确的无参构造方 法。将建立一个指定类的实例作为验证器，用来代替执行查询的连接验证。例如：com.mycompany.project.SimpleValidator。  |  -   |
| initSQL  |  (String) 当连接第一次建立时执行的SQL  |  -   |
| jdbcInterceptors  |  (String)（jdbc拦截器——jdbc-pool的高级扩展属 性）用分号分隔的、继承org.apache.tomcat.jdbc.pool.JdbcInterceptor的类名列表。这些拦截器将被插入到对 java.sql.Connection操作之前的拦截器链上。<br>预制的拦截器有：<br>org.apache.tomcat.jdbc.pool.interceptor.ConnectionState - 追踪自动提交、只读状态、catalog和事务隔离等级等状态。（keeps track of auto commit， read only， catalog and transaction isolation level.）org.apache.tomcat.jdbc.pool.interceptor.StatementFinalizer - 追踪打开的statement，当连接被归还时关闭它们。（keeps track of opened statements， and closes them when the connection is returned to the pool.）<br>更多预制拦截器详细描述请参见JDBC拦截器部分  |  -   |
| validationInterval  |  (long) 避免过度验证，保证验证不超过这个频率——以毫秒为单位。如果一个连接应该被验证，但上次验证未达到指定间隔，将不再次验证。  |  30000（30秒）   |
| jmxEnabled  |  (boolean) 是否将连接注册到JMX |  true   |
| fairQueue  |   (boolean) 如果被设为true ，getConnection 方法将被以先进先出的方式对待。此属性使用 org.apache.tomcat.jdbc.pool.FairBlockingQueue 实现闲置连接列表。<br>如果需要使用异步连接回收，这个标记是必须的。<br>这个标记确保线程取得连接的顺序和他们调用getConnection 方法的顺序是相同的。<br>在性能测试中，这个标记对锁和锁等待有非常大的影响。当fairQueue=true ，将有一个依赖于操作系统的线程作为决定线程。如果是Linux 系统（ 系统属性os.name=Linux ）可以在线程池的类加载之前设置系统属性 org.apache.tomcat.jdbc.pool.FairBlockingQueue.ignoreOS=true 关闭Linux 特定行为但仍然使用公平队列  |  true   |
| abandonWhenPercentageFull  |  (int)正在被使用的连接超过这个百分比以前被丢弃的连接不会被断开或报告。这个值应被设为0-100之间。默认值为0，意味着达到 removeAbandonedTimeout 时将被尽快关闭。  |  0   |
| maxAge  |  (long)保持连接的最大毫秒数。当一个连接被归还时，连接池将检查是否满足：现在时间-连接时长>maxAge，如果条件满足，连接将被关闭而不是回到池中。默认值为0，标识禁用该功能。  |  0   |
| useEquals  |  (boolean)如果希望ProxyConnection类使用String.equals方法对比方法名，设为true；否则将使用==判断。这个属性不会影响单独配置的拦截器。	  |  true   |
| suspectTimeout  |  (int)以秒为单位的超时时间（怀疑超时）。类似 removeAbandonedTimeout，但不会将连接丢弃甚至关闭，如果logAbandoned为true，则 只是记录一个警告。如果这个值小于等于0，不会有怀疑超时检测被执行。怀疑检测只有当超时时间大于0并且连接未丢弃，或者丢弃检测被禁用的情况下才占用空间。如果一个连接被怀疑，将记录一条警告消息，并发送一个JMX通知。  |  0   |
| rollbackOnReturn  |   (boolean)如果autoCommit==false，当连接被归还时，通过调用连接的rollback方法中断事务。  |  false  |
| commitOnReturn  |  (boolean)如果autoCommit==false，当连接被归还时，通过调用连接的commit方法完成事务。如果rollbackOnReturn==ture，这个属性将被忽略。  |  false   |
| alternateUsernameAllowed  |  (boolean)为了提高性能，默认情况下，jdbc-pool将忽略 DataSource.getConnection(username,password)调用，直接返回一个以已有的全局配置的用户名和密码创建的连 接。连接池仍然可以用不同的用户名和密码，但已经通过旧的用户名和密码创建的连接将被关闭，然后重新以新的用户名和密码连接。这样连接池将以全局级别管理 连接数，而不是schema级别。设置这个属性为true来启用 DataSource.getConnection(username,password)方法描述的行为。<br> 这个属性为bug 50025 增加。  |   false  |
| dataSource  |   (javax.sql.DataSource)向连接池注入一个数据源，连接池将使用这个数据源索取连接，而不是通过java.sql.Driver接口建立。当您希望池化XA连接或者使用数据源而不是url时，这个属性非常有用。  |  -   |
| dataSourceJNDI  |  (String)用来建立数据连接的JNDI名称。参考dataSource属性。  |  -   |
| useDisposableConnectionFacade  |  (boolean)如果您希望在连接上建立一道屏障防止连接关闭之后被重新使用，设置这个属性为true。这个属性用来预防线程保持已关闭连接的引用，并在上面执行查询动作。  |  true   |
| logValidationErrors  |  (boolean)如果设置为true，将在验证相位时向日志文件写入错误。如果值为true，错误将被记录为SEVER。默认值是false以向后兼容。  |  false   |
| propagateInterruptState  |  (boolean)设置这个属性为true，可以传播一个被中断的线程（还没有清除中断状态）的中断状态。默认值为false以向后兼容。  |   false  |

##样例

```xml
<bean id="parentDataSource" abstract="true" class="org.apache.tomcat.jdbc.pool.DataSource" destroy-method="close"
          p:maxWait="5000"
          p:removeAbandoned="true"
          p:removeAbandonedTimeout="180"
          p:connectionProperties="clientEncoding=UTF-8"
          p:validationQuery="SELECT 1"
          p:validationQueryTimeout="1"
          p:validationInterval="30000"
          p:testOnBorrow="false"
          p:testOnReturn="false"
          p:testWhileIdle="true"
          p:timeBetweenEvictionRunsMillis="10000"
          p:minEvictableIdleTimeMillis="60000"
          p:numTestsPerEvictionRun="20"
          p:logAbandoned="false"
          p:jmxEnabled="true"
          p:defaultAutoCommit="true"/>
          
<bean id="dataSource" parent="parentDataSource"
          p:driverClassName="com.microsoft.sqlserver.jdbc.SQLServerDriver"
          p:url="jdbc:sqlserver://127.0.0.1:1433;DatabaseName=test;sendStringParametersAsUnicode=false"
          p:username="test"
          p:password="test"
          p:initialSize="100"
          p:maxActive="500"
          p:maxIdle="100"
          p:minIdle="50" />
```