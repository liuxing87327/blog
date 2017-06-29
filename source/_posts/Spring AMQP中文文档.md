title: "Spring AMQP中文文档"
date: 2017-06-30 02:59:00
category: Java
tags: [AMQP,rabbitmq]

---

## 1.前言
Spring AMQP项目是用于开发AMQP的解决方案。 我们提供一个“模板”作为发送和接收消息的抽象。我们还为普通POJO进行提供消息处理支持。这些库促进AMQP资源的管理，同时支持使用依赖注入和声明式配置。 在所有情况下，您将看到与Spring Framework中的JMS支持的相似之处。有关其他项目相关信息，请访问Spring AMQP[项目主页](http://projects.spring.io/spring-amqp/)。

## 2.介绍
该帮助文档的第一部分是Spring AMQP以及基本概念和一些代码段的概述，可以尽快帮助您快速使用。

### 2.1 快速入门
#### 2.1.1 介绍
五分钟快速使用Spring AMQP.

先决条件：安装并运行[RabbitMQ](http://www.rabbitmq.com/download.html)。然后在您的项目中加入如下MAVEN依赖:

```xml
<dependency>
  <groupId>org.springframework.amqp</groupId>
  <artifactId>spring-rabbit</artifactId>
  <version>1.7.2.RELEASE</version>
</dependency>
```

##### 兼容性
虽然默认的`Spring Framework`版本依赖关系为4.3.x，但`Spring AMQP`通常与早期版本的`Spring Framework`兼容。
基于注解的监听器和`RabbitMessagingTemplate`需要`Spring Framework 4.1`或更高版本。

最低的`amqp-client`的java客户端库版本是4.0.0。

注意这是指java客户端库;
一般来说，它将适用于较旧的代理版本。

##### 非常非常快
使用简单同步的Java发送和接收消息：
```java
ConnectionFactory connectionFactory = new CachingConnectionFactory();
AmqpAdmin admin = new RabbitAdmin(connectionFactory);
admin.declareQueue(new Queue("myqueue"));
AmqpTemplate template = new RabbitTemplate(connectionFactory);
template.convertAndSend("myqueue", "foo");
String foo = (String) template.receiveAndConvert("myqueue");
```

请注意，Java Rabbit客户端中也有一个`ConnectionFactory`。
我们在上面的代码中使用了Spring抽象的`ConnectionFactory`。
我们使用Rabbit的默认`exchange`(因为发送中没有指定)，并且所有队列默认绑定到默认`exchange`(因此我们可以在发送中使用队列名称作为`routing key`)。
这些行为在AMQP规范中定义。

##### 使用XML配置
上述例子在XML中的配置形式如下

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/rabbit
           http://www.springframework.org/schema/rabbit/spring-rabbit.xsd
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <rabbit:connection-factory id="connectionFactory"/>
    <rabbit:template id="amqpTemplate" connection-factory="connectionFactory"/>
    <rabbit:admin connection-factory="connectionFactory"/>
    <rabbit:queue name="myqueue"/>

</beans>
```

```java
ApplicationContext context =
    new GenericXmlApplicationContext("classpath:/rabbit-context.xml");
AmqpTemplate template = context.getBean(AmqpTemplate.class);
template.convertAndSend("myqueue", "foo");
String foo = (String) template.receiveAndConvert("myqueue");
```

默认情况下，`<rabbit:admin />`声明会自动查找类型为`Queue`，`Exchange`和`Binding`的bean，并将他们绑定，因此不需要在简单的Java程序中明确使用该bean。在XML模式中配置组件的属性有很多选项 - 您可以使用XML编辑器的自动完成功能来浏览它们并查看其文档。

##### 使用java配置
相同的代码在java代码中的另一种配置
```java
ApplicationContext context =
    new AnnotationConfigApplicationContext(RabbitConfiguration.class);
AmqpTemplate template = context.getBean(AmqpTemplate.class);
template.convertAndSend("myqueue", "foo");
String foo = (String) template.receiveAndConvert("myqueue");

........

@Configuration
public class RabbitConfiguration {

    @Bean
    public ConnectionFactory connectionFactory() {
        return new CachingConnectionFactory("localhost");
    }

    @Bean
    public AmqpAdmin amqpAdmin() {
        return new RabbitAdmin(connectionFactory());
    }

    @Bean
    public RabbitTemplate rabbitTemplate() {
        return new RabbitTemplate(connectionFactory());
    }

    @Bean
    public Queue myQueue() {
       return new Queue("myqueue");
    }
}
```

### 2.2 新特性
参考http://docs.spring.io/spring-amqp/reference/htmlsingle/#whats-new

## 3.参考
帮助文档的一部分详细介绍了Spring AMQP的各种组件。
主要章节介绍开发AMQP应用程序的核心教程。
本部分还包括有关示例应用程序。

### 3.1 使用Spring AMQP
在本章中，我们将探讨使用Spring AMQP开发应用程序的基本组件的接口和类。

#### 3.1.1 AMQP抽象
##### 介绍
Spring AMQP由几个模块组成，每个模块由发布中的JAR表示。这些模块有：spring-amqp和spring-rabbit。 spring-amqp模块包含`org.springframework.amqp.core`心 AMQP"model"的类。我们的目的是提供不依赖于任何特定AMQP代理实现或客户端库的泛型抽象。最终用户代码在供应商实现中将更加便携，因为它只能针对抽象层进行开发。这些抽象然后由代理特定的模块实现，例如spring-rabbit。目前只有一个RabbitMQ实现;但是除了RabbitMQ之外，使用Apache Qpid的.NET中的抽象已被验证。由于AMQP原则上在协议级别运行，所以RabbitMQ客户端可以与支持相同协议版本的任何代理一起使用，但目前我们还没有测试任何其他代理。

这里的概述假设您已经熟悉AMQP规范的基础知识。如果没有，请查看第5章“其他资源”中列出的资源。

##### Message(消息体)
0-8和0-9-1 AMQP规范不定义Message类或接口。
相反，当执行诸如basicPublish()的操作时，内容作为字节数组参数传递，并且附加属性作为单独的参数传入。
Spring AMQP将Message类定义为更普通的AMQP域模型。
Message类的目的是简单地将主体和属性封装在单个实例中，以便API可以更简单。

```java
public class Message {

    private final MessageProperties messageProperties;

    private final byte[] body;

    public Message(byte[] body, MessageProperties messageProperties) {
        this.body = body;
        this.messageProperties = messageProperties;
    }

    public byte[] getBody() {
        return this.body;
    }

    public MessageProperties getMessageProperties() {
        return this.messageProperties;
    }
}
```

MessageProperties接口定义了几个常见的属性，如messageId，timestamp，contentType等等。
这些属性也可以通过调用setHeader(String key，Object value)方法来扩展用户定义的头属性。

##### exchange(交换机)
Exchange接口表示AMQP Exchange，这是消息生产者发送到的。
代理的虚拟主机中的每个Exchange将具有唯一的名称以及一些其他属性：
```java
public interface Exchange {

    String getName();

    String getExchangeType();

    boolean isDurable();

    boolean isAutoDelete();

    Map<String, Object> getArguments();

}
```

如您所见，Exchange还具有由ExchangeTypes中定义的常量表示的类型。
基本类型有：Direct, Topic, Fanout, 和 Headers。
在核心包中，您将找到每种类型的Exchange接口的实现。
这些Exchange类型的行为在如何处理与队列绑定方面有所不同。
例如，Direct exchange允许队列被固定的routing key(通常是队列的名称)绑定。
Topic exchange支持绑定与路由模式，可能包括`*`和`#`通配符
> AMQP规范还要求任何代理提供没有名称的“默认”Exchange。
所有被声明的队列将被绑定到该默认的Exchange名称作为routing key。
您将在3.1.4节“AmqpTemplate”中了解Spring AMQP中默认Exchange的使用情况。

##### Queue(队列)
Queue类表示消息使用者接收消息的组件。
像各种Exchange类一样，我们的实现意图是这个核心AMQP类型的抽象表示。

```java
public class Queue  {

    private final String name;

    private volatile boolean durable;

    private volatile boolean exclusive;

    private volatile boolean autoDelete;

    private volatile Map<String, Object> arguments;

    /**
     * 队列是持久的，非排他的和非自动删除的。
     *
     * @param name 队列名
     */
    public Queue(String name) {
        this(name, true, false, false);
    }

    // Getters and Setters omitted for brevity

}
```

请注意，构造函数采用队列名称。根据实现，管理模板可以提供用于生成唯一命名的队列的方法。这样的队列可以用作`reply-to`地址或其他临时情况。因此，自动生成的队列的exclusive和autoDelete属性都将设置为true。

> 有关使用命名空间支持(包括队列参数)声明队列的信息，请参见第3.1.10节“配置代理”中的队列部分。

##### Binding(绑定)
鉴于生产者发送到Exchange并且消费者从队列接收到消息，将队列连接到exchange的绑定对于通过消息传递连接这些生产者和消费者至关重要。
在Spring AMQP中，我们定义一个Binding类来表示这些连接。
我们来看看将队列绑定到交换机的基本选项。

您可以使用固定的routing key将队列绑定到DirectExchange。
```java
new Binding(someQueue, someDirectExchange, "foo.bar")
```

您可以使用路由模式将队列绑定到TopicExchange。
```java
new Binding(someQueue, someTopicExchange, "foo.*")
```

您可以使用无routing key将Queue绑定到FanoutExchange。
```java
new Binding(someQueue, someFanoutExchange)
```

我们还提供了一个BindingBuilder进行链式风格的构建。
```java
Binding b = BindingBuilder.bind(someQueue).to(someTopicExchange).with("foo.*");
```

> 为了清楚起见，上面显示了BindingBuilder类，但是对于bind()方法使用静态导入时，此样式很好。

Binding类的一个实例本身就是持有关于连接的数据。
换句话说，它不是一个“活跃”组件。
但是，正如您将在3.1.10节“配置代理”中看到的，AmqpAdmin类可以使用Binding实例来实际触发代理上的绑定操作。
另外，正如你将在同一部分中看到的，Binding实例可以使用@Configuration类中的Spring的@ Bean风格进行定义。
还有一个方便的基类，它进一步简化了生成AMQP相关bean定义的方法，并识别队列，交换和绑定，以便在应用程序启动时将它们全部声明在AMQP代理上。

AmqpTemplate也在核心包中定义。作为实际AMQP消息传递中涉及的主要组件之一，将在其自己的部分中详细讨论(参见第3.1.4节“AmqpTemplate”)。

#### 3.1.2连接和资源管理
##### 介绍
我们上一节描述的AMQP模型是通用的，适用于所有实现，当我们进入资源管理时，特定的场景需要特殊实现。因此，在本节中，我们将专注于仅存在于我们的“spring-rabbit”模块中的代码，因为在这一点上，RabbitMQ是唯一支持的实现。

用于管理与RabbitMQ代理的连接的中心组件是`ConnectionFactory`接口。 `ConnectionFactory`实现的责任是提供一个`org.springframework.amqp.rabbit.connection.Connection`的实例，它是`com.rabbitmq.client.Connection`的包装器。我们提供的唯一具体实现是`CachingConnectionFactory`，默认情况下，它建立可以由应用程序共享的单个连接代理。连接是共享的，因为与AMQP通信的“工作单位”实际上是一个“通道”(在某些方面，这与JMS中的连接和会话之间的关系类似)。您可以想像，连接实例提供了一个`createChannel`方法。 `CachingConnectionFactory`实现支持对这些通道的缓存，并且基于它们是否是事务来维护单独的通道高速缓存。创建`CachingConnectionFactory`实例时，可以通过构造函数提供主机名。还应提供用户名和密码属性。如果要配置通道缓存的大小(默认值为25)，您也可以在此处调用`setChannelCacheSize()`方法。

从1.3版开始，`CachingConnectionFactory`可以配置为缓存连接以及仅通道。
在这种情况下，每次调用`createConnection()`都会创建一个新的连接(或从缓存中检索一个空闲的连接)。
关闭连接将返回到缓存(如果尚未达到高速缓存大小)。
在这种连接上创建的通道也被缓存。
使用单独的连接在某些环境中可能是有用的，例如从HA群集中消耗负载均衡器连接到不同的群集成员。
将`cacheMode`设置为`CacheMode.CONNECTION`。

> 这不限制连接数，它指定允许多少空闲打开连接。

从版本1.5.5开始，提供了一个新的属性`connectionLimit`。当设置此项时，它限制允许的连接总数。设置后，如果达到限制，则使用`channelCheckoutTimeLimit`等待连接变为空闲状态。如果超过时间，则抛出`AmqpTimeoutException`。

> 重要提示
    当缓存模式为CONNECTION时，不支持自动声明队列等(请参阅“自动声明交换，队列和绑定”一节)。
    此外，在编写本文时，rabbitmq-client库默认为每个连接创建一个固定的线程池(5个线程)。当使用大量连接时，应考虑在CachingConnectionFactory上设置自定义执行程序。然后，所有连接将使用相同的执行程序，并且可以共享它的线程。执行者的线程池应该是无限制的，或者针对预期的利用率进行适当设置(通常每个连接至少有一个线程)。如果在每个连接上创建多个通道，则池大小将影响并发性，因此变量(或简单的缓存)线程池执行器将是最合适的。

重要的是要明白，缓存大小(默认情况下)不是限制，只是可以缓存的通道数。具有例如10的高速缓存大小，实际上可以使用任何数量的频道。如果正在使用10个以上的通道，并将它们全部返回到缓存，则10将进入高速缓存;其余部分将被物理关闭。

从版本1.6开始，默认通道缓存大小从1增加到25。在高容量，多线程环境中，小缓存意味着以高速率创建和关闭通道。增加默认缓存大小将避免这种开销。您应该通过RabbitMQ管理界面监视正在使用的频道，并考虑在创建和关闭许多通道时进一步增加高速缓存大小。缓存只会按需增长(以适应应用程序的并发需求)，因此此更改不会影响现有的低容量应用程序。

从版本1.4.2开始，`CachingConnectionFactory`具有一个属性`channelCheckoutTimeout`。当此属性大于零时，`channelCacheSiz`e将成为可在连接上创建的通道数量的限制。如果达到限制，调用线程将阻塞，直到通道可用或达到此超时，在这种情况下抛出一个`AmqpTimeoutException`。

> 框架内使用的通道(例如RabbitTemplate)将可靠地返回到缓存。如果您在框架之外创建渠道(例如通过直接访问连接并调用createChannel())，则必须可靠地将其返回(通过关闭)，也许在finally块中，以避免使用通道。

```java
CachingConnectionFactory connectionFactory = new CachingConnectionFactory("somehost");
connectionFactory.setUsername("guest");
connectionFactory.setPassword("guest");

Connection connection = connectionFactory.createConnection();
```

使用XML时，配置可能如下所示：

```xml
<bean id="connectionFactory"
      class="org.springframework.amqp.rabbit.connection.CachingConnectionFactory">
    <constructor-arg value="somehost"/>
    <property name="username" value="guest"/>
    <property name="password" value="guest"/>
</bean>
```

> 还有一个`SingleConnectionFactory`实现，仅在框架的单元测试代码中可用。它比`CachingConnectionFactory`简单，因为它不缓存通道，但是由于其缺乏性能和弹性，它不适用于简单测试之外的实际使用。如果您因为某些原因需要实现自己的`ConnectionFactory`，那么`AbstractConnectionFactory`基类可能会提供一个很好的起点。

可以使用rabbit命名空间快速方便地创建ConnectionFactory：

```xml
<rabbit:connection-factory id="connectionFactory"/>
```

在大多数情况下，这将是优先的，因为框架可以为您选择最佳默认值。创建的实例将是一个`CachingConnectionFactory`。请注意，通道的默认缓存大小为25.如果要更多通道被缓存，则通过channelCacheSize属性设置较大的值。在XML中，它将如下所示：

```xml
<bean id="connectionFactory"
      class="org.springframework.amqp.rabbit.connection.CachingConnectionFactory">
    <constructor-arg value="somehost"/>
    <property name="username" value="guest"/>
    <property name="password" value="guest"/>
    <property name="channelCacheSize" value="50"/>
</bean>
```

使用命名空间，您只需添加channel-cache-size属性即可：
```xml
<rabbit:connection-factory
    id="connectionFactory" channel-cache-size="50"/>
```

默认缓存模式是CHANNEL，但您可以将其配置为缓存连接;在这种情况下，我们使用connection-cache-size：
```xml
<rabbit:connection-factory
    id="connectionFactory" cache-mode="CONNECTION" connection-cache-size="25"/>
```

可以使用命名空间设置主机和端口属性
```xml
<rabbit:connection-factory
    id="connectionFactory" host="somehost" port="5672"/>
```

或者，如果在群集环境中运行，请使用addresses属性。
```xml
<rabbit:connection-factory
    id="connectionFactory" addresses="host1:5672,host2:5672"/>
```

这里有一个自定义线程工厂的例子，它使用rabbitmq-前缀线程名。
```xml
<rabbit:connection-factory id="multiHost" virtual-host="/bar" addresses="host1:1234,host2,host3:4567"
    thread-factory="tf"
    channel-cache-size="10" username="user" password="password" />

<bean id="tf" class="org.springframework.scheduling.concurrent.CustomizableThreadFactory">
    <constructor-arg value="rabbitmq-" />
</bean>
```

从1.7版开始，提供了一个`ConnectionNameStrategy`，用于注入到`AbstractionConnectionFactory`中。生成的名称用于目标RabbitMQ连接的应用程序特定标识。如果RabbitMQ服务器支持，连接名称将显示在管理界面中。该值不必是唯一的，不能用作连接标识符，例如在HTTP API请求中。该值应该是人类可读的，并且是`connection_name`键下的`ClientProperties`的一部分。可以用作简单的Lambda：

```java
connectionFactory.setConnectionNameStrategy(connectionFactory -> "MY_CONNECTION");
```

`ConnectionFactory`参数可以用来区分目标连接名称一些逻辑。默认情况下，使用`AbstractConnectionFactory`的`beanName`和内部计数器来生成`connection_name`。`<rabbit:connection-factory>`命名空间组件也提供了`connection-name-strategy`属性。

##### 配置底层客户端连接工厂
`CachingConnectionFactory`使用Rabbit客户端`ConnectionFactory`的实例;当设置`CachingConnectionFactory`上的等效属性时，会传递一些配置属性(例如，`host`，`port`，`userName`，`password`，`requestedHeartBeat`，`connectionTimeout`)。要设置其他属性(例如`clientProperties`)，请定义 rabbit factory的实例，并使用适当的`CachingConnectionFactory`构造函数提供对它的引用。当使用如上所述的命名空间时，在`connection-factory`属性中提供对配置工厂的引用。为方便起见，提供了一个工厂bean，以帮助在Spring应用程序环境中配置连接工厂，如下一节所述。

```xml
<rabbit:connection-factory
      id="connectionFactory" connection-factory="rabbitConnectionFactory"/>
```

> 4.0.x客户端默认启用自动恢复;与此功能兼容，Spring AMQP具有自己的恢复机制，并且通常不需要客户端恢复功能。建议禁用`amqp-client`自动恢复，以避免在代理可用时使`AutoRecoverConnectionNotCurrentlyOpenException`异常，但连接尚未恢复。您可能会注意到这种异常，例如，当`RabbitTemplate`中配置`RetryTemplate`时，即使在群集中的其他代理进行故障转移时也是如此。由于自动恢复连接在定时器上恢复，因此使用Spring AMQP的恢复机制可以更快地恢复连接。从版本1.7.1开始，除非您明确创建自己的RabbitMQ连接工厂并将其提供给`CachingConnectionFactory`，否则Spring AMQP将禁用它。由`RabbitConnectionFactoryBean`创建的`RabbitMQ ConnectionFactory`实例也将默认禁用该选项。

##### RabbitConnectionFactoryBean和配置SSL
从版本1.4开始，提供了一个方便的`RabbitConnectionFactoryBean`，以便使用依赖注入在底层客户端连接工厂上方便地配置SSL属性。其他设置者只需委托给底层工厂。以前，您必须以编程方式配置SSL选项。

```xml
<rabbit:connection-factory id="rabbitConnectionFactory"
    connection-factory="clientConnectionFactory"
    host="${host}"
    port="${port}"
    virtual-host="${vhost}"
    username="${username}" password="${password}" />

<bean id="clientConnectionFactory"
        class="org.springframework.xd.dirt.integration.rabbit.RabbitConnectionFactoryBean">
    <property name="useSSL" value="true" />
    <property name="sslPropertiesLocation" value="file:/secrets/rabbitSSL.properties"/>
</bean>
```
有关配置SSL的信息，请参阅RabbitMQ文档。省略keyStore和trustStore配置以通过SSL连接，而无需证书验证。密钥和信任存储配置可以提供如下：

`sslPropertiesLocation`属性是指向包含以下键的属性文件的Spring `Resource`：

```properties
keyStore=file:/secret/keycert.p12
trustStore=file:/secret/trustStore
keyStore.passPhrase=secret
trustStore.passPhrase=secret
```

`keyStore`和`truststore`是指向磁盘的Spring资源文件。通常，此属性文件将由操作系统保护，应用程序具有读访问权限。

从Spring AMQP版本1.5开始，这些属性可以直接在工厂bean上设置。如果提供了离散属性和`sslPropertiesLocation`，后者中的属性将覆盖离散值。

##### 路由连接工厂
从1.3版开始，引入了`AbstractRoutingConnectionFactory`。这提供了一种机制来为几个`ConnectionFactories`配置映射，并在运行时由某些`lookupKey`确定目标`ConnectionFactory`。通常，实现检查线程绑定的上下文。为了方便起见，Spring AMQP提供了`SimpleRoutingConnectionFactory`，它从`SimpleResourceHolder`获取当前线程绑定的`lookupKey`：

```xml
<bean id="connectionFactory"
      class="org.springframework.amqp.rabbit.connection.SimpleRoutingConnectionFactory">
	<property name="targetConnectionFactories">
		<map>
			<entry key="#{connectionFactory1.virtualHost}" ref="connectionFactory1"/>
			<entry key="#{connectionFactory2.virtualHost}" ref="connectionFactory2"/>
		</map>
	</property>
</bean>

<rabbit:template id="template" connection-factory="connectionFactory" />
```

```java
public class MyService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void service(String vHost, String payload) {
        SimpleResourceHolder.bind(rabbitTemplate.getConnectionFactory(), vHost);
        rabbitTemplate.convertAndSend(payload);
        SimpleResourceHolder.unbind(rabbitTemplate.getConnectionFactory());
    }

}
```
重要的是在使用后取消绑定资源。有关更多信息，请参阅`AbstractRoutingConnectionFactory`的JavaDocs。

从版本1.4开始，RabbitTemplate支持SpEL sendConnectionFactorySelectorExpression和receiveConnectionFactorySelectorExpression属性，它们在每个AMQP协议交互操作(send，sendAndReceive，receive或receiveAndReply)上进行评估，解析为所提供的AbstractRoutingConnectionFactory的lookupKey值。
Bean的引用,比如“@vHostResolver.getVHost(#root)”可以在表达式中使用。对于发送操作，要发送的消息是root评估对象;对于接收操作，queueName是root评估对象。

路由算法是：如果选择器表达式为空，或者被评估为null，或者所提供的ConnectionFactory不是AbstractRoutingConnectionFactory的一个实例，则一切都将如前所述，依赖于提供的ConnectionFactory实现。如果评估结果不为空，但是没有用于该lookupKey的目标ConnectionFactory，并且使用lenientFallback = true配置AbstractRoutingConnectionFactory，则会发生相同的情况。当然，在一个AbstractRoutingConnectionFactory的情况下，它会基于determinCurrentLookupKey()来回溯到其路由实现。但是，如果lenientFallback = false，则会抛出IllegalStateException异常。

命名空间支持还在`<rabbit:template>`组件上提供`send-connection-factory-selector-expression`和`receive-connection-factory-selector-expression`属性。

从1.4版开始，您可以在监听器容器中配置路由连接工厂。在这种情况下，队列名称列表用作查找键。例如，如果使用setQueueNames(“foo”，“bar”)配置容器，则查找键将为“[foo，bar]”(无空格)。

从版本1.6.9开始，您可以在监听器容器上使用setLookupKeyQualifier添加限定符到查找键。例如，这可以使用相同名称监听队列，但是不可以在不同的虚拟主机中(每个队列中都有一个连接工厂)。

例如，使用查找键限定符foo和容器监听队列栏，您将注册目标连接工厂的查找键将为foo [bar]。

##### Queue Affinity和LocalizedQueueConnectionFactory
在群集中使用HA队列时，为获得最佳性能，可能希望连接到主队列所在的物理代理。而CachingConnectionFactory可以配置多个代理地址;这是故障切换，客户端将尝试按顺序连接。 LocalizedQueueConnectionFactory使用由管理插件提供的REST API来确定哪个节点已被掌握。然后，它将创建(或从缓存中检索)将连接到该节点的CachingConnectionFactory。如果连接失败，则确定新的主节点，并且消费者连接到它。 LocalizedQueueConnectionFactory配置了默认连接工厂，以防无法确定队列的物理位置，在这种情况下，它将正常连接到集群。

LocalizedQueueConnectionFactory是一个RoutingConnectionFactory，SimpleMessageListenerContainer使用队列名称作为查询键，如上面的“路由连接工厂”部分所述。

> 由于这个原因(使用查询的队列名称)，只有当容器配置为监听单个队列时，才能使用LocalizedQueueConnectionFactory。

> 必须在每个节点上启用RabbitMQ管理插件。

> 警告
    此连接工厂用于长期连接，例如SimpleMessageListenerContainer使用的连接。它不用于短连接使用，例如使用RabbitTemplate，因为在进行连接之前调用REST API的开销。此外，对于发布操作，队列是未知的，并且消息也被发布到所有集群成员，所以查找节点的逻辑没有什么价值。
    
以下是一个示例配置，使用Spring Boot的RabbitProperties配置工厂：

```java
@Autowired
private RabbitProperties props;

private final String[] adminUris = { "http://host1:15672", "http://host2:15672" };

private final String[] nodes = { "rabbit@host1", "rabbit@host2" };

@Bean
public ConnectionFactory defaultConnectionFactory() {
    CachingConnectionFactory cf = new CachingConnectionFactory();
    cf.setAddresses(this.props.getAddresses());
    cf.setUsername(this.props.getUsername());
    cf.setPassword(this.props.getPassword());
    cf.setVirtualHost(this.props.getVirtualHost());
    return cf;
}

@Bean
public ConnectionFactory queueAffinityCF(
        @Qualifier("defaultConnectionFactory") ConnectionFactory defaultCF) {
    return new LocalizedQueueConnectionFactory(defaultCF,
            StringUtils.commaDelimitedListToStringArray(this.props.getAddresses()),
            this.adminUris, this.nodes,
            this.props.getVirtualHost(), this.props.getUsername(), this.props.getPassword(),
            false, null);
}
```

请注意，前三个参数是addresses，adminUris和nodes。这些是位置性的，因为当容器尝试连接到队列时，它确定队列被掌握在哪个节点上并连接到同一阵列位置的地址。

##### Publisher Confirms and Returns
通过将CachingConnectionFactory的publisherConfirms和publisherReturns属性分别设置为“true”，支持确认和返回的消息。

设置这些选项时，工厂创建的通道将被包装在PublisherCallbackChannel中，该通道用于方便回调。当获得这样的频道时，客户端可以使用频道注册一个PublisherCallbackChannel.Listener。 PublisherCallbackChannel实现包含将 确认或返回 路由到适当的监听器的逻辑。以下部分将进一步说明这些功能。

> 有关更多背景信息，请参阅RabbitMQ小组题为“引入发布者确认”的博客文章。

##### 记录通道关闭事件
版本1.5中引入了一种使用户能够控制日志记录级别的机制。

CachingConnectionFactory使用默认策略来记录通道关闭，如下所示：
- 正常通道关闭(200 OK)不记录。
- 如果通道由于被动队列声明失败而关闭，则会在调试级别进行记录。
- 如果由于消费者条件验证不通过，则通道关闭，则会在INFO级别记录。
- 所有其他情况都记录在ERROR级别。

要修改此行为，请在其closeExceptionLogger属性中的CachingConnectionFactory中注入自定义ConditionalExceptionLogger。

另请参阅“消费者失败事件”一节。

##### 运行缓存属性

Table 3.1 CacheMode.CHANNEL的缓存属性

|属性|说明|
|-|-|
|connectionName|ConnectionNameStrategy生成的连接的名称。|
|channelCacheSize|当前配置的允许空闲的最大通道。|
|localPort|连接的本地端口(如果可用)。这可以用于与RabbitMQ管理界面上的连接/通道相关联。|
|idleChannelsTx|当前空闲(缓存)的事务通道的数量。|
|idleChannelsNotTx|当前空闲(高速缓存)的非事务性通道的数量。|
|idleChannelsTxHighWater|同时空闲(缓存)的事务通道的最大数量。|
|idleChannelsNotTxHighWater|非事务性通道的最大数量同时处于空闲状态(缓存)。|

Table 3.2 CacheMode.CONNECTION的缓存属性

|属性|说明|
|-|-|
|connectionName:localPort|ConnectionNameStrategy生成的连接的名称。|
|openConnections|表示与经纪人连接的连接对象数。|
|channelCacheSize|当前配置的允许空闲的最大通道。|
|connectionCacheSize|当前配置的允许空闲的最大连接。|
|idleConnections|当前空闲的连接数。|
|idleConnectionsHighWater|同时空闲的最大连接数。|
|idleChannelsTx:localPort|当前为此连接空闲(高速缓存)的事务通道的数量。属性名称的localPort部分可用于与RabbitMQ Admin UI上的连接/通道相关联。|
|idleChannelsNotTx:localPort|当前为此连接空闲(高速缓存)的非事务性通道的数量。属性名称的localPort部分可用于与RabbitMQ Admin UI上的连接/通道相关联。|
|idleChannelsTxHighWater:localPort|同时空闲(缓存)的事务通道的最大数量。属性名称的localPort部分可用于与RabbitMQ Admin UI上的连接/通道相关联。|
|idleChannelsNotTxHighWater:localPort|非事务性通道的最大数量同时处于空闲状态(缓存)。属性名称的localPort部分可用于与RabbitMQ Admin UI上的连接/通道相关联。|

`cacheMode`属性(还包括`CHANNEL`或`CONNECTION`)。

![此处输入图片的描述][1]


  [1]: http://docs.spring.io/spring-amqp/reference/htmlsingle/images/cacheStats.png.pagespeed.ce.IDb__HrgKf.png
  
  
##### RabbitMQ自动连接/拓扑恢复
从Spring AMQP的第一个版本开始，框架在发生代理失败时提供了自己的连接和通道恢复。另外，如第3.1.10节“配置代理”所述，当重新建立连接时，RabbitAdmin将重新声明任何基础架构bean(队列等)。因此，它不依赖于由amqp-client库提供的自动恢复。 Spring AMQP现在使用4.0.x版本的amqp-client，默认情况下启用自动恢复。 Spring AMQP仍然可以使用自己的恢复机制，如果您希望，在客户端禁用它(通过将底层RabbitMQ connectionFactory上的automaticRecoveryEnabled属性设置为false)。但是，该框架与启用自动恢复完全兼容。这意味着您在代码中创建的任何消费者(可能通过RabbitTemplate.execute())都可以自动恢复。

#### 3.1.3 添加自定义客户端连接属性
CachingConnectionFactory现在允许您访问底层连接工厂，例如设置自定义客户端属性：

```java
connectionFactory.getRabbitConnectionFactory().getClientProperties().put("foo", "bar");
```

查看连接时，这些属性将显示在RabbitMQ管理界面中。

#### 3.1.4 AmqpTemplate
##### 介绍
与Spring框架和相关项目提供的许多其他高级抽象一样，Spring AMQP提供了一个起着核心作用的“模板”。定义主要操作的界面称为AmqpTemplate。这些操作涵盖发送和接收消息的一般行为。换句话说，它们不是任何实现的唯一，因此名称中的“AMQP”。另一方面，该接口的实现与AMQP协议的实现相关。不同于JMS，它是本身是一个接口级API，AMQP是一个线级协议。该协议的实现提供自己的客户端库，因此模板接口的每个实现将取决于特定的客户端库。目前，只有一个实现：RabbitTemplate。在下面的示例中，您将经常看到“AmqpTemplate”的使用，但是当您查看配置示例或调用模板实例化和/或设置的任何代码摘录时，您将看到实现类型(例如“RabbitTemplate”)。

如上所述，AmqpTemplate接口定义了发送和接收消息的所有基本操作。我们将在下面的两个部分分别探讨消息发送和接收。

另请参见“AsyncRabbitTemplate”一节。

##### 添加重试功能
从版本1.3开始，您现在可以将RabbitTemplate配置为使用RetryTemplate来帮助处理代理连接问题。有关完整信息，请参阅spring-retry项目。以下只是使用指数退出策略和默认SimpleRetryPolicy的一个示例，它将在向调用者抛出异常之前进行三次尝试。

使用XML命名空间：
```xml
<rabbit:template id="template" connection-factory="connectionFactory" retry-template="retryTemplate"/>

<bean id="retryTemplate" class="org.springframework.retry.support.RetryTemplate">
    <property name="backOffPolicy">
        <bean class="org.springframework.retry.backoff.ExponentialBackOffPolicy">
            <property name="initialInterval" value="500" />
            <property name="multiplier" value="10.0" />
            <property name="maxInterval" value="10000" />
        </bean>
    </property>
</bean>
```

使用@Configuration:
```java
@Bean
public AmqpTemplate rabbitTemplate();
    RabbitTemplate template = new RabbitTemplate(connectionFactory());
    RetryTemplate retryTemplate = new RetryTemplate();
    ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy();
    backOffPolicy.setInitialInterval(500);
    backOffPolicy.setMultiplier(10.0);
    backOffPolicy.setMaxInterval(10000);
    retryTemplate.setBackOffPolicy(backOffPolicy);
    template.setRetryTemplate(retryTemplate);
    return template;
}
```

从版本1.4开始，除了retryTemplate属性之外，RabbitTemplate还支持recoveryCallback选项。它被用作`RetryTemplate.execute(RetryCallback<T，E> retryCallback，RecoveryCallback<T> recoveryCallback)`的第二个参数。

> RecoveryCallback有些限制，因为重试上下文仅包含lastThrowable字段。对于更复杂的用例，您应该使用外部RetryTemplate，以便您可以通过上下文的属性向RecoveryCallback传递附加信息：

```java
retryTemplate.execute(
    new RetryCallback<Object, Exception>() {

        @Override
        public Object doWithRetry(RetryContext context) throws Exception {
            context.setAttribute("message", message);
            return rabbitTemplate.convertAndSend(exchange, routingKey, message);
        }
    }, new RecoveryCallback<Object>() {

        @Override
        public Object recover(RetryContext context) throws Exception {
            Object message = context.getAttribute("message");
            Throwable t = context.getLastThrowable();
            // Do something with message
            return null;
        }
    });
}
```
在这种情况下，您不会将RetryTemplate注入RabbitTemplate。

##### Publisher Confirms and Returns
AmqpTemplate的RabbitTemplate实现Publisher Confirms and Returns。

对于返回的消息，模板的必需属性必须设置为true，否则强制表达式必须对特定消息进行求值。此功能需要一个CachedConnectionFactory，其publisherReturns属性设置为true(参见“发布者确认和返回”一节)。通过调用setReturnCallback(ReturnCallback callback)注册一个RabbitTemplate.ReturnCallback，返回给客户端。回调必须实现这个方法：

```java
void returnedMessage(Message message, int replyCode, String replyText,
          String exchange, String routingKey);
```

每个RabbitTemplate只支持一个ReturnCallback。另见“Reply Timeout”一节。

对于发布者确认(aka Publisher Acknowledgements)，该模板需要一个CachedConnectionFactory，其publisherConfirms属性设置为true。通过调用setConfirmCallback(ConfirmCallback callback)注册RabbitTemplate.ConfirmCallback，确认发送给客户端。回调必须实现这个方法：

```java
void confirm(CorrelationData correlationData, boolean ack, String cause);
```

CorrelationData是发送原始消息时由客户端提供的对象。 ack是真实的，对于一个nack是假的。对于nack，原因可能包含一个原因，如果在生成nack时可用。一个例子是向不存在的交换发送消息时。在这种情况下，经纪人关闭渠道;原因包括在内的原因。原因在版本1.4中被添加。

RabbitTemplate只支持一个ConfirmCallback。

> RabbitTemplate发送操作完成后，通道关闭;这将阻止在连接工厂缓存已满时(当缓存中有空间，通道没有物理关闭并且返回/确认将正常进行时)的接收确认或返回。当缓存已满时，框架会将关闭延迟最多5秒，以便允许接收确认/返回的时间。当使用确认时，当接收到最后一次确认时，通道将被关闭。当仅使用退货时，通道将保持打开5秒钟。通常建议将连接工厂的channelCacheSize设置为足够大的值，以便将发布消息的通道返回到缓存，而不是关闭。您可以使用RabbitMQ管理插件监视频道使用情况;如果您看到频道正在快速打开/关闭，您应该考虑增加缓存大小以减少服务器上的开销。

##### 消息集成
从版本1.4开始，构建在`RabbitTemplate`之上的`RabbitMessagingTemplate`提供了与Spring Framework消息抽象(即`org.springframework.messaging.Message`)的集成。这允许您使用`spring-messaging` `Message<?>`抽象发送和接收消息。这种抽象是由Spring Integration和Spring的STOMP支持的其他Spring项目使用的。有两个消息转换器涉及;一个用于在Spring消息传递`Message<?>`和Spring AMQP的`Message`抽象之间进行转换，另一个用于在Spring AMQP的Message抽象与底层RabbitMQ客户端库所需的格式之间进行转换。默认情况下，消息有效载荷由提供的`RabbitTemplate`的消息转换器转换。或者，您可以使用其他有效载荷转换器注入自定义`MessagingMessageConverter`：

```java
MessagingMessageConverter amqpMessageConverter = new MessagingMessageConverter();
amqpMessageConverter.setPayloadConverter(myPayloadConverter);
rabbitMessagingTempalte.setAmqpMessageConverter(amqpMessageConverter);
```

##### Validated User Id
从版本1.6开始，该模板现在支持用户id表达式(使用Java配置时的userIdExpression)。如果发送消息，则在评估此表达式后，设置用户id属性(如果尚未设置)。用于评估的根对象是要发送的消息。

例子：
```xml
<rabbit:template ... user-id-expression="'guest'" />

<rabbit:template ... user-id-expression="@myConnectionFactory.username" />
```

第一个例子是一个文字表达;第二个从应用程序上下文中的连接工厂bean获取用户名属性。

#### 3.1.5 发送消息
##### 介绍
发送消息时，可以使用以下任一方法：
```java
void send(Message message) throws AmqpException;

void send(String routingKey, Message message) throws AmqpException;

void send(String exchange, String routingKey, Message message) throws AmqpException;
```

我们可以用上面列出的最后一个方法开始讨论，因为它实际上是最明确的。它允许在运行时提供AMQP Exchange名称以及路由密钥。最后一个参数是负责实例创建Message实例的回调。使用此方法发送消息的示例可能是这样的：

```java
amqpTemplate.send("marketData.topic", "quotes.nasdaq.FOO",
    new Message("12.34".getBytes(), someProperties));
```

如果您打算在大部分或全部时间使用该模板实例发送到同一个交换机，则可以在模板本身上设置“交换”属性。在这种情况下，可以使用上面列出的第二种方法。以下示例在功能上等同于上一个示例：

```java
amqpTemplate.setExchange("marketData.topic");
amqpTemplate.send("quotes.nasdaq.FOO", new Message("12.34".getBytes(), someProperties));
```

如果在模板上设置了“交换”和“路由密钥”属性，则可以使用仅接受消息的方法：

```java
amqpTemplate.setExchange("marketData.topic");
amqpTemplate.setRoutingKey("quotes.nasdaq.FOO");
amqpTemplate.send(new Message("12.34".getBytes(), someProperties));
```

更好的思考交换和路由关键属性的方法是显式方法参数将始终覆盖模板的默认值。实际上，即使你没有在模板上显式设置这些属性，总是存在默认值。在这两种情况下，默认是一个空字符串，但这实际上是一个明智的默认值。就路由密钥而言，首先并不总是必需的(例如扇出交换机)。此外，队列可能与一个空字符串绑定到一个Exchange。这些都是依赖模板的路由密钥属性的默认空字符串值的合法场景。就Exchange名称而言，空字符串是常用的，因为AMQP规范将“默认Exchange”定义为没有名称。由于所有队列都使用其名称作为绑定值自动绑定到该默认Exchange(即直接Exchange)，所以上述第二种方法可用于通过默认Exchange进行的任何队列的简单点对点消息传递。只需提供队列名称作为“routingKey” - 或者通过在运行时提供方法参数：

```java
RabbitTemplate template = new RabbitTemplate(); // using default no-name Exchange
template.send("queue.helloWorld", new Message("Hello World".getBytes(), someProperties));
```

或者，如果您希望创建一个将主要或专门用于单个队列发布的模板，以下是完全合理的：

```java
RabbitTemplate template = new RabbitTemplate(); // using default no-name Exchange
template.setRoutingKey("queue.helloWorld"); // but we'll always send to this Queue
template.send(new Message("Hello World".getBytes(), someProperties));
```

##### Message Builder API
从版本1.3开始，消息构建器API由MessageBuilder和MessagePropertiesBuilder提供;它们提供了一种方便的“流利”手段创建消息或消息属性：

```java
Message message = MessageBuilder.withBody("foo".getBytes())
    .setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN)
    .setMessageId("123")
    .setHeader("bar", "baz")
    .build();
```
或
```java
MessageProperties props = MessagePropertiesBuilder.newInstance()
    .setContentType(MessageProperties.CONTENT_TYPE_TEXT_PLAIN)
    .setMessageId("123")
    .setHeader("bar", "baz")
    .build();
Message message = MessageBuilder.withBody("foo".getBytes())
    .andProperties(props)
    .build();
```

可以设置MessageProperties上定义的每个属性。其他方法包括setHeader(String key，String value)，removeHeader(String key)，removeHeaders()和copyProperties(MessageProperties属性)。每个属性设置方法都有一个set*IfAbsent()变体。在存在默认初始值的情况下，该方法命名为set*IfAbsentOrDefault()。

提供了五种静态方法来创建初始消息构建器：

```java
public static MessageBuilder withBody(byte[] body) 1

public static MessageBuilder withClonedBody(byte[] body) 2

public static MessageBuilder withBody(byte[] body, int from, int to) 3

public static MessageBuilder fromMessage(Message message) 4

public static MessageBuilder fromClonedMessage(Message message) 5

```

1.由构建器创建的消息将具有直接引用参数的主体。
2.由构建器创建的消息将具有一个包含参数中字节副本的新数组的主体。
3.由构建器创建的消息将具有一个新的数组，其中包含参数中的字节范围。有关更多详细信息，请参阅Arrays.copyOfRange();
4.他由建设者创建的消息将具有一个直接引用参数体的主体。参数的属性被复制到一个新的MessageProperties对象。
5.由构建器创建的消息将具有一个包含参数的正文副本的新数组的主体。参数的属性被复制到一个新的MessageProperties对象。

```java
public static MessagePropertiesBuilder newInstance() 1

public static MessagePropertiesBuilder fromProperties(MessageProperties properties) 2

public static MessagePropertiesBuilder fromClonedProperties(MessageProperties properties) 3
```

1.新的消息属性对象使用默认值初始化。
2.构建器初始化，并且build()将返回所提供的属性对象。
3.参数的属性被复制到一个新的MessageProperties对象。

使用AmqpTemplate的RabbitTemplate实现，每个send()方法都有一个重载的版本，它接受一个附加的CorrelationData对象。当发布者确认被启用时，该对象将在第3.1.4节“AmqpTemplate”中描述的回调中返回。这允许发送者将确认(ack或nack)与发送的消息相关联。

从1.6.7版开始，引入了CorrelationAwareMessagePostProcessor接口，允许在消息转换后修改相关数据：

```java
Message postProcessMessage(Message message, Correlation correlation);
```

同样从版本1.6.7开始，提供了一个新的回调接口CorrelationDataPostProcessor;所有MessagePostProcessor(在send()方法中提供)以及在setBeforePublishPostProcessors()中提供的那些)之后调用。实现可以更新或替换send()方法中提供的相关数据(如果有的话)。消息和原始CorrelationData(如果有)作为参数提供。

```java
CorrelationData postProcess(Message message, CorrelationData correlationData);
```

##### Publisher Returns
当模板的强制属性为true时，返回的消息由第3.1.4节“AmqpTemplate”中描述的回调提供。

从版本1.4开始，RabbitTemplate支持根据每个请求消息评估的Spel mandatoryExpression属性作为根评估对象，解析为布尔值。
Bean的引用,比如“@myBean.isMandatory(#root)”可以在表达式中使用。

发送者返回也可以在RabbitTemplate内部用于发送和接收操作。有关详细信息，请参阅“回复超时”一节。

##### Batching
从版本1.4.2开始，已经介绍了BatchingRabbitTemplate。这是RabbitTemplate的一个子类，具有重写的发送方法，根据BatchingStrategy对消息进行批处理;只有批量完成时才将消息发送给RabbitMQ。

```java
public interface BatchingStrategy {

	MessageBatch addToBatch(String exchange, String routingKey, Message message);

	Date nextRelease();

	Collection<MessageBatch> releaseBatches();

}
```

> 警告
    批量数据保存在内存中;在发生系统故障的情况下，未发送的消息可能会丢失。
    
提供了SimpleBatchingStrategy。它支持将消息发送到单个交换/路由密钥。它有属性：

- batchSize 发送批次之前的消息数
- bufferLimit 批量消息的最大大小;这将超过batchSize，并且导致部分批处理被发送
- timeout 当没有新活动向批量添加消息时，将发送部分批次的时间

SimpleBatchingStrategy通过使用4字节二进制长度的每个嵌入式消息进行格式化。通过将springBatchFormat消息属性设置为lengthHeader4，将其传送到接收系统。

> 注意
批量消息由监听器容器(使用springBatchFormat消息头)自动分段。拒绝批次中的任何消息将导致整个批次被拒绝。

#### 3.1.6 Receiving messages
##### 介绍
消息接收总是比发送更复杂一些。有两种方式可以接收消息。更简单的选择是一次轮询方法调用来轮询单个消息。更复杂但更常见的方法是注册将按需异步接收消息的监听器。我们将在接下来的两个小节中看一下每种方法的一个例子。

##### Polling Consumer(轮询消费者)
AmqpTemplate本身可用于轮询的消息接收。默认情况下，如果没有消息可用，则立即返回null;没有阻塞。从版本1.5开始，您现在可以设置receiveTimeout(以毫秒为单位)，并且接收方法将阻塞长达数秒，等待消息。小于零的值意味着无限期地阻止(或至少直到与代理的连接丢失)。版本1.6引入了接收方法的变体，允许每次调用传递超时。

> 警告
由于接收操作为每个消息创建一个新的QueueingConsumer，因此该技术并不适用于大容量环境;考虑使用异步消费者，或者对于这些用例使用receiveTimeout为零。

有四种简单的接收方法可用。与发送方的Exchange一样，有一种方法需要直接在模板本身设置默认队列属性，并且有一种在运行时接受队列参数的方法。版本1.6引入了接受timeoutMillis的变体来根据每个请求重写receiveTimeout。

```java
Message receive() throws AmqpException;

Message receive(String queueName) throws AmqpException;

Message receive(long timeoutMillis) throws AmqpException;

Message receive(String queueName, long timeoutMillis) throws AmqpException;
```

就像在发送消息的情况下，AmqpTemplate有一些方便的接收POJO而不是Message实例的方法，并且实现将提供一种自定义用于创建返回的对象的MessageConverter的方法：

```java
Object receiveAndConvert() throws AmqpException;

Object receiveAndConvert(String queueName) throws AmqpException;

Message receiveAndConvert(long timeoutMillis) throws AmqpException;

Message receiveAndConvert(String queueName, long timeoutMillis) throws AmqpException;
```

类似于sendAndReceive方法，从版本1.3开始，AmqpTemplate具有几个方便的receiveAndReply方法来同步接收，处理和回复消息：

```java
<R, S> boolean receiveAndReply(ReceiveAndReplyCallback<R, S> callback)
	   throws AmqpException;

<R, S> boolean receiveAndReply(String queueName, ReceiveAndReplyCallback<R, S> callback)
 	throws AmqpException;

<R, S> boolean receiveAndReply(ReceiveAndReplyCallback<R, S> callback,
	String replyExchange, String replyRoutingKey) throws AmqpException;

<R, S> boolean receiveAndReply(String queueName, ReceiveAndReplyCallback<R, S> callback,
	String replyExchange, String replyRoutingKey) throws AmqpException;

<R, S> boolean receiveAndReply(ReceiveAndReplyCallback<R, S> callback,
 	ReplyToAddressCallback<S> replyToAddressCallback) throws AmqpException;

<R, S> boolean receiveAndReply(String queueName, ReceiveAndReplyCallback<R, S> callback,
			ReplyToAddressCallback<S> replyToAddressCallback) throws AmqpException;
```

AmqpTemplate实现负责接收和回复阶段。在大多数情况下，您应该只提供一个ReceiveAndReplyCallback的实现来为接收到的消息执行一些业务逻辑，如果需要，可以构建回复对象或消息。注意，ReceiveAndReplyCallback可能返回null。在这种情况下，没有发送回复，receiveAndReply类似于receive方法。这允许将相同的队列用于消息的混合，其中一些可能不需要回复。

仅当提供的回调不是ReceiveAndReplyMessageCallback的实例(提供原始消息交换合同)时，才应用自动消息(请求和回复)转换。

ReplyToAddressCallback对于需要自定义逻辑在运行时根据接收到的消息和ReceiveAndReplyCallback的回复来确定replyTo地址的情况很有用。默认情况下，请求消息中的replyTo信息用于路由回复。

以下是基于POJO的接收和回复的示例...

```java
boolean received =
        this.template.receiveAndReply(ROUTE, new ReceiveAndReplyCallback<Order, Invoice>() {

                public Invoice handle(Order order) {
                        return processOrder(order);
                }
        });
if (received) {
        log.info("We received an order!");
}
```

##### Asynchronous Consumer(异步消费者)

> 注意
Spring AMQP还通过使用@RabbitListener注解来支持带注解的监听端点，并提供了一种开放的基础设施，以编程方式注册端点。这是设置异步消费者的最方便的方法，有关详细信息，请参阅“注解驱动的监听器端点”一节。

**Message Listener**

对于异步消息接收，涉及专用组件(而不是AmqpTemplate)。该组件是消息回收消息的容器。我们将在短时间内查看容器及其属性，但首先我们应该看一下回调，因为这样你的应用程序代码将与邮件系统集成在一起。从MessageListener接口的实现开始，有几个回调选项：

```java
public interface MessageListener {
    void onMessage(Message message);
}
```

如果您的回调逻辑由于任何原因取决于AMQP Channel实例，您可以改为使用ChannelAwareMessageListener。它看起来相似，但有一个额外的参数：

```java
public interface ChannelAwareMessageListener {
    void onMessage(Message message, Channel channel) throws Exception;
}
```

**MessageListenerAdapter**
如果您希望在应用程序逻辑和消息传递API之间保持更严格的分隔，则可以依靠框架提供的适配器实现。这通常被称为“消息驱动的POJO”支持。使用适配器时，只需要提供适配器本身应该调用的实例的引用。

```java
MessageListenerAdapter listener = new MessageListenerAdapter(somePojo);
    listener.setDefaultListenerMethod("myMethod");
```

您可以将适配器子类化并提供getListenerMethodName()的实现，以根据消息动态选择不同的方法。这个方法有两个参数，即原始的消息和extractMessage，后者是任何转换的结果。默认情况下，配置SimpleMessageConverter;有关可用的其他转换器的更多信息和信息，请参阅“SimpleMessageConverter”一节。

从版本1.4.2开始，原始消息具有consumerQueue和consumerTag属性，可用于确定从哪个队列接收消息。

从版本1.5开始，您可以将消费者队列/标记的映射配置为方法名称，以动态选择要调用的方法。如果map中没有条目，我们将回到默认监听器方法。

**Container**
现在您已经看到了Message-listening回调的各种选项，我们可以将注意力转向容器。基本上，容器处理“主动”的责任，使得监听器回调可以保持被动。容器是“生命周期”组件的示例。它提供了启动和停止的方法。配置容器时，您基本上弥合了AMQP队列和MessageListener实例之间的差距。您必须提供对ConnectionFactory以及该监听器应从其消费消息的队列名称或队列实例的引用。这是使用默认实现的最基本的例子SimpleMessageListenerContainer：

```java
SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
container.setConnectionFactory(rabbitConnectionFactory);
container.setQueueNames("some.queue");
container.setMessageListener(new MessageListenerAdapter(somePojo));
```

作为“活动”组件，最常见的是使用bean定义创建监听器容器，以便它可以在后台运行。这可以通过XML来完成：

```xml
<rabbit:listener-container connection-factory="rabbitConnectionFactory">
    <rabbit:listener queues="some.queue" ref="somePojo" method="handle"/>
</rabbit:listener-container>
```

或者，您可能更喜欢使用与上述实际代码片段非常相似的@Configuration样式：

```java
@Configuration
public class ExampleAmqpConfiguration {

    @Bean
    public SimpleMessageListenerContainer messageListenerContainer() {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(rabbitConnectionFactory());
        container.setQueueName("some.queue");
        container.setMessageListener(exampleListener());
        return container;
    }

    @Bean
    public ConnectionFactory rabbitConnectionFactory() {
        CachingConnectionFactory connectionFactory =
            new CachingConnectionFactory("localhost");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        return connectionFactory;
    }

    @Bean
    public MessageListener exampleListener() {
        return new MessageListener() {
            public void onMessage(Message message) {
                System.out.println("received: " + message);
            }
        };
    }
}
```

从RabbitMQ版本3.2开始，代理现在支持消费者优先级(请参阅使用RICSMQ使用消费者优先级)。这通过在消费者上设置x-priority参数来启用。 SimpleMessageListenerContainer现在支持设置消费者参数：

```java
container.setConsumerArguments(Collections.
<String, Object> singletonMap("x-priority", Integer.valueOf(10)));
```

为方便起见，命名空间提供了listener元素的priority属性：

```xml
<rabbit:listener-container connection-factory="rabbitConnectionFactory">
    <rabbit:listener queues="some.queue" ref="somePojo" method="handle" priority="10" />
</rabbit:listener-container>
```

从版本1.3开始，容器正在监听的队列可以在运行时修改;Section 3.1.18, “Listener Container Queues”.

**auto-delete Queues**
当容器配置为监听自动删除队列时，或者队列具有x-expires选项，或者Broker上配置了“即时生存”策略，则当容器为停止(最后消费者被取消)。在版本1.3之前，由于队列丢失，容器无法重新启动;当连接被关闭/打开时，RabbitAdmin只会自动重新发送队列等，当容器停止/启动时，不会发生这种情况。

从版本1.3开始，容器现在将在启动期间使用RabbitAdmin重新声明任何丢失的队列。

您还可以使用条件声明(称为“条件声明”一节)以及auto-startup =“false”管理员延迟队列声明，直到容器启动。

```xml
<rabbit:queue id="otherAnon" declared-by="containerAdmin" />

<rabbit:direct-exchange name="otherExchange" auto-delete="true" declared-by="containerAdmin">
    <rabbit:bindings>
        <rabbit:binding queue="otherAnon" key="otherAnon" />
    </rabbit:bindings>
</rabbit:direct-exchange>

<rabbit:listener-container id="container2" auto-startup="false">
    <rabbit:listener id="listener2" ref="foo" queues="otherAnon" admin="containerAdmin" />
</rabbit:listener-container>

<rabbit:admin id="containerAdmin" connection-factory="rabbitConnectionFactory"
    auto-startup="false" />
```

在这种情况下，队列和交换由containerAdmin声明，它具有auto-startup =“false”，因此在上下文初始化期间不会声明元素。同样，由于同样的原因，容器也没有启动。当容器稍后启动时，它使用它来引用containerAdmin来声明元素。

##### Batched Messages(批量消息)
批量消息由监听器容器(使用springBatchFormat消息头)自动分段。拒绝批次中的任何消息将导致整个批次被拒绝。有关批处理的更多信息，请参阅“批处理”一节。

##### Consumer Failure Events(消费失败事件)
从1.5版开始，SimpleMessageListenerContainer每当监听器(消费者)遇到某种故障时，都会发布应用程序事件。事件ListenerContainerConsumerFailedEvent具有以下属性：

- container - 消费者遇到问题的监听器容器。
- reason - 失败的一个文本原因。
- fatal - 一个布尔值表示失败是否是致命的;与非致命的例外，容器将尝试重新启动消费者，根据retryInterval。
- throwable - the Throwable that was caught.

这些事件可以通过实现`ApplicationListener<ListenerContainerConsumerFailedEvent>`来消耗。

> 当并发消费者大于1时，系统范围的事件(如连接失败)将由所有消费者发布。

如果一个消费者失败，因为一个如果它的队列被专门使用，默认情况下，以及发布事件，将发出一个WARN日志。要更改此日志记录行为，请在SimpleMessageListenerContainer的exclusiveConsumerExceptionLogger属性中提供自定义ConditionalExceptionLogger。另见“记录通道关闭事件”一节。

致命错误始终记录在ERROR级别;这不可修改。

##### Consumer Tags(消费者标签)
从版本1.4.5开始，您现在可以提供生成消费者标签的策略。默认情况下，消费者标签将由代理生成。

```java
public interface ConsumerTagStrategy {

    String createConsumerTag(String queue);

}
```
队列可用，因此可以(可选地)在标签中使用。
See Section 3.1.15, “Message Listener Container Configuration”.

##### Annotation-driven Listener Endpoints(注解驱动的监听器端点)

**介绍**
从版本1.4开始，异步接收消息的最简单的方法是使用带注解的监听端点基础结构。简而言之，它允许您将托管bean的方法公开为Rabbit监听器端点。

```java
@Component
public class MyService {

    @RabbitListener(queues = "myQueue")
    public void processOrder(String data) {
        ...
    }

}
```

上述示例的想法是，每当org.springframework.amqp.core.Queue“myQueue”上都有可用的消息时，将相应地调用processOrder方法(在这种情况下，与消息的有效内容相关)。

注解架构使用RabbitListenerContainerFactory为每个注解方法在幕后创建一个消息监听器容器。

在上面的例子中，myQueue必须已经存在并被绑定到一些交换。从版本1.5.0开始，只要应用程序上下文中存在RabbitAdmin，队列可以自动声明和绑定。

```java
@Component
public class MyService {

  @RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "myQueue", durable = "true"),
        exchange = @Exchange(value = "auto.exch", ignoreDeclarationExceptions = "true"),
        key = "orderRoutingKey")
  )
  public void processOrder(String data) {
    ...
  }

  @RabbitListener(bindings = @QueueBinding(
        value = @Queue,
        exchange = @Exchange(value = "auto.exch"),
        key = "invoiceRoutingKey")
  )
  public void processInvoice(String data) {
    ...
  }

}
```

在第一个示例中，如果需要，队列myQueue将自动声明(持久)，并与交换机绑定到路由密钥。在第二个示例中，将声明并绑定匿名(独占，自动删除)队列。可以提供多个QueueBinding条目，允许监听器监听多个队列。

只有DIRECT，FANOUT，TOPIC和HEADERS，这种机制支持交换类型。当需要更多高级配置时，请使用正常的@Bean定义

在第一个例子中，请注意ignoreDeclarationExchangeions对交换。这允许例如绑定到可能具有不同设置(如内部)的现有交换机。默认情况下，现有交换机的属性必须匹配。

从版本1.6开始，现在可以在@QueueBinding注解中为队列，交换和绑定指定参数。例如：

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "auto.headers", autoDelete = "true",
                        arguments = @Argument(name = "x-message-ttl", value = "10000",
                                                type = "java.lang.Integer")),
        exchange = @Exchange(value = "auto.headers", type = ExchangeTypes.HEADERS, autoDelete = "true"),
        arguments = {
                @Argument(name = "x-match", value = "all"),
                @Argument(name = "foo", value = "bar"),
                @Argument(name = "baz")
        })
)
public String handleWithHeadersExchange(String foo) {
    ...
}
```
请注意，x-message-ttl参数为队列设置为10秒。由于参数类型不是String，我们必须指定其类型;在这种情况下整数。与所有这样的声明一样，如果队列已经存在，那么这些参数必须与队列上的一致。对于headers exchange，我们设置绑定参数以匹配头foo设置为bar的消息，并且头baz必须与任何值一起显示。 x匹配参数意味着必须满足这两个条件。

参数名称，值，和类型可以是财产的占位符($ {…})或该表达式(# {…})。名称必须解析为字符串；类型表达式必须解析为类或类的完全限定名。价值必须解决的东西，可以被defaultconversionservice的类型(如在上面的例子中x-message-ttl)。

如果名称解析为null或空字符串，则忽略该参数。

**Meta-Annotations(元注解)**
有时您可能希望为多个监听器使用相同的配置。为了减少样板设置，您可以使用元注解来创建自己的监听器注解：

```java
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@RabbitListener(bindings = @QueueBinding(
        value = @Queue,
        exchange = @Exchange(value = "metaFanout", type = ExchangeTypes.FANOUT)))
public @interface MyAnonFanoutListener {
}

public class MetaListener {

    @MyAnonFanoutListener
    public void handle1(String foo) {
        ...
    }

    @MyAnonFanoutListener
    public void handle2(String foo) {
        ...
    }

}
```

在此示例中，由@MyAnonFanoutListener注解创建的每个监听器将将匿名自动删除队列绑定到扇出交换机的metaFanout。元注解机制很简单，因为用户定义的注解上的属性未被检查 - 因此您不能从元注解中覆盖设置。当需要更多高级配置时，请使用正常的@Bean定义。

**启用监听器端点注解**
要启用对@RabbitListener注解的支持，请将@EnableRabbit添加到您的一个@Configuration类中。

```java
@Configuration
@EnableRabbit
public class AppConfig {

    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        factory.setConcurrentConsumers(3);
        factory.setMaxConcurrentConsumers(10);
        return factory;
    }
}
```
默认情况下，基础组件将查找名为rabbitListenerContainerFactory的bean作为工厂用于创建消息监听器容器的源。在这种情况下，忽略RabbitMQ基础架构设置，可以使用3个线程的核心轮询大小和10个线程的最大池大小来调用processOrder方法。

可以自定义监听器容器工厂以使用每个注解，或者可以通过实现RabbitListenerConfigurer接口来配置显式默认值。仅当至少有一个端点没有特定的容器工厂注册时，才需要默认值。有关详细信息和示例，请参阅javadoc。

如果您喜欢XML配置，请使用`<rabbit:annotation-driven>`元素。

```xml
<rabbit:annotation-driven/>

<bean id="rabbitListenerContainerFactory"
      class="org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="concurrentConsumers" value="3"/>
    <property name="maxConcurrentConsumers" value="10"/>
</bean>
```

**Message Conversion for Annotated Methods**
在调用监听器之前，有两个转换步骤在管道中。第一个使用MessageConverter将传入的Spring AMQP消息转换为spring-message消息。当调用目标方法时，如有必要，将消息有效载荷转换为方法参数类型。

第一步的默认MessageConverter是一个Spring AMQP SimpleMessageConverter，用于处理转换为String和java.io.Serializable对象;所有其他的都保留为一个byte[]。在下面的讨论中，我们称之为消息转换器。

第二步的默认转换器是一个GenericMessageConverter，它委托给转换服务(DefaultFormattingConversionService的一个实例)。在下面的讨论中，我们称之为方法参数转换器。

要更改消息转换器，只需将其作为属性添加到容器工厂bean中：

```java
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    ...
    factory.setMessageConverter(new Jackson2JsonMessageConverter());
    ...
    return factory;
}
```

这配置了一个Jackson2转换器，期望头信息存在，以引导转换。

您还可以考虑一个ContentTypeDelegatingMessageConverter，它可以处理不同内容类型的转换。

在大多数情况下，除非要使用自定义ConversionService，否则不需要自定义方法参数转换器。

在1.6之前的版本中，转换JSON的类型信息必须在消息头中提供，或需要自定义ClassMapper。从版本1.6开始，如果没有类型信息头，则可以从目标方法参数推断类型。

> 此类型推断仅适用于方法级别的@RabbitListener。

See the section called “Jackson2JsonMessageConverter” for more information.

如果你想自定义方法参数转换器，你可以这样做，如下所示：
```java
@Configuration
@EnableRabbit
public class AppConfig implements RabbitListenerConfigurer {

    ...

    @Bean
    public DefaultMessageHandlerMethodFactory myHandlerMethodFactory() {
        DefaultMessageHandlerMethodFactory factory = new DefaultMessageHandlerMethodFactory();
        factory.setMessageConverter(new GenericMessageConverter(myConversionService()));
        return factory;
    }

    @Bean
    public ConversionService myConversionService() {
        DefaultConversionService conv = new DefaultConversionService();
        conv.addConverter(mySpecialConverter());
        return conv;
    }

    @Override
    public void configureRabbitListeners(RabbitListenerEndpointRegistrar registrar) {
        registrar.setMessageHandlerMethodFactory(myHandlerMethodFactory());
    }

    ...

}
```

> 重点
对于多方法监听器(参见“Multi-Method Listeners”一节)，方法选择是基于消息转换后消息的有效载荷;方法参数转换器只有在方法被选择之后才被调用。

**Programmatic Endpoint Registration**
RabbitListenerEndpoint提供了Rabbit端点的模型，并负责为该模型配置容器。基础设施允许您以编程方式配置端点，除了由RabbitListener注解检测到的端点。

```java
@Configuration
@EnableRabbit
public class AppConfig implements RabbitListenerConfigurer {

    @Override
    public void configureRabbitListeners(RabbitListenerEndpointRegistrar registrar) {
        SimpleRabbitListenerEndpoint endpoint = new SimpleRabbitListenerEndpoint();
        endpoint.setQueueNames("anotherQueue");
        endpoint.setMessageListener(message -> {
            // processing
        });
        registrar.registerEndpoint(endpoint);
    }
}
```
在上面的例子中，我们使用了SimpleRabbitListenerEndpoint，它提供了实际的MessageListener来调用，但是你也可以构建自己的端点变体来描述自定义调用机制。

应该注意的是，您也可以跳过使用@RabbitListener，只通过RabbitListenerConfigurer以编程方式注册您的端点。

**Annotated Endpoint Method Signature**
到目前为止，我们已经在我们的端点注入了一个简单的String，但实际上它可以有一个非常灵活的方法签名。我们重写它以使用自定义标题注入Order：
```java
@Component
public class MyService {

    @RabbitListener(queues = "myQueue")
    public void processOrder(Order order, @Header("order_type") String orderType) {
        ...
    }
}
```

这些是您可以在监听器端点注入的主要元素：

- 原始的org.springframework.amqp.core.Message。
- 接收消息的com.rabbitmq.client.Channel
- 表示传入AMQP消息的org.springframework.messaging.Message。请注意，此消息包含自定义标头和标准标题(由AmqpHeaders定义)。
    > 从版本1.6开始，入站deliveryMode标题现在在名称为AmqpHeaders.RECEIVED_DELIVERY_MODE的标题中可用，而不是AmqpHeaders.DELIVERY_MODE。
- @ Header-annotated方法参数来提取特定的头值，包括标准AMQP头。
- @ Headers-annotated参数，必须也可以分配给java.util.Map以获取对所有头的访问。

-

不被支持的类型(即消息和信道)的非注释元素被认为是有效载荷。您可以通过使用@Payload注释参数来使其显式。您还可以通过添加额外的@Valid来启用验证。

注入Spring Message消息抽象的能力特别适用于从传输特定消息中存储的所有信息中获益，而不依赖于特定于传输的API。

```java
@RabbitListener(queues = "myQueue")
public void processOrder(Message<Order> order) { ...
}
```

方法参数的处理由DefaultMessageHandlerMethodFactory提供，可以进一步自定义以支持其他方法参数。转换和验证支持也可以在这里定制。

例如，如果我们想在处理之前确保我们的订单有效，我们可以使用@Valid对有效负载进行注释，并配置必要的验证器，如下所示：
```java
@Configuration
@EnableRabbit
public class AppConfig implements RabbitListenerConfigurer {

    @Override
    public void configureRabbitListeners(RabbitListenerEndpointRegistrar registrar) {
        registrar.setMessageHandlerMethodFactory(myHandlerMethodFactory());
    }

    @Bean
    public DefaultMessageHandlerMethodFactory myHandlerMethodFactory() {
        DefaultMessageHandlerMethodFactory factory = new DefaultMessageHandlerMethodFactory();
        factory.setValidator(myValidator());
        return factory;
    }
}
```

**Listening to Multiple Queues**
使用queues属性时，可以指定关联的容器可以监听多个队列。您可以使用@Header注释来创建POJO方法可接收消息的队列名称。

```java
@Component
public class MyService {

    @RabbitListener(queues = { "queue1", "queue2" } )
    public void processOrder(String data, @Header(AmqpHeaders.CONSUMER_QUEUE) String queue) {
        ...
    }

}
```

从版本1.5开始，您可以使用属性占位符和SpEL来对队列名称进行外部化：

```java
@Component
public class MyService {

    @RabbitListener(queues = "#{'${property.with.comma.delimited.queue.names}'.split(',')}" )
    public void processOrder(String data, @Header(AmqpHeaders.CONSUMER_QUEUE) String queue) {
        ...
    }

}
```
在版本1.5之前，只能通过这种方式指定一个队列;每个队列需要一个单独的属性。

**Reply Management**
MessageListenerAdapter中的现有支持已经允许您的方法具有非空返回类型。在这种情况下，调用的结果被封装在以原始消息的ReplyToAddress头中指定的地址或在监听器上配置的默认地址中发送的消息中。现在可以使用消息传递抽象的@SendTo注释来设置该默认地址。

假设我们的processOrder方法现在应该返回一个OrderStatus，可以按如下方式写入自动发送回复：
```java
@RabbitListener(destination = "myQueue")
@SendTo("status")
public OrderStatus processOrder(Order order) {
    // order processing
    return status;
}
```
如果您需要以传输独立的方式设置其他标头，则可以返回一条消息，如下所示：
```java
@RabbitListener(destination = "myQueue")
@SendTo("status")
public Message<OrderStatus> processOrder(Order order) {
    // order processing
    return MessageBuilder
        .withPayload(status)
        .setHeader("code", 1234)
        .build();
}
```

`@SendTo`值被假定为`exchange/routingKey`之后的回复exchange和routingKey，其中可以省略其中一个部分。有效值为：

1.foo/bar - the replyTo exchange and routingKey.
2.foo/ - the replyTo exchange and default (empty) routingKey.
3.bar or /bar - the replyTo routingKey and default (empty) exchange.
4./ or empty - the replyTo default exchange and default routingKey.

还可以使用@SendTo而不使用value属性。这种情况等于一个空的sendTo模式。 @SendTo仅在入站邮件没有replyToAddress属性时使用。

从版本1.5开始，@SendTo值可以是一个bean初始化SpEL Expression，例如...

```java
@RabbitListener(queues = "test.sendTo.spel")
@SendTo("#{spelReplyTo}")
public String capitalizeWithSendToSpel(String foo) {
    return foo.toUpperCase();
}
...
@Bean
public String spelReplyTo() {
    return "test.sendTo.reply.spel";
}
```

表达式必须求值为一个String，它可以是一个简单的队列名称(发送到默认exchange)，也可以是如上所述的exchange / routingKey形式。

> `#{…}`表达式在初始化时执行一次

对于动态回复路由，消息发送方应包含一个reply_to消息属性或使用下面描述的备用运行时Spel表达式。
从版本1.6开始，@SendTo可以是在运行时针对请求和回复进行评估的Spel表达式：
```java
@RabbitListener(queues = "test.sendTo.spel")
@SendTo("!{'some.reply.queue.with.' + result.queueName}")
public Bar capitalizeWithSendToSpel(Foo foo) {
    return processTheFooAndReturnABar(foo);
}
```

Spel表达式的运行时性质由`!{...}`分隔符指示。表达式的评估上下文`#root`对象具有三个属性：

>- request - o.s.amqp.core.Message请求对象。
- source - 转换后的o.s.messaging.Message<？>
- result - 方法结果。

上下文具有映射属性访问器，标准类型转换器和bean解析器，允许引用上下文中的其他bean(例如`@someBeanName.determineReplyQ(request, result))`)。

总而言之，`#{...}`在初始化期间被评估一次，`#root`对象是应用程序上下文; bean被其名称引用。在运行时对每个消息的`!{...} `进行评估，其中根对象具有上述属性，并且bean以其名称引用，以@为前缀。

**Multi-Method Listeners**
从版本1.5.0开始，现在可以在类级别上指定@RabbitListener注释。与新的@RabbitHandler注释一起，这允许单个监听器基于传入消息的有效载荷类型来调用不同的方法。这最好用一个例子来描述：
```java
@RabbitListener(id="multi", queues = "someQueue")
public class MultiListenerBean {

    @RabbitHandler
    @SendTo("my.reply.queue")
    public String bar(Bar bar) {
        ...
    }

    @RabbitHandler
    public String baz(Baz baz) {
        ...
    }

    @RabbitHandler
    public String qux(@Header("amqp_receivedRoutingKey") String rk, @Payload Qux qux) {
        ...
    }

}
```

在这种情况下，如果转换的有效载荷是Bar，Baz或Qux，则会调用单独的@RabbitHandler方法。重要的是要了解系统必须能够基于有效载荷类型识别唯一的方法。检查类型是否具有不具有注释的单个参数的可分配性，或者使用@Payload注释进行注释。请注意，相同的方法签名适用于上述方法级@RabbitListener中讨论的方法。

请注意，必须在每个方法(如果需要)上指定@SendTo;它在类级别不支持。

**@Repeatable @RabbitListener**
从版本1.6开始，@RabbitListener注释被标记为@Repeatable。这意味着注释可以多次出现在相同的注释元素(方法或类)上。在这种情况下，为每个注释创建一个单独的监听器容器，每个注释都调用相同的监听器@Bean。可重复注释可与Java 8或更高版本一起使用;当使用Java 7或更早版本时，通过使用@RabbitListeners“容器”注释可以获得与@RabbitListener注释数组相同的效果。

**Proxy @RabbitListener and Generics**
如果您的服务旨在被代理(例如，在@Transactional的情况下)，当接口具有通用参数时，有一些注意事项。具有通用界面和特定实现，例如：

```java
interface TxService<P> {

   String handle(P payload, String header);

}

static class TxServiceImpl implements TxService<Foo> {

    @Override
    @RabbitListener(...)
    public String handle(Foo foo, String rk) {
         ...
    }

}
```
您必须切换到CGLIB目标类代理，因为实际实现的接口句柄方法是一种桥接方法。在事务管理的情况下，使用注释选项 -  @EnableTransactionManagement(proxyTargetClass = true)来配置CGLIB的使用。在这种情况下，所有注释必须在实现中的目标方法上声明：

```java
static class TxServiceImpl implements TxService<Foo> {

    @Override
    @Transactional
    @RabbitListener(...)
    public String handle(@Payload Foo foo, @Header("amqp_receivedRoutingKey") String rk) {
        ...
    }

}
```

**Container Management**
为注释创建的容器未在应用程序上下文中注册。您可以通过调用`RabbitListenerEndpointRegistry` bean上的`getListenerContainers()`来获取所有容器的集合。然后，您可以遍历此集合，例如，停止/启动所有容器或调用注册表本身的Lifecycle方法，这将调用每个容器上的操作。

您还可以使用其id来获取对单个容器的引用，使用`getListenerContainer(String id)`;例如上面的代码段创建的容器的`registry.getListenerContainer(“multi”)`。

从1.5.2版开始，您可以使用`getListenerContainerIds()`获取注册容器的id。

从1.5版开始，您现在可以将组分配给RabbitListener端点上的容器。这提供了一种获取对容器子集的引用的机制;添加一个组属性会使一个类型为`Collection<MessageListenerContainer>`的bean注册到具有组名称的上下文中。

##### Threading and Asynchronous Consumers(线程和异步消费者)
异步消费者涉及到多线程。

在SimpleMessageListener中配置的TaskExecutor的线程用于在RabbitMQ Client发送新消息时调用MessageListener。如果未配置，则使用SimpleAsyncTaskExecutor。如果使用了线程池，请确保池大小足以处理配置的并发。

> 当使用默认的SimpleAsyncTaskExecutor时，对于调用监听器的线程，监听器容器beanName用作threadNamePrefix。这对日志分析很有用;通常建议在日志追踪器配置中始终包含线程名称。当TaskExecutor通过SimpleMessageListenerContainer上的taskExecutor属性特别提供时，它将按原样使用，无需修改。建议您使用类似的技术来命名由自定义TaskExecutor bean定义创建的线程，以帮助日志消息中的线程标识。

在CachingConnectionFactory中配置的执行程序在创建连接时被传递到RabbitMQ客户端，其线程用于将新消息传递给监听器容器。在撰写本文时，如果未配置，客户端将使用池大小为5的内部线程池执行程序。

RabbitMQ客户端使用ThreadFactory创建用于低级I / O(套接字)操作的线程。要修改此工厂，您需要配置底层RabbitMQ ConnectionFactory，如“Configuring the Underlying Client Connection Factory(配置底层客户端连接工厂)”一节中所述。

##### Detecting Idle Asynchronous Consumers(检测空闲异步消费者)

虽然效率高， 但是异步消费者的一个问题是检测他们什么时候空闲 - 如果没有消息到达一段时间，用户可能需要采取一些行动。

从版本1.6开始，现在可以将监听器容器配置为发布ListenerContainerIdleEvent，当有一段时间没有消息传递。当容器空闲时，每个idleEventInterval毫秒将发布一个事件。

要配置此功能，请在容器上设置idleEventInterval：
xml
```xml
<rabbit:listener-container connection-factory="connectionFactory"
        ...
        idle-event-interval="60000"
        ...
        >
    <rabbit:listener id="container1" queue-names="foo" ref="myListener" method="handle" />
</rabbit:listener-container>
```
Java
```java
@Bean
public SimpleMessageListenerContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    ...
    container.setIdleEventInterval(60000L);
    ...
    return container;
}
```

@RabbitListener
```java
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(rabbitConnectionFactory());
    factory.setIdleEventInterval(60000L);
    ...
    return factory;
}
```

在每种情况下，容器空闲时，每分钟会发布一次事件。

**Event Consumption**
您可以通过实现ApplicationListener来捕获这些事件，无论是通用监听器还是收件人，只能收到此特定事件。您还可以使用Spring Framework 4.2中引用的@EventListener。

以下示例将@RabbitListener和@EventListener组合到一个类中。了解应用程序监听器将获取所有容器的事件非常重要，因此如果要根据哪个容器空闲来执行特定操作，则可能需要检查监听器ID。您也可以为此使用@EventListener条件。

事件有4个属性：
>- source 监听器容器实例
- id 监听器id(或容器bean名称)
- idleTime 当事件发布时，容器空闲的时间
- queueNames 容器监听到的队列的名称

```java
public class Listener {

    @RabbitListener(id="foo", queues="#{queue.name}")
    public String listen(String foo) {
        return foo.toUpperCase();
    }

    @EventListener(condition = "event.listenerId == 'foo'")
    public void onApplicationEvent(ListenerContainerIdleEvent event) {
        ...
    }

}
```

> 注意
事件听众将看到所有容器的事件;因此，在上面的示例中，我们缩小了基于监听器ID接收的事件。
警告
如果要使用空闲事件来停止lister容器，则不应在调用监听器的线程上调用container.stop()，这将导致延迟和不必要的日志消息。相反，您应该将事件切换到另一个线程，然后可以停止容器。

#### 3.1.7 Message Converters(消息转换器)
##### 介绍
AmqpTemplate还定义了几种用于发送和接收将委托给MessageConverter的消息的方法。 MessageConverter本身很简单。它为每个方向提供单一方法：一种用于转换为消息，另一种用于从消息转换。请注意，转换为消息时，除了对象之外，还可以提供属性。 “object”参数通常对应于消息体。

```java
public interface MessageConverter {

    Message toMessage(Object object, MessageProperties messageProperties)
            throws MessageConversionException;

    Object fromMessage(Message message) throws MessageConversionException;

}
```

以下列出了AmqpTemplate中的相关消息发送方法。它们比以前讨论的方法简单，因为它们不需要Message实例。相反，MessageConverter负责通过将提供的对象转换为消息体的字节数组，然后添加任何提供的MessageProperties来“创建”每个消息。

```java
void convertAndSend(Object message) throws AmqpException;

void convertAndSend(String routingKey, Object message) throws AmqpException;

void convertAndSend(String exchange, String routingKey, Object message)
    throws AmqpException;

void convertAndSend(Object message, MessagePostProcessor messagePostProcessor)
    throws AmqpException;

void convertAndSend(String routingKey, Object message,
    MessagePostProcessor messagePostProcessor) throws AmqpException;

void convertAndSend(String exchange, String routingKey, Object message,
    MessagePostProcessor messagePostProcessor) throws AmqpException;
```

在接收端，只有两种方法：一种接受队列名称，一种依赖于模板的“队列”属性已被设置。

```java
Object receiveAndConvert() throws AmqpException;

Object receiveAndConvert(String queueName) throws AmqpException;
```

> “异步消费者”一节中提到的MessageListenerAdapter也使用MessageConverter。

##### SimpleMessageConverter
MessageConverter策略的默认实现称为SimpleMessageConverter。如果您没有显式配置替代方案，这将是RabbitTemplate实例使用的转换器。它处理基于文本的内容，序列化的Java对象和简单的字节数组。

**Converting From a Message**
如果输入消息的内容类型以“text”(例如“text / plain”)开头，它还将检查content-encoding属性，以确定将Message body字节数组转换为Java String时要使用的字符集。如果在输入消息中未设置content-encoding属性，则默认情况下将使用“UTF-8”字符集。如果需要覆盖该默认设置，可以配置SimpleMessageConverter的实例，设置其“defaultCharset”属性，然后将其注入到RabbitTemplate实例中。

如果输入Message的content-type属性值被设置为“application / x-java-serialized-object”，那么SimpleMessageConverter将尝试将字节数组反序列化(再水化)为Java对象。虽然这对于简单的原型设计可能是有用的，但一般不建议依赖Java序列化，因为它会导致生产者和消费者之间的紧密耦合。当然，它也排除了非Java系统在任何一方的使用。使用AMQP作为线程协议，不幸的是，通过这样的限制可以损失大部分优势。在接下来的两节中，我们将探讨一些传递丰富域对象内容的方法，而不依赖于Java序列化。

对于所有其他内容类型，SimpleMessageConverter将直接将消息体内容作为字节数组返回。

有关重要信息，请参阅“Java Deserialization”一节。

**Converting To a Message**
当从任意Java对象转换为消息时，SimpleMessageConverter同样处理字节数组，字符串和可序列化实例。它会将每个字节转换为字节(在字节数组的情况下，没有任何转换)，并且将相应地设置content-type属性。如果要转换的对象与这些类型不匹配，则消息体将为空。

##### SerializerMessageConverter
该转换器类似于SimpleMessageConverter，除了可以使用其他Spring Framework Serializer和Deserializer实现进行配置，以实现应用程序/ x-java-serialized-object转换。

##### Jackson2JsonMessageConverter
**Converting to a Message**
如上一节所述，通常不推荐依靠Java序列化。一种比较常见的替代方法是在不同语言和平台上更加灵活和便携，是JSON(JavaScript Object Notation)。可以在任何RabbitTemplate实例上配置转换器，以覆盖其SimpleMessageConverter默认值的使用。 Jackson2JsonMessageConverter使用com.fasterxml.jackson 2.x库。

```xml
<bean class="org.springframework.amqp.rabbit.core.RabbitTemplate">
    <property name="connectionFactory" ref="rabbitConnectionFactory"/>
    <property name="messageConverter">
        <bean class="org.springframework.amqp.support.converter.Jackson2JsonMessageConverter">
            <!-- if necessary, override the DefaultClassMapper -->
            <property name="classMapper" ref="customClassMapper"/>
        </bean>
    </property>
</bean>
```

如上所示，Jackson2JsonMessageConverter默认使用DefaultClassMapper。类型信息被添加到MessageProperties(并从中检索)。如果入站邮件在MessageProperties中不包含类型信息，但您知道预期的类型，则可以使用defaultType属性配置静态类型

```xml
<bean id="jsonConverterWithDefaultType"
      class="o.s.amqp.support.converter.Jackson2JsonMessageConverter">
    <property name="classMapper">
        <bean class="org.springframework.amqp.support.converter.DefaultClassMapper">
            <property name="defaultType" value="foo.PurchaseOrder"/>
        </bean>
    </property>
</bean>
```

**Converting from a Message**
根据发送系统添加到标题的类型信息，将入站消息转换为对象。

在1.6之前的版本中，如果类型信息不存在，转换将失败。从版本1.6开始，如果类型信息丢失，转换器将使用Jackson默认值(通常为map)转换JSON。

此外，从版本1.6开始，当使用@RabbitListener注释(在方法上)时，推断的类型信息将添加到MessageProperties中;这允许转换器转换为目标方法的参数类型。这仅适用于没有注释的一个参数或@Payload注释的单个参数。分析期间忽略消息类型的参数。

注意：
> 默认情况下，推断的类型信息将覆盖由发送系统创建的入站`__TypeId__`和相关头。这允许接收系统自动转换成不同的域对象。这仅适用于参数类型为具体(不是抽象或接口)或来自java.util包的情况。在所有其他情况下，将使用`__TypeId__`和相关的标题。在某些情况下，您可能希望覆盖默认行为，并始终使用`__TypeId__`信息。例如，假设你有一个`@RabbitListener`，它接受一个Foo参数，但该消息包含一个Bar，它是Foo的一个子类(这是具体的)。推断的类型将不正确。为了处理这种情况，将`Jackson2JsonMessageConverter`上的`TypePrecedence`属性设置为`TYPE_ID`，而不是默认的`INFERRED`。该属性实际上是在转换器的`DefaultJackson2JavaTypeMapper`上，但为了方便起见，转换器上提供了一个setter。如果您注入自定义类型的映射器，则应该在映射器上设置属性。


> 从消息转换时，传入的`MessageProperties.getContentType()`必须符合JSON(使用逻辑`contentType.contains(“json”)`)。否则，`WARN`日志消息无法转换带有`content-type[...]`的传入消息，并且发送`message.getBody()` - 作为`byte[]`返回。因此，为了满足消费者方面的`Jackson2JsonMessageConverter`要求，生产者必须添加`contentType`消息属性，例如作为`application/json`，`text/x-json`或者只是使用`Jackson2JsonMessageConverter`，它将自动设置标题。

```java
@RabbitListener
public void foo(Foo foo) {...}

@RabbitListener
public void foo(@Payload Foo foo, @Header("amqp_consumerQueue") String queue) {...}

@RabbitListener
public void foo(Foo foo, o.s.amqp.core.Message message) {...}

@RabbitListener
public void foo(Foo foo, o.s.messaging.Message<Foo> message) {...}

@RabbitListener
public void foo(Foo foo, String bar) {...}

@RabbitListener
public void foo(Foo foo, o.s.messaging.Message<?> message) {...}
```
在前四个情况下，转换器将尝试转换为Foo类型。第五个例子是无效的，因为我们无法确定哪个参数应该接收消息有效载荷。在第六个例子中，由于通用类型是通配符，Jackson的默认值将会被应用。

但是，您可以创建一个自定义转换器，并使用targetMethod消息属性来决定将JSON转换为哪种类型。

注意
> 只有在方法级别声明@RabbitListener注释时，才能实现此类型推断。使用类级别的@RabbitListener，转换后的类型用于选择要调用哪个@RabbitHandler方法。因此，基础架构提供了可以由自定义转换器用于确定类型的targetObject消息属性。

##### MarshallingMessageConverter
另一个选择是MarshallingMessageConverter。它委托Spring OXM库的Marshaller和Unmarshaller策略接口的实现。在配置方面，最常见的是提供构造函数参数，因为Marshaller的大多数实现也将实现Unmarshaller。

```xml
<bean class="org.springframework.amqp.rabbit.core.RabbitTemplate">
    <property name="connectionFactory" ref="rabbitConnectionFactory"/>
    <property name="messageConverter">
        <bean class="org.springframework.amqp.support.converter.MarshallingMessageConverter">
            <constructor-arg ref="someImplemenationOfMarshallerAndUnmarshaller"/>
        </bean>
    </property>
</bean>
```

##### ContentTypeDelegatingMessageConverter
该类在1.4.2版本中引入，并允许根据MessageProperties中的内容类型属性委派给特定的MessageConverter。默认情况下，如果没有contentType属性，或者与没有配置的转换器匹配的值，它将委托给SimpleMessageConverter。

```xml
<bean id="contentTypeConverter" class="ContentTypeDelegatingMessageConverter">
    <property name="delegates">
        <map>
            <entry key="application/json" value-ref="jsonMessageConverter" />
            <entry key="application/xml" value-ref="xmlMessageConverter" />
        </map>
    </property>
</bean>
```

##### Java Deserialization
注意
> 从不受信任的源反序列化java对象时，可能存在一个漏洞。如果您使用`content-type` `application/x-java-serialized-object`接受来自不受信任来源的邮件，则应考虑配置哪些包/类被允许反序列化。这适用于`SimpleMessageConverter`和`SerializerMessageConverter`，当它被配置为使用`DefaultDeserializer`  - 隐式或通过配置。默认情况下，白名单为空，表示所有类将被反序列化。您可以设置模式列表，如`foo.*`，`foo.bar.Baz`或`*.MySafeClass`。将按顺序检查模式，直到找到匹配项。如果没有匹配，则抛出`SecurityException`。使用这些转换器上的`whiteListPatterns`属性设置模式。

##### Message Properties Converters
MessagePropertiesConverter策略接口用于在Rabbit Client BasicProperties和Spring AMQP MessageProperties之间进行转换。默认实现(DefaultMessagePropertiesConverter)通常足以满足大多数用途，但如果需要，您可以实现自己的。当大小不大于1024字节时，默认属性转换器将将LongString类型的BasicProperties元素转换为String。较大的LongString不会转换(见下文)。可以使用构造函数参数覆盖此限制。

从版本1.6开始，长度超过长字符串限制的标题(默认1024)现在默认由DefaultMessagePropertiesConverter保留为LongString。您可以通过`getBytes[]`，`toString()`或`getStream()`方法访问内容。

以前，DefaultMessagePropertiesConverter将这样的头部转换为DataInputStream(实际上它只是引用了LongString的DataInputStream)。在输出时，该标头未被转换(除了一个字符串，例如，通过调用流上的toString()，java.io.DataInputStream@1d057a39)。

大量传入的LongString headers现在在输出上也被正确地“转换”(默认情况下)。

提供了一个新的构造函数，允许您将转换器配置为如前所述：

```java
/**
 * Construct an instance where LongStrings will be returned
 * unconverted or as a java.io.DataInputStream when longer than this limit.
 * Use this constructor with 'true' to restore pre-1.6 behavior.
 * @param longStringLimit the limit.
 * @param convertLongLongStrings LongString when false,
 * DataInputStream when true.
 * @since 1.6
 */
public DefaultMessagePropertiesConverter(int longStringLimit, boolean convertLongLongStrings) { ... }
```

从1.6版开始，一个新的属性correlationIdString已经添加到MessageProperties中。以前，当转换为RabbitMQ客户端使用的BasicProperties时，会执行不必要的`byte[]<->String`转换，因为MessageProperties.correlationId是一个byte[]，但是BasicProperties使用一个String。 (最终，RabbitMQ客户端使用UTF-8将String转换为字节以输入协议消息)。

为了提供最大的向后兼容性，将新的属性correlationIdPolicy添加到DefaultMessagePropertiesConverter中。这需要一个DefaultMessagePropertiesConverter.CorrelationIdPolicy枚举参数。默认情况下，它被设置为BYTES，它复制以前的行为。

入站消息：
> - STRING - 只有correlationIdString属性被映射
- BYTES - 只有correlationId属性被映射
- BOTH - 两个属性都被映射

出站消息：
> - STRING - 只有correlationIdString属性被映射
- BYTES - 只有correlationId属性被映射
- BOTH - 两个属性将被考虑，String属性优先

从版本1.6开始，入站deliveryMode属性不再映射到MessageProperties.deliveryMode，而是映射到MessageProperties.receivedDeliveryMode。此外，入站userId属性不再映射到MessageProperties.userId，而是映射到MessageProperties.receivedUserId。如果将相同的MessageProperties对象用于出站消息，则这些更改是为了避免这些属性的意外传播。

#### 3.1.8 Modifying Messages - Compression and More(修改消息、压缩)
可以在注解配置的消息接收之前或消息发送之后对消息内容进行修改

从第3.1.7节“消息转换器”可以看出，在`AmqpTemplate` `convertAndReceive`操作中，您可以提供一个`MessagePostProcessor`。
例如，在您的POJO被转换之后，`MessagePostProcessor`使您能够在消息中设置自定义headers或properties。

从版本1.4.2开始，其他扩展点已添加到`RabbitTemplate` - `setBeforePublishPostProcessors()`和`setAfterReceivePostProcessors()`中。第一个是后台处理器在发送到RabbitMQ之前立即运行。当使用批处理(参见“批处理”一节)时，会在批量组装之后并在发送批处理之前调用该批处理。第二个是在收到消息后立即调用。

这些扩展点用于压缩等功能，为此，提供了几个MessagePostProcessor：

用于消息发送之前处理的
>- GZipPostProcessor
- ZipPostProcessor

用于消息接收之前处理的
>- GUnzipPostProcessor
- UnzipPostProcessor

`SimpleMessageListenerContainer`也有一个`setAfterReceivePostProcessors()`方法，允许在容器接收到消息之后执行解压缩。

#### 3.1.9 Request/Reply Messaging(请求/回复消息)
##### 介绍
`AmqpTemplate`还提供了各种`sendAndReceive`方法，它们接受与单向发送操作(exchange，routingKey和Message)相同的参数选项。这些方法对于请求/回复方案非常有用，因为它们在发送之前处理必要的“回复”属性的配置，并且可以在为此目的在内部创建的排他队列上监听回复消息。

类似的请求/应答MessageConverter的方法也可以应用于请求和回复。
这些方法是convertSendAndReceive命名。有关更多详细信息，请参阅AmqpTemplate的Javadoc。

##### Reply Timeout(回复超时)
默认情况下，发送和接收方法将在5秒后超时，并返回null。这可以通过设置
`replyTimeout`属性来修改。从版本1.5开始，如果将`mandatory`属性设置为true(或`mandatory-expression`对特定消息计算为true)，则如果消息无法传递到队列，则将抛出`AmqpMessageReturnedException`。这个异常已经返回了`Message`，`replyCode`，`replyText`属性，以及用于发送的`exchange`和`routingKey`。

> 此功能使用发布者返回，并通过在CachingConnectionFactory上将publisherReturns设置为true来启用(请参阅“Publisher Confirms and Returns”一节)。此外，您不能在RabbitTemplate中注册自己的ReturnCallback

##### RabbitMQ Direct reply-to(RabbitMQ直接回复)
> 从版本3.4.0开始，RabbitMQ服务器现在支持直接回复;这是取消固定应答队列的主要原因(以避免为每个请求创建一个临时队列)。从Spring AMQP版本1.4.1开始，默认情况下将使用直接回复(如果服务器支持)，而不是创建临时回复队列。当没有提供replyQueue(或者使用名称amq.rabbitmq.reply-to设置)时，RabbitTemplate将自动检测是否支持Direct reply-to，并使用它或回退到使用临时回复队列。当使用直接回复时，不需要回复监听器，不需要配置。

命名队列(不同于amq.rabbitmq.reply-to)支持回复监听器，允许控制回复并发等。

从1.6版开始，如果由于某些原因希望为每个回复使用临时的、排他性的自动删除队列时，请将useTemporaryReplyQueues属性设置为true。如果您设置了replyAddress，则此属性将被忽略。

可以通过对RabbitTemplate进行子类化并覆盖useDirectReplyTo()来更改是否使用直接回复。该方法只被调用一次;在发送第一个请求时。

##### Message Correlation With A Reply Queue(与回复队列的消息相关)
当使用固定的应答队列(不同于amq.rabbitmq.reply-to)时，需要提供相关数据，以便可以将请求与请求相关联。请参阅RabbitMQ远程过程调用(RPC)。默认情况下，标准correlationId属性将用于保存相关数据。但是，如果要使用自定义属性来保存关联数据，可以在<rabbit-template />上设置correlation-key属性。将属性显式设置为correlationId与默认属性相同。当然，客户端和服务器必须使用相同的头相关数据。

> Spring AMQP版本1.1对此数据使用了一个自定义属性spring_reply_correlation。如果您希望使用当前版本恢复此行为，或许要使用1.1保持与其他应用程序的兼容性，则必须将属性设置为spring_reply_correlation。

##### Reply Listener Container(回复监听容器)
在3.4.0之前使用RabbitMQ版本时，每个回复都使用一个新的临时队列。但是，可以在模板上配置单个应答队列，这样可以更有效，并且还可以在该队列中设置参数。但是，在这种情况下，您还必须提供一个`<reply-listener />`子元素。此元素为回复队列提供监听器容器，模板为监听器。除了连接工厂和消息转换器之外，元素上允许使用`<listener-container />`上允许的所有3.1.15节“Message Listener容器配置”属性，该属性从模板的配置继承。

> 如果您运行多个应用程序实例或使用多个RabbitTemplate，则必须为每个RabbitTemplate使用唯一的应答队列，否则RabbitMQ无法从队列中选择消息，因此，如果它们都使用相同的队列，则每个实例将竞争回复而不一定收到自己的。

```xml
<rabbit:template id="amqpTemplate"
        connection-factory="connectionFactory"
        reply-queue="replies"
        reply-address="replyEx/routeReply">
    <rabbit:reply-listener/>
</rabbit:template>
```

虽然容器和模板共享连接工厂，但它们不共享通道，因此请求和回复不会在同一事务中执行(如果是事务性的)。

> 在版本1.5.0之前，`reply-address`属性不可用，回复始终使用默认exchage和reply-queue名称作为routing key。这仍然是默认值，但您现在可以指定新的reply-address属性。reply-address可以包含一个形式为`<exchange>` / `<routingKey>`的地址，并且回复将被路由到指定的交换机并路由到与routing key绑定的队列。reply-address优先于reply-queue。必须将`<reply-listener>`配置为单独的`<listener-container>`组件，当只有reply-address正在使用时，无论如何，reply-address和reply-queue(或`<listener-container>`上的队列属性)必须引用在逻辑上相同的队列。

在这个配置中,我们使用SimpleListenerContainer收到回复;当使用`<rabbit：template />`定义模板时，如上所示，解析器将模板中的 container和wires定义为监听器。

> 当模板不使用固定的replyQueue(或正在使用Direct reply-to  - 请参阅“RabbitMQ Direct reply-to”一节)时，不需要监听器容器。直接回复是使用RabbitMQ 3.4.0或更高版本的首选机制。

如果您将RabbitTemplate定义为`<bean />`，或者使用`@Configuration`类将其定义为`@Bean`，或者以编程方式创建模板，则需要自己定义并连接回复监听器容器。如果您无法执行此操作，模板将永远不会收到回复，并将最终超时并返回null作为对`sendAndReceive`方法的调用的回复。

从1.5版开始，RabbitTemplate将检测它是否被配置为MessageListener以接收回复。如果没有，尝试发送和接收具有回复地址的消息将失败，并显示IllegalStateException(因为不会收到回复)。

此外，如果使用一个简单的replyAddress(队列名称)，那么回复监听器容器将验证它正在监听具有相同名称的队列。如果回复地址是exchange和routing key，并且调试日志消息将被写入，则无法执行此检查。

> 当您自己接收回复监听器和模板时，确保模板的replyQueue和容器的队列(或queueNames)属性引用相同的队列是非常重要的。模板将回复队列插入到出站消息replyTo属性中。

以下是如何手动连接bean的示例。

```xml
<bean id="amqpTemplate" class="org.springframework.amqp.rabbit.core.RabbitTemplate">
    <constructor-arg ref="connectionFactory" />
    <property name="exchange" value="foo.exchange" />
    <property name="routingKey" value="foo" />
    <property name="replyQueue" ref="replyQ" />
    <property name="replyTimeout" value="600000" />
</bean>

<bean class="org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer">
    <constructor-arg ref="connectionFactory" />
    <property name="queues" ref="replyQ" />
    <property name="messageListener" ref="amqpTemplate" />
</bean>

<rabbit:queue id="replyQ" name="my.reply.queue" />
```

```java
    @Bean
    public RabbitTemplate amqpTemplate() {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory());
        rabbitTemplate.setMessageConverter(msgConv());
        rabbitTemplate.setReplyQueue(replyQueue());
        rabbitTemplate.setReplyTimeout(60000);
        return rabbitTemplate;
    }

    @Bean
    public SimpleMessageListenerContainer replyListenerContainer() {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory());
        container.setQueues(replyQueue());
        container.setMessageListener(amqpTemplate());
        return container;
    }

    @Bean
    public Queue replyQueue() {
        return new Queue("my.reply.queue");
    }

```

该[测试用例](https://github.com/spring-projects/spring-amqp/tree/master/spring-rabbit/src/test/java/org/springframework/amqp/rabbit/listener/JavaConfigFixedReplyQueueTests.java)显示了一个连接了固定应答队列的RabbitTemplate的完整示例，以及处理请求并返回答复的“远程”监听器容器。

> 当回复超时(replyTimeout)时，sendAndReceive()方法返回null。

在版本1.3.6之前，只是记录了超时消息的迟到回复。现在，如果收到迟到的回复，它将被拒绝(模板抛出一个`AmqpRejectAndDontRequeueException`)。如果回复队列被配置为将拒绝的消息发送到dead letter exchange，则可以检索回复以供稍后分析。只需使用等于回复queue名称的routing key将队列绑定到配置的dead letter exchange中。

有关配置dead lettering的更多信息，请参阅[ RabbitMQ Dead Letter Documentation](https://www.rabbitmq.com/dlx.html)。您还可以查看一个示例的`FixedReplyQueueDeadLetterTests`测试用例。

##### AsyncRabbitTemplate
版本1.6引入了`AsyncRabbitTemplate`。这与`AmqpTemplate`中的`sendAndReceive`(和`convertSendAndReceive`)类似，但是它们返回一个`ListenableFuture`。

您可以稍后通过调用`get()`来同步检索结果，也可以注册一个将结果异步调用的回调。

```java
@Autowired
private AsyncRabbitTemplate template;

...

public void doSomeWorkAndGetResultLater() {

    ...

    ListenableFuture<String> future = this.template.convertSendAndReceive("foo");

    // do some more work

    String reply = null;
    try {
        reply = future.get();
    }
    catch (ExecutionException e) {
        ...
    }

    ...

}

public void doSomeWorkAndGetResultAsync() {

    ...

    RabbitConverterFuture<String> future = this.template.convertSendAndReceive("foo");
    future.addCallback(new ListenableFutureCallback<String>() {

        @Override
        public void onSuccess(String result) {
            ...
        }

        @Override
        public void onFailure(Throwable ex) {
            ...
        }

    });

    ...

}
```
如果设置了`mandatory`，并且消息无法传递，则将来会抛出一个执行错误的`ExecutionException`，其原因是`AmqpMessageReturnedException`封装返回的消息和有关返回的信息。

如果已设置enableConfirms，未来将具有一个属性确认，它本身是一个`ListenableFuture <Boolean>`，表示成功发布。如果确认的未来是false，RabbitFuture将有另一个属性nackCause(失败的原因(如果有的话))。

> 如果在回复后收到发件人确认，则会被丢弃 - 因为回复意味着成功发布。

设置模板上的receiveTimeout属性以配置超时回复(默认为30000  -  30秒)。如果发生超时，将来将使用AmqpReplyTimeoutException完成。

该模板实现SmartLifecycle;在有待处理的回复的情况下停止模板将导致未处理的Future被取消。

##### Spring Remoting with AMQP
Spring框架具有一般的远程处理能力，允许使用各种传输的远程过程调用(RPC)。 Spring-AMQP支持与客户端上的AmqpProxyFactoryBean和服务器上的AmqpInvokerServiceExporter类似的机制。这提供了RPC over AMQP。在客户端，如上所述使用RabbitTemplate;在服务器端，调用者(配置为MessageListener)接收消息，调用配置的服务，并使用入站消息的replyTo信息返回回复。

客户端工厂bean可以注入任何bean(使用其serviceInterface);然后，客户端可以在代理上调用方法，导致通过AMQP进行远程执行。

> 使用默认MessageConverter，方法参数和返回值必须是Serializable的实例。

在服务器端，AmqpInvokerServiceExporter具有AmqpTemplate和MessageConverter属性。目前，模板的MessageConverter没有被使用。如果您需要提供自定义消息转换器，那么您应该使用messageConverter属性提供它。在客户端，可以将自定义消息转换器添加到使用其amqpTemplate属性提供给AmqpProxyFactoryBean的AmqpTemplate中。

客户端和服务器配置示例如下所示。
```xml
<bean id="client"
	class="org.springframework.amqp.remoting.client.AmqpProxyFactoryBean">
	<property name="amqpTemplate" ref="template" />
	<property name="serviceInterface" value="foo.ServiceInterface" />
</bean>

<rabbit:connection-factory id="connectionFactory" />

<rabbit:template id="template" connection-factory="connectionFactory" reply-timeout="2000"
	routing-key="remoting.binding" exchange="remoting.exchange" />

<rabbit:admin connection-factory="connectionFactory" />

<rabbit:queue name="remoting.queue" />

<rabbit:direct-exchange name="remoting.exchange">
	<rabbit:bindings>
		<rabbit:binding queue="remoting.queue" key="remoting.binding" />
	</rabbit:bindings>
</rabbit:direct-exchange>
```

```xml
<bean id="listener"
	class="org.springframework.amqp.remoting.service.AmqpInvokerServiceExporter">
	<property name="serviceInterface" value="foo.ServiceInterface" />
	<property name="service" ref="service" />
	<property name="amqpTemplate" ref="template" />
</bean>

<bean id="service" class="foo.ServiceImpl" />

<rabbit:connection-factory id="connectionFactory" />

<rabbit:template id="template" connection-factory="connectionFactory" />

<rabbit:queue name="remoting.queue" />

<rabbit:listener-container connection-factory="connectionFactory">
	<rabbit:listener ref="listener" queue-names="remoting.queue" />
</rabbit:listener-container>
```

> AmqpInvokerServiceExporter只能处理正确的消息，例如从AmqpProxyFactoryBean发送的消息。如果收到不能解释的消息，则将序列化的RuntimeException作为回复发送。如果该消息没有replyToAddress属性，且未配置Dead Letter Exchange，该消息将被拒绝并永久丢失。

<br/>
> 默认情况下，如果请求消息无法传递，则调用线程将最终超时，并将抛出RemoteProxyFailureException。超时时间默认为5秒，可以通过设置RabbitTemplate上的replyTimeout属性进行修改。从1.5版开始，将mandatory属性设置为true，并在连接工厂启用返回(请参阅“发布者确认和返回”一节)，调用线程将抛出一个AmqpMessageReturnedException。有关详细信息，请参阅“回复超时”一节。

#### 3.1.10 Configuring the broker
##### 介绍
AMQP规范描述了如何使用协议来配置代理上的队列，交换和绑定。这些可从0.8规范和更高版本移植的操作存在于org.springframework.amqp.core包中的AmqpAdmin界面中。该类的RabbitMQ实现是位于org.springframework.amqp.rabbit.core包中的RabbitAdmin。

AmqpAdmin接口基于使用Spring AMQP域抽象，如下所示：
```java
public interface AmqpAdmin {

    // Exchange Operations

    void declareExchange(Exchange exchange);

    void deleteExchange(String exchangeName);

    // Queue Operations

    Queue declareQueue();

    String declareQueue(Queue queue);

    void deleteQueue(String queueName);

    void deleteQueue(String queueName, boolean unused, boolean empty);

    void purgeQueue(String queueName, boolean noWait);

    // Binding Operations

    void declareBinding(Binding binding);

    void removeBinding(Binding binding);

    Properties getQueueProperties(String queueName);

}
```

getQueueProperties()方法返回有关队列的一些有限信息(消息计数和消费者计数)。返回的属性的键可用作RabbitTemplate(QUEUE_NAME，QUEUE_MESSAGE_COUNT，QUEUE_CONSUMER_COUNT)中的常量。 RabbitMQ REST API在QueueInfo对象中提供了更多的信息。

no-arg declareQueue()方法定义了一个自动生成名称的代理上的队列。此自动生成队列的附加属性为exclusive = true，autoDelete = true和durable = false。

declareQueue(Queue queue)方法接受Queue对象并返回声明队列的名称。如果提供的Queue的name属性是空字符串，则代理使用生成的名称声明队列，并将该名称返回给调用者。队列对象本身没有改变。此功能只能通过直接调用RabbitAdmin以编程方式使用。管理员自动声明不支持在应用程序上下文中声明性地定义队列。

这与AnonymousQueue形成对照，其中框架生成一个唯一的(UUID)名称，并将持久性设置为false和exclusive，autoDelete为true。具有空或缺少的name属性的`<rabbit：queue />`将始终创建AnonymousQueue。

请参阅“AnonymousQueue”部分了解为什么AnonymousQueue优先于代理生成的队列名称，以及如何控制名称的格式。声明队列必须具有固定名称，因为它们可能在上下文中的其他位置被引用，例如在监听器中：

```xml
<rabbit:listener-container>
    <rabbit:listener ref="listener" queue-names="#{someQueue.name}" />
</rabbit:listener-container>
```

请参考“Automatic Declaration of Exchanges, Queues and Bindings”这一节.

该接口的RabbitMQ实现是RabbitAdmin，当使用Spring XML进行配置时，它将如下所示：
```xml
<rabbit:connection-factory id="connectionFactory"/>

<rabbit:admin id="amqpAdmin" connection-factory="connectionFactory"/>
```

当CachingConnectionFactory缓存模式是CHANNEL(默认值)时，RabbitAdmin实现会在同一个ApplicationContext中声明的Queue，Exchanges和Bindings自动延迟声明。这些组件将被声明为s0on，因为连接已打开到代理。有一些命名空间功能使得这非常方便，例如：

```xml
<rabbit:queue id="tradeQueue"/>

<rabbit:queue id="marketDataQueue"/>

<fanout-exchange name="broadcast.responses"
                 xmlns="http://www.springframework.org/schema/rabbit">
    <bindings>
        <binding queue="tradeQueue"/>
    </bindings>
</fanout-exchange>

<topic-exchange name="app.stock.marketdata"
                xmlns="http://www.springframework.org/schema/rabbit">
    <bindings>
        <binding queue="marketDataQueue" pattern="${stocks.quote.pattern}"/>
    </bindings>
</topic-exchange>
```

在上面的例子中，我们使用匿名队列(实际上内部只是具有由框架生成的名称的队列，而不是broker)，并通过ID引用它们。我们还可以使用显式名称声明队列，这也可以作为上下文中bean定义的标识符。例如。

```xml
<rabbit:queue name="stocks.trade.queue"/>
```
> 您可以同时提供id和name属性。这允许您通过独立于队列名称的id来引用队列(例如在绑定中)。它还允许标准的Spring功能，如属性占位符和队列名称的SpEL表达式;当使用名称作为bean标识符时，这些功能不可用。

可以使用其他参数配置队列，例如x-message-ttl或x-ha-policy。使用命名空间支持，使用`<rabbit：queue-arguments>`元素以参数名称/参数值对映射的形式提供它们。
```xml
<rabbit:queue name="withArguments">
    <rabbit:queue-arguments>
        <entry key="x-ha-policy" value="all"/>
    </rabbit:queue-arguments>
</rabbit:queue>
```

默认情况下，这些参数被假定为字符串。对于其他类型的参数，需要提供类型。
```xml
<rabbit:queue name="withArguments">
    <rabbit:queue-arguments value-type="java.lang.Long">
        <entry key="x-message-ttl" value="100"/>
    </rabbit:queue-arguments>
</rabbit:queue>
```
当提供混合类型的参数时，为每个条目元素提供类型：
```xml
<rabbit:queue name="withArguments">
    <rabbit:queue-arguments>
        <entry key="x-message-ttl">
            <value type="java.lang.Long">100</value>
        </entry>
        <entry key="x-ha-policy" value="all"/>
    </rabbit:queue-arguments>
</rabbit:queue>
```
使用Spring Framework 3.2及更高版本，可以更简洁地声明这一点：
```xml
<rabbit:queue name="withArguments">
    <rabbit:queue-arguments>
        <entry key="x-message-ttl" value="100" value-type="java.lang.Long"/>
        <entry key="x-ha-policy" value="all"/>
    </rabbit:queue-arguments>
</rabbit:queue>
```
>RabbitMQ代理将不允许声明具有不匹配参数的队列。例如，如果`queue`已经存在，没有`time to live`参数，并且尝试使用`key =“x-message-ttl”value =“100”`进行声明，则会抛出异常。

默认情况下，当发生任何异常时，RabbitAdmin将立即停止处理所有声明;这可能会导致下游问题，例如监听器容器无法初始化，因为未声明另一个队列(在错误之后定义)。

可以通过在RabbitAdmin上将ignore-declaration-exceptions属性设置为true来修改此行为。此选项指示RabbitAdmin记录异常，并继续声明其他元素。当使用java配置RabbitAdmin时，此属性为ignoreDeclarationExceptions。这是一个适用于所有元素，队列，交换和绑定的全局设置，具有仅适用于这些元素的类似属性。

在版本1.6之前，此属性仅在通道上发生IOException(例如当前和所需属性不匹配时)才会生效。现在，此属性对任何异常生效，包括TimeoutException等。

另外，任何声明异常将导致发布DeclarationExceptionEvent，它是可以由上下文中任何ApplicationListener使用的ApplicationEvent。该事件包含对管理员的引用，被声明的元素和Throwable。

从版本1.3开始，HeadersExchange可以配置为匹配多个标头;您还可以指定任何或所有标题必须匹配：
```xml
<rabbit:headers-exchange name="headers-test">
    <rabbit:bindings>
        <rabbit:binding queue="bucket">
            <rabbit:binding-arguments>
                <entry key="foo" value="bar"/>
                <entry key="baz" value="qux"/>
                <entry key="x-match" value="all"/>
            </rabbit:binding-arguments>
        </rabbit:binding>
    </rabbit:bindings>
</rabbit:headers-exchange>
```

从版本1.6开始 Exchanges可以使用internal标志(默认为false)进行配置，并且Exchange将通过RabbitAdmin(如果存在于应用程序上下文中)在代理上正确配置。如果交换机的内部标志为true，则RabbitMQ将不允许客户端使用交换机。这对于您不希望交易所直接由发布商使用的死信交换或交换到交换绑定非常有用。

要了解如何使用Java来配置AMQP基础设施，请查看Stock示例应用程序，其中有@Configuration类AbstractStockRabbitConfiguration，它们又具有RabbitClientConfiguration和RabbitServerConfiguration子类。 AbstractStockRabbitConfiguration的代码如下所示

```java
@Configuration
public abstract class AbstractStockAppRabbitConfiguration {

    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory =
            new CachingConnectionFactory("localhost");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        return connectionFactory;
    }

    @Bean
    public RabbitTemplate rabbitTemplate() {
        RabbitTemplate template = new RabbitTemplate(connectionFactory());
        template.setMessageConverter(jsonMessageConverter());
        configureRabbitTemplate(template);
        return template;
    }

    @Bean
    public MessageConverter jsonMessageConverter() {
        return new JsonMessageConverter();
    }

    @Bean
    public TopicExchange marketDataExchange() {
        return new TopicExchange("app.stock.marketdata");
    }

    // additional code omitted for brevity

}
```
在库存应用程序中，服务器使用以下@Configuration类进行配置：
```java
@Configuration
public class RabbitServerConfiguration extends AbstractStockAppRabbitConfiguration  {

    @Bean
    public Queue stockRequestQueue() {
        return new Queue("app.stock.request");
    }
}
```

这是@Configuration类的整个继承链的结尾。最终的结果是在应用程序启动时，TopicExchange和Queue将被声明给代理。在服务器配置中没有将TopicExchange绑定到队列，因为这在客户端应用程序中完成。然而，库存请求队列将自动绑定到AMQP默认交换 - 此行为由规范定义。

客户端@Configuration类有点有趣，如下所示。

```java
@Configuration
public class RabbitClientConfiguration extends AbstractStockAppRabbitConfiguration {

    @Value("${stocks.quote.pattern}")
    private String marketDataRoutingKey;

    @Bean
    public Queue marketDataQueue() {
        return amqpAdmin().declareQueue();
    }

    /**
     * Binds to the market data exchange.
     * Interested in any stock quotes
     * that match its routing key.
     */
    @Bean
    public Binding marketDataBinding() {
        return BindingBuilder.bind(
                marketDataQueue()).to(marketDataExchange()).with(marketDataRoutingKey);
    }

    // additional code omitted for brevity

}
```

客户端通过AmqpAdmin上的declareQueue()方法声明另一个队列，并通过在外部属性文件中的routing pattern 将该队列绑定到market data exchange。

##### Builder API for Queues and Exchanges(构建Queues和Exchanges的API)
版本1.6引入了方便的API，用于在使用Java配置时配置queue和Exchange对象：
```java
@Bean
public Queue queue() {
    return QueueBuilder.nonDurable("foo")
        .autoDelete()
        .exclusive()
        .withArgument("foo", "bar")
        .build();
}

@Bean
public Exchange exchange() {
  return ExchangeBuilder.directExchange("foo")
      .autoDelete()
      .internal()
      .withArgument("foo", "bar")
      .build();
}
```
有关更多信息，请参阅org.springframework.amqp.core.QueueBuilder和org.springframework.amqp.core.ExchangeBuilder的javadocs。

##### Declaring Collections of Exchanges, Queues, Bindings(声明交换、队列、绑定的集合)
从1.5版开始，现在可以通过重新生成一个集合来声明一个@Bean的多个实体。

只考虑第一个元素是可声明的集合，并且只处理来自这些集合的可声明元素。

```java
@Configuration
public static class Config {

    @Bean
    public ConnectionFactory cf() {
        return new CachingConnectionFactory("localhost");
    }

    @Bean
    public RabbitAdmin admin(ConnectionFactory cf) {
        return new RabbitAdmin(cf);
    }

    @Bean
    public DirectExchange e1() {
    	return new DirectExchange("e1", false, true);
    }

    @Bean
    public Queue q1() {
    	return new Queue("q1", false, false, true);
    }

    @Bean
    public Binding b1() {
    	return BindingBuilder.bind(q1()).to(e1()).with("k1");
    }

    @Bean
    public List<Exchange> es() {
    	return Arrays.<Exchange>asList(
    			new DirectExchange("e2", false, true),
    			new DirectExchange("e3", false, true)
    	);
    }

    @Bean
    public List<Queue> qs() {
    	return Arrays.asList(
    			new Queue("q2", false, false, true),
    			new Queue("q3", false, false, true)
    	);
    }

    @Bean
    public List<Binding> bs() {
    	return Arrays.asList(
    			new Binding("q2", DestinationType.QUEUE, "e2", "k2", null),
    			new Binding("q3", DestinationType.QUEUE, "e3", "k3", null)
    	);
    }

    @Bean
    public List<Declarable> ds() {
    	return Arrays.<Declarable>asList(
    			new DirectExchange("e4", false, true),
    			new Queue("q4", false, false, true),
    			new Binding("q4", DestinationType.QUEUE, "e4", "k4", null)
    	);
    }

}
```
##### Conditional Declaration(条件声明)
默认情况下，所有队列，交换和绑定都由应用程序上下文中的所有RabbitAdmin实例(具有auto-startup =“true”)声明。

> 从1.2版本开始，可以有条件地声明这些元素。当应用程序连接到多个代理程序并且需要指定哪个代理程序应该声明特定元素时，这是特别有用的。

代表这些元素的类实现Declarable，它有两个方法：shouldDeclare()和getDeclaringAdmins()。 RabbitAdmin使用这些方法来确定特定实例是否应该实际处理其连接上的声明。

这些属性作为命名空间中的属性，如以下示例所示。
```xml
<rabbit:admin id="admin1" connection-factory="CF1" />

<rabbit:admin id="admin2" connection-factory="CF2" />

<rabbit:queue id="declaredByBothAdminsImplicitly" />

<rabbit:queue id="declaredByBothAdmins" declared-by="admin1, admin2" />

<rabbit:queue id="declaredByAdmin1Only" declared-by="admin1" />

<rabbit:queue id="notDeclaredByAny" auto-declare="false" />

<rabbit:direct-exchange name="direct" declared-by="admin1, admin2">
	<rabbit:bindings>
		<rabbit:binding key="foo" queue="bar"/>
	</rabbit:bindings>
</rabbit:direct-exchange>
```
>默认情况下，auto-declare属性为true，如果未提供声明(或为空)，则所有RabbitAdmin将声明该对象(只要admin的auto-startup属性为true;默认值为true)。

同样，您也可以使用基于Java的@Configuration实现相同的效果。在这个例子中，这些组件将由admin1声明，而不是admin2：
```java
@Bean
public RabbitAdmin admin() {
    RabbitAdmin rabbitAdmin = new RabbitAdmin(cf1());
    rabbitAdmin.afterPropertiesSet();
    return rabbitAdmin;
}

@Bean
public RabbitAdmin admin2() {
    RabbitAdmin rabbitAdmin = new RabbitAdmin(cf2());
    rabbitAdmin.afterPropertiesSet();
    return rabbitAdmin;
}

@Bean
public Queue queue() {
    Queue queue = new Queue("foo");
    queue.setAdminsThatShouldDeclare(admin());
    return queue;
}

@Bean
public Exchange exchange() {
    DirectExchange exchange = new DirectExchange("bar");
    exchange.setAdminsThatShouldDeclare(admin());
    return exchange;
}

@Bean
public Binding binding() {
    Binding binding = new Binding("foo", DestinationType.QUEUE, exchange().getName(), "foo", null);
    binding.setAdminsThatShouldDeclare(admin());
    return binding;
}
```

##### AnonymousQueue
通常，当需要唯一命名的，排他性的自动删除队列时，建议使用AnonymousQueue而不是代理定义的队列名称(使用“”作为队列名称将导致broker生成队列名称) 。

原因如下：

 1. 在建立到代理的连接时，队列实际上是被声明的；这是在bean被创建并连接在一起之后的很长时间；使用队列的bean需要知道它的名字。事实上，当应用程序启动时，代理甚至可能无法运行。
 2. 如果由于某种原因与代理的连接丢失，则管理员将重新声明具有相同名称的AnonymousQueue。如果我们使用代理声明的队列，队列名称将会更改。
 
从1.5.3版开始，您可以控制AnonymousQueue使用的队列名称的格式。

默认情况下，队列名称是UUID的String表示形式;例如：07afcfe9-fe77-4983-8645-0061ec61a47a。

您现在可以在构造函数参数中提供AnonymousQueue.NamingStrategy实现：
```java
@Bean
public Queue anon1() {
    return new AnonymousQueue(new AnonymousQueue.Base64UrlNamingStrategy());
}

@Bean
public Queue anon2() {
    return new AnonymousQueue(new AnonymousQueue.Base64UrlNamingStrategy("foo-"));
}
```

第一个将生成一个前缀为spring.gen-的后缀为UUID的base64表示形式的队列名称，例如：spring.gen-MRBv9sqISkuCiPfOYfpo4g。第二个将生成一个以foo为前缀的队列名称，后跟UUID的base64表示形式。

Base64的编码采用“URL和文件安全的字母“RFC 4648；尾部的填充字符(=)被移除。

您可以提供自己的命名策略，您可以将其他信息(例如应用程序，客户端主机)包含在队列名称中。

从版本1.6开始，可以在使用XML配置时指定命名策略;命名策略属性存在于实现AnonymousQueue.NamingStrategy的bean引用的`<rabbit：queue>`元素中。

```xml
<rabbit:queue id="uuidAnon" />

<rabbit:queue id="springAnon" naming-strategy="springNamer" />

<rabbit:queue id="customAnon" naming-strategy="customNamer" />

<bean id="springNamer" class="org.springframework.amqp.core.AnonymousQueue.Base64UrlNamingStrategy" />

<bean id="customNamer" class="org.springframework.amqp.core.AnonymousQueue.Base64UrlNamingStrategy">
    <constructor-arg value="custom.gen-" />
</bean>
```

第一个创建具有UUID的String表示形式的名称。第二个创建名称如spring.gen-MRBv9sqISkuCiPfOYfpo4g。第三个创建名称，如custom.gen-MRBv9sqISkuCiPfOYfpo4g

当然，您可以提供自己的命名策略bean。

#### 3.1.11 Delayed Message Exchange(延迟消息Exchange)
1.6版引入了对延迟消息exchange插件的支持
> 该插件目前被标记为实验性，但已有一年以上(在撰写本文时)。如果插件的更改需要，我们将尽快添加对这些更改的支持。因此，Spring AMQP中的这种支持也应该被认为是实验性的。该功能使用RabbitMQ 3.6.0和版本0.0.1的插件进行了测试。

要使用RabbitAdmin将延迟声明为exchange，只需将exchange bean上的delayed属性设置为true即可。 RabbitAdmin将使用交换类型(Direct，Fanout等)设置x-delayed-type参数，并使用x-delayed-message类型声明交换。

使用XML配置交换bean时，delayed(默认为false)也可用。
```xml
<rabbit:topic-exchange name="topic" delayed="true" />
```
要发送延迟的消息，只需要通过MessageProperties设置x-delay：
```java
MessageProperties properties = new MessageProperties();
properties.setDelay(15000);
template.send(exchange, routingKey,
        MessageBuilder.withBody("foo".getBytes()).andProperties(properties).build());
```
或者
```java
rabbitTemplate.convertAndSend(exchange, routingKey, "foo", new MessagePostProcessor() {

    @Override
    public Message postProcessMessage(Message message) throws AmqpException {
        message.getMessageProperties().setDelay(15000);
        return message;
    }

});
```

要检查消息是否延迟，请在MessageProperties上使用getReceivedDelay()方法。它是一个单独的属性，以避免意外传播到从输入消息生成的输出消息。

#### 3.1.12 RabbitMQ REST API
启用管理插件后，RabbitMQ服务器公开一个REST API来监视和配置代理。现在提供了API的Java绑定。一般来说，您可以直接使用该API，但是提供了一个方便的包装器来使用熟悉的Spring AMQP队列、Exchange和Binding域对象与API。当直接使用com.rabbitmq.http.client.Client API(分别为QueueInfo，ExchangeInfo和BindingInfo)时，这些对象可以获得更多信息。以下操作在RabbitManagementTemplate上可用：

```java
public interface AmqpManagementOperations {

    void addExchange(Exchange exchange);

    void addExchange(String vhost, Exchange exchange);

    void purgeQueue(Queue queue);

    void purgeQueue(String vhost, Queue queue);

    void deleteQueue(Queue queue);

    void deleteQueue(String vhost, Queue queue);

    Queue getQueue(String name);

    Queue getQueue(String vhost, String name);

    List<Queue> getQueues();

    List<Queue> getQueues(String vhost);

    void addQueue(Queue queue);

    void addQueue(String vhost, Queue queue);

    void deleteExchange(Exchange exchange);

    void deleteExchange(String vhost, Exchange exchange);

    Exchange getExchange(String name);

    Exchange getExchange(String vhost, String name);

    List<Exchange> getExchanges();

    List<Exchange> getExchanges(String vhost);

    List<Binding> getBindings();

    List<Binding> getBindings(String vhost);

    List<Binding> getBindingsForExchange(String vhost, String exchange);

}
```

有关详细信息，请参阅javadocs。

#### 3.1.13 Exception Handling(异常处理器)
使用RabbitMQ Java客户端的许多操作可以抛出已检查的异常。例如，可能会抛出IOExceptions的情况很多。 RabbitTemplate，SimpleMessageListenerContainer和其他Spring AMQP组件将捕获这些异常并将其转换为运行时层次结构中的一个异常。这些在org.springframework.amqp包中定义，AmqpException是层次结构的基础。

当一个监听器抛出一个异常时，它被包装在一个ListenerExecutionFailedException中，通常这个消息被代理拒绝和重新排序。将defaultRequeueRejected设置为false将导致消息被丢弃(或路由到dead letter exchange)。如在“消息监听器和异步事件”一节中所讨论的，监听器可以抛出一个AmqpRejectAndDontRequeueException来有条件地控制这种行为。

但是，有一类错误，监听器无法控制行为。当遇到无法转换的消息(例如无效的content_encoding标头)时，会在消息达到用户代码之前抛出一些异常。将defaultRequeueRejected设置为true(默认)，这些消息将被重新传递。在版本1.3.2之前，用户需要编写一个自定义的ErrorHandler，如3.1.13节“异常处理”所述，以避免这种情况。

从版本1.3.2开始，默认的ErrorHandler现在是一个ConditionalRejectingErrorHandler，它将拒绝(而不是重新排序)消息，并发生不可恢复的错误：

>- o.s.amqp...MessageConversionException
- o.s.messaging...MessageConversionException
- o.s.messaging...MethodArgumentNotValidException
- o.s.messaging...MethodArgumentTypeMismatchException
- java.lang.NoSuchMethodException
- java.lang.ClassCastException

使用MessageConverter转换传入的消息有效负载时，可以抛出第一个。如果在映射到@RabbitListener方法时需要额外的转换，则转换服务可能会抛出第二个。如果在监听器中使用验证(例如@Valid)，并且验证失败，则可能会抛出第三个。如果入站邮件转换为目标方法不正确的类型，则可能会抛出第四个邮件。例如，该参数被声明`为Message<Foo>`，但接收到`Message<Bar>`。

版本1.6.3中添加了第五和第六。

可以使用FatalExceptionStrategy配置此错误处理程序的实例，以便用户可以提供自己的条件消息拒绝规则，例如。来自Spring Retry(称为“消息监听器和异步情况”的部分)的BinaryExceptionClassifier的委托实现。另外，ListenerExecutionFailedException现在有一个failMessage属性可以在决定中使用。如果FatalExceptionStrategy.isFatal()方法返回true，则错误处理程序将抛出一个AmqpRejectAndDontRequeueException异常。当异常确定为致命时，默认的FatalExceptionStrategy会记录一条警告消息。

自1.6.3版本以来，将用户异常添加到致命列表中的方便方法是将ConditionalRejectingErrorHandler.DefaultExceptionStrategy子类化，并覆盖方法isUserCauseFatal(Throwable cause)为致命异常返回true。

#### 3.1.14 Transactions(事务)
##### 介绍
Spring Rabbit框架支持在同步和异步使用情况下进行自动事务管理，具有多种不同的语义，可以声明式选择，正如Spring事务的现有用户所熟悉的那样。这使得许多如果不是最常见的消息传递模式非常容易实现。

有两种方式将所需的事务语义信号发送到框架。在RabbitTemplate和SimpleMessageListenerContainer中都有一个标志channelTransacted，如果为true，则告知框架使用事务通道，并根据结果结束提交或回滚以结束所有操作(发送或接收)，并发出异常指示回滚。另一个信号是提供一个外部事务与Spring的PlatformTransactionManager实现之一作为正在进行的操作的上下文。如果在框架发送或接收消息时已经有一个事务正在进行，并且channelTransacted标志为真，那么消息传递事务的提交或回滚将被推迟到当前事务结束。如果channelTransacted标志为false，则没有事务语义适用于消息传递操作(它是自动检测的)。

channelTransacted标志是一个配置时间设置：当AMQP组件被创建时，它通常在应用程序启动时被声明和处理一次。外部事务原则上更动态，因为系统在运行时响应当前的Thread状态，但实际上当事务按声明方式分层到应用程序时通常也是一个配置设置。

对于使用RabbitTemplate的同步用例，外部事务由调用者提供，无论是声明性还是根据味道(通常的Spring事务模型)强制执行。声明性方法(通常是首选，因为它是非侵入性的)的示例，其中模板已配置为channelTransacted = true：

```java
@Transactional
public void doSomething() {
    String incoming = rabbitTemplate.receiveAndConvert();
    // do some more database processing...
    String outgoing = processInDatabaseAndExtractReply(incoming);
    rabbitTemplate.convertAndSend(outgoing);
}
```

在一个标记为@Transactional的方法内接收、转换和发送一个字符串作为一个消息体，因此如果数据库处理异常失败，传入的消息将返回给代理，而传出的消息将不会被发送。

对于使用SimpleMessageListenerContainer的异步使用情况，如果需要外部事务，则必须在容器设置监听器时请求它。为了表明需要外部事务，用户在配置时向容器提供了PlatformTransactionManager的实现。例如：

```java
@Configuration
public class ExampleExternalTransactionAmqpConfiguration {

    @Bean
    public SimpleMessageListenerContainer messageListenerContainer() {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(rabbitConnectionFactory());
        container.setTransactionManager(transactionManager());
        container.setChannelTransacted(true);
        container.setQueueName("some.queue");
        container.setMessageListener(exampleListener());
        return container;
    }

}
```

在上面的例子中，事务管理器被添加为从另一个bean定义(未显示)注入的依赖关系，并且channelTransacted标志也被设置为true。效果是，如果监听器失败并发生异常，则事务将被回滚，并且该消息也将返回给代理。重要的是，如果事务无法提交(例如数据库约束错误或连接性问题)，则AMQP事务也将被回滚，并且该消息将被返回给代理。这有时被称为最佳努力1阶段提交，并且是可靠消息传递的非常强大的模式。如果在上面的示例中将channelTransacted标志设置为false，这是默认值，则仍将为监听器提供外部事务，但所有消息传递操作都将自动检测，因此其效果是即使提交消息传递操作在业务运行的回滚。

##### Conditional Rollback
在版本1.6.6之前，当回收规则添加到容器的transactionAttribute中时，使用外部事务管理器(例如JDBC)不起作用;异常总是回滚事务。

此外，当在容器的建议链中使用[事务建议](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/transaction.html#transaction-declarative)时，条件回滚并不是非常有用，因为所有监听器异常都被包装在ListenerExecutionFailedException中。

第一个问题已得到纠正，规则现在得到适当应用。此外，现在提供了ListenerFailedRuleBasedTransactionAttribute;它是RuleBasedTransactionAttribute的一个子类，唯一的区别是它知道ListenerExecutionFailedException，并且使用规则的这种异常的原因。此事务属性可以直接在容器中使用，也可以通过事务建议使用。

使用此规则的示例如下：
```java
@Bean
public AbstractMessageListenerContainer container() {
    ...
    container.setTransactionManager(transactionManager);
    RuleBasedTransactionAttribute transactionAttribute =
        new ListenerFailedRuleBasedTransactionAttribute();
    transactionAttribute.setRollbackRules(Collections.singletonList(
        new NoRollbackRuleAttribute(DontRollBackException.class)));
    container.setTransactionAttribute(transactionAttribute);
    ...
}
```

##### A note on Rollback of Received Messages
AMQP事务只适用于发送到代理的消息和acks，所以当回滚Spring事务并且收到一条消息时，Spring AMQP必须做的不仅仅是回滚事务，而且手动拒绝消息(这是一个坏消息，但这不是规范所说的)。对消息拒绝采取的操作与事务无关，并且取决于defaultRequeueRejected属性(默认为true)。有关拒绝失败消息的更多信息，请参阅“消息监听器和异步事件”一节。

有关RabbitMQ事务的更多信息及其限制，请参阅RabbitMQ代理语法。

> 在RabbitMQ 2.7.0之前，这样的消息(以及任何在通道关闭或中断时未被解除的消息)都会到达Rabbit代理队列的后面，因为2.7.0，拒绝的消息到队列的前面，以与JMS相似的方式回滚消息。

> 事务回滚中的消息重新排序在本地事务和提供TransactionManager时不一致。在前一种情况下，适用正常的重新排序逻辑(AmqpRejectAndDontRequeueException或defaultRequeueRejected = false)(参见“消息监听器和异步情况”一节);与一个事务管理器，该消息是无条件地回滚。从版本1.7.1开始，您可以通过将容器的alwaysRequeueWithTxManagerRollback属性设置为false来启用一致的行为;在默认情况下，它将为false。请参见第3.1.15节“消息监听器容器配置”。

##### Using the RabbitTransactionManager
RabbitTransactionManager是在外部事务中执行Rabbit操作并与外部事务同步的替代方法。此事务管理器是PlatformTransactionManager接口的实现，应与单个Rabbit ConnectionFactory一起使用。

> 此策略不能提供XA事务，例如为了在消息传递和数据库访问之间共享事务。

需要应用程序代码通过ConnectionFactoryUtils.getTransactionalResourceHolder(ConnectionFactory，boolean)来检索事务性Rabbit资源，而不是后续通道创建时的标准Connection.createChannel()调用。当使用Spring AMQP的RabbitTemplate时，它将自动检测线程绑定的通道并自动参与其事务。

使用Java配置，您可以使用以下方式设置新的RabbitTransactionManager：
```java
@Bean
public RabbitTransactionManager rabbitTransactionManager() {
    return new RabbitTransactionManager(connectionFactory);
}
```
如果您喜欢使用XML配置，请在XML应用程序上下文文件中声明以下bean：
```xml
<bean id="rabbitTxManager"
      class="org.springframework.amqp.rabbit.transaction.RabbitTransactionManager">
    <property name="connectionFactory" ref="connectionFactory"/>
</bean>
```

#### 3.1.15 Message Listener Container Configuration(Message Listener容器配置)
有很多的选项配置SimpleMessageListenerContainer相关事务和服务，其中一些可以相互作用。

下表显示了使用命名空间配置`<rabbit：listener-container />`时的容器属性名称及其等效属性名称(括号中)。


PS：其他内容还在继续翻译当中