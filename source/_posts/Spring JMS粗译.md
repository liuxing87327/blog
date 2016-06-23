title: "Spring JMS粗译"
date: 2015-07-05 21:30:00
category: Java
tags: [Spring,JMS]

---

## 24.1介绍

Spring提供了JMS集成框架简化了JMS API的使用，像Spring JDBC API的使用。

JMS的功能大致上分为两块，叫做消息发送和消息监听。JmsTemplate 用于发送消息和同步消息监听。和Java EE的事件驱动Bean风格类似，对于异步接收消息，Spring提供了一些消息监听容器来创建消息驱动的POJO（MDP）。

包 `org.springframework.jms.core` 提供使用JMS的核心功能。 就象为JDBC提供的 JdbcTemplate一样，它提供了JMS模板类来处理资源的创建和释放以简化JMS的使用。Spring模板类的公共设计原则就是通过提供工具方法去执行公共的操作，并将实际的处理任务委派到用户实现的回调接口上，从而完成更复杂的操作。JMS模板也遵循这样的设计原则。这些类提供众多便利的方法来发送消息、同步接收消息、使用户可以接触到JMS session和消息产生者。

包 `org.springframework.jms.support` 提供 `JMSException` 的转换功能。它将受控的 `JMSException` 异常层次转换到一个对应的非受控异常层次。任何受控 `javax.jms.JMSException` 异常的子类都被包装在非受控  `UncategorizedJmsException` 异常里。

包 `org.springframework.jms.support.converter` 提供一个 `MessageConverter` 用来抽象Java对象和JMS消息之间的转换操作。

包 `org.springframework.jms.support.destination` 为管理JMS目的地提供多种策略，例如为存储在JNDI中的目的地提供一个服务定位器。

包 `org.springframework.jms.annotation` 提供了必要的基础设施 支持注解驱动的端点侦听器使用 `@JmsListener`。

包 `org.springframework.jms.config` 提供的解析器实现 jms 命名空间以及java配置支持容器和配置监听器、创建端点侦听器。

包 `org.springframework.jms.connection` 提供一个适合在独立应用中使用的 `ConnectionFactory` 的实现。它还为JMS提供了一个Spring的 `PlatformTransactionManager` 的实现（现在叫做 `JmsTransactionManager`）。这样可以把JMS作为一个事务资源无缝地集成到Spring的事务管理机制中去。

## 24.2使用Spring JMS
### 24.2.1 JmsTemplate
使用 **JmsTemplate** 的代码只需要实现规范中定义的回调接口。**MessageCreator** 回调接口通过 **JmsTemplate** 中调用代码提供的**Session**来创建一条消息。然而，为了允许更复杂的JMS API应用，回调接口**SessionCallback** 为用户提供JMS session，并且回调接口**ProducerCallback** 将**Session**和**MessageProducer**对显露给用户。

JMS API有两种发送方法，一种采用发送模式、优先级和存活时间作为服务质量（QOS）参数，另一种使用无需QOS参数的缺省值方法。由于在 JmsTemplate 中有许多种发送方法，QOS参数通过bean的属性方式进行设置，从而避免在多种发送方法中重复。同样，使用 setReceiveTimeout 属性值来设置同步接收调用的超时值。

某些JMS供应者允许通过ConnectionFactory的配置来设置缺省的QOS值。这样在调用 MessageProducer 的发送方法 send(Destination destination, Message message) 时会使用那些不同的QOS缺省值，而不是JMS规范中定义的值。所以，为了提供对QOS值的一致管理，JmsTemplate必须通过设置布尔值属性 isExplicitQosEnabled 为true，使它能够使用自己的QOS值。

>JmsTemplate 类的实例 一经配置便是线程安全 的。 这很重要，因为这意味着你可以配置一个 JmsTemplate 的单例，然后把这个 共享的 引用安全的实例注入多个协作的对象中。 要清楚一点，JmsTemplate 是有状态的，因为它维护了 ConnectionFactory 的引用，但这个状态不属于会话状态。

Spring Framework 4.1提供了一个**JmsMessagingTemplate**，这个是对JmsTemplate的包装，主要用来发送最基本的消息内容，即`org.springframework.messaging.Message`

### 24.2.2连接
JmsTemplate 需要一个对 ConnectionFactory 的引用。ConnectionFactory 是JMS规范的一部分，并且是使用JMS的入口。客户端应用通常用它作工厂配合JMS提供者去创建连接，并封装许多和供应商相关的配置参数，例如SSL的配置选项。

当在EJB里使用JMS时，供应商会提供JMS接口的实现，这样们可以参与声明式事务管理并提供连接池和会话池。 为了使用这个JMS实现，Java EE容器通常要求你在EJB或servlet部署描述符中声明一个JMS连接工厂做为 resource-ref。 为确保可以在EJB内使用 JmsTemplate 的这些特性，客户应用应当确保它引用了被管理的ConnectionFactory实现。

**缓存消息传递资源**
标准的API执行流程如下：
ConnectionFactory->Connection->Session->MessageProducer->send

从创建到发送有三个中间对象的创建和销毁，为了提高性能，Spring提供了**ConnectionFactory**

**SingleConnectionFactory**
Spring提供了一个 ConnectionFactory 接口的实现，SingleConnectionFactory，它将在所有的 createConnection 调用中返回一个相同的 Connection，并忽略所有对 close的调用。这在测试和独立环境中相当有用，因为多个 JmsTemplate 调用可以使用同一个连接以跨越多个事务。SingleConnectionFactory 通常引用一个来自JNDI的标准 ConnectionFactory。

**CachingConnectionFactory**
CachingConnectionFactory类扩展自SingleConnectionFactory，主要用于提供缓存JMS资源功能。具体包括MessageProducer、MessageConsumer和Session的缓存功能。
默认情况下，CachingConnectionFactory只缓存一个session，在它的JavaDoc中，它声明对于低并发情况下这是足够的。可以使用SessionCacheSize进行配置。


### 24.2.3目的地管理
和连接工厂一样，目的地是可以在JNDI中存储和获取的JMS管理的对象。配置一个Spring应用上下文时，可以使用JNDI工厂类 JndiObjectFactoryBean 把对你对象的引用依赖注入到JMS目的地中。然而，如果在应用中有大量的目的地，或者JMS供应商提供了特有的高级目的地管理特性，这个策略常常显得很麻烦。创建动态目的地或支持目的地的命名空间层次就是这种高级目的地管理的例子。JmsTemplate 将目的地名称到JMS目的地对象的解析委派给 DestinationResolver 接口的一个实现。JndiDestinationResolver 是 JmsTemplate 使用的默认实现，并且提供动态目的地解析。同时 JndiDestinationResolver 作为JNDI中的目的地服务定位器，还可选择回退去使用 DynamicDestinationResolver 中的行为。

经常见到一个JMS应用中使用的目的地只有在运行时才知道，因此，当部署一个应用时，它不能用可管理的方式创建。这是经常发生的，因为在互相作用的系统组件间有些共享应用逻辑会在运行的时候按照共同的命名规范创建消息目的地。虽然动态创建目的地不是JMS规范的一部分，但是大多数供应商已经提供了这个功能。用户为动态创建的目的地定义和临时目的地不同的名字，并且通常不被注册到JNDI中。不同供应商创建动态消息目的地所使用的API差异很大，因为和目的地相关的属性是供应商特有的。然而，有时由供应商会作出一个简单的实现选择-忽略JMS规范中的警告，使用 TopicSession 的方法 createTopic(String topicName) 或者 QueueSession 的方法 createQueue(String queueName) 来创建一个带默认值属性的新目的地。依赖于供应商的实现，DynamicDestinationResolver 也可能创建一个物理上的目的地，而不再仅仅是一个解析。

布尔属性 pubSubDomain 用来配置 JmsTemplate 使用什么样的JMS域。这个属性的默认值是false，使用点到点的域，也就是队列。在1.0.2的实现中，这个属性值用来决定 JmsTemplate 将消息发送到一个 Queue 还是一个 Topic。这个标志在1.1的实现中对发送操作没有影响。然而，在这两个JMS版本中，这个属性决定了通过接口 DestinationResolver 的实现来决定如何解析动态消息目的地。

你还可以通过属性 defaultDestination 配置一个带有默认目的地的 JmsTemplate。不指明目的地的发送和接受操作将使用该默认目的地。

### 24.2.4消息监听容器
在EJB世界里，JMS消息最常用的功能之一是用于实现消息驱动Bean（MDB）。Spring提供了一个方法来创建消息驱动的POJO（MDP），并且不会把用户绑定在某个EJB容器上。

通常用消息监听器容器从JMS消息队列接收消息并驱动被注射进来的MDP。消息监听器容器负责消息接收的多线程处理并分发到各MDP中。一个消息侦听容器是MDP和消息提供者之间的一个中介，用来处理消息接收的注册，事务管理的参与，资源获取和释放，异常转换等等。这使得应用开发人员可以专注于开发和接收消息（可能的响应）相关的（复杂）业务逻辑，把和JMS基础框架有关的样板化的部分委托给框架处理。

**SimpleMessageListenerContainer**
这个消息侦听容器是最简单的。它在启动时创建固定数量的JMS session并在容器的整个生命周期中使用它们。这个类不能动态的适应运行时的要求或参与消息接收的事务处理。然而它对JMS提供者的要求也最低。它只需要简单的JMS API。

**DefaultMessageListenerContainer**
这个消息侦听器使用的最多。和 SimpleMessageListenerContainer 相反，这个子类可以动态适应运行时侯的要求，也可以参与事务管理。每个收到的消息都注册到一个XA事务中（如果使用 JtaTransactionManager 配置过），这样就可以利用XA事务语义的优势了。这个类在对JMS提供者的低要求和提供包括事务参于等的强大功能上取得了很好的平衡。

### 24.2.5事务管理
Spring提供了 JmsTransactionManager 为单个JMS ConnectionFactory 管理事务。这将允许JMS应用利用Spring的事务管理功能。JmsTransactionManager 绑定 ConnectionFactory 的一个Connection/Session对到线程上，来提供本地资源事务。JmsTemplate 自动检测到这些事务性资源从而对它们进行操作。

在Java EE环境中，SingleConnectionFactory将把Connection和Session放到缓冲池中，因此这些资源在事务中得到了有效的复用。在独立环境中使用Spring的 SingleConnectionFactory 会存在共享的JMS Connection，但每个事务有自己独立的 Session。另外可以考虑使用供应商特定的池适配器,，如ActiveMQ的 PooledConnectionFactory 类。

JmsTemplate 也可以和 JtaTransactionManager 以及具有XA能力的JMS ConnectionFactory一起使用来提供分布式事务。记住这需要使用JTA事务管理器或合适的可配置的XA ConnectionFactory！（参考你所使用的J2EE服务器/JMS供应商的文档。）

当使用JMS API从一个 Connection 中创建 Session 时，在受管理的和非受管理的事务环境下重用代码会可能会让人迷惑。这是因为JMS API只有一个工厂方法来创建 Session ，并且它需要用于事务和模式确认的值。在受管理的环境下，由事务结构环境负责设置这些值，这样在供应商包装的JMS连接中可以忽略这些值。当在一个非管理性的环境中使用 JmsTemplate 时，你可以通过使用属性 SessionTransacted 和 SessionAcknowledgeMode 来指定这些值。当 JmsTemplate 配合 PlatformTransactionManager 使用时，模板将一直被赋予一个事务性JMS的 Session。

## 24.3发送消息
JmsTemplate 包含许多方便的方法来发送消息。有些发送方法可以使用 javax.jms.Destination 对象指定目的地，也可以使用字符串在JNDI中查找目的地。没有目的地参数的发送方法使用默认的目的地。

```java
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Queue;
import javax.jms.Session;

import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.core.JmsTemplate;

public class JmsQueueSender {

    private JmsTemplate jmsTemplate;
    private Queue queue;

    public void setConnectionFactory(ConnectionFactory cf) {
        this.jmsTemplate = new JmsTemplate(cf);
    }

    public void setQueue(Queue queue) {
        this.queue = queue;
    }

    public void simpleSend() {
        this.jmsTemplate.send(this.queue, new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage("hello queue world");
            }
        });
    }
}
```
这个例子使用 MessageCreator 回调接口从提供的 Session 对象中创建一个文本消息，并且通过一个 ConnectionFactory 的引用来创建 JmsTemplate。提供了一个无参数的构造器和 connectionFactory 可用于创建实例（使用一个BeanFactory或者普通Java代码）。或者考虑继承Spring的基类 JmsGatewaySupport，它对JMS配置具有内置的Bean属性。

方法 send(String destinationName, MessageCreator creator) 让你利用目的地的字符串名字发送消息。如果这个名字在JNDI中注册，你应当将模板中的 destinationResolver 属性设置为 JndiDestinationResolver 的一个实例。
如果你创建 JmsTemplate 并指定一个默认的目的地，send(MessageCreator c) 发送消息到这个目的地。

### 24.3.1使用消息转换器
为便于发送领域模型对象，JmsTemplate 有多种以一个Java对象为参数并做为消息数据内容的发送方法。JmsTemplate 里可重载的方法 convertAndSend 和 receiveAndConvert 将转换的过程委托给接口 MessageConverter 的一个实例。这个接口定义了一个简单的合约用来在Java对象和JMS消息间进行转换。缺省的实现 SimpleMessageConverter 支持 String 和 TextMessage，byte[] 和 BytesMesssage，以及 java.util.Map 和 MapMessage 之间的转换。使用转换器，可以使你和你的应用关注于通过JMS接收和发送的业务对象而不用操心它是具体如何表达成JMS消息的。

目前的沙箱模型包括一个 MapMessageConverter，它使用反射转换JavaBean和 MapMessage。其他流行可选的实现方式包括使用已存在的XML编组的包，例如JAXB、Castor、XMLBeans或XStream的转换器来创建一个表示对象的 TextMessage。

为方便那些不能以通用方式封装在转换类里的消息属性，消息头和消息体的设置，通过 MessagePostProcessor 接口你可以在消息被转换后并且在发送前访问该消息。下例展示了如何在 java.util.Map 已经转换成一个消息后更改消息头和属性。

```java
public void sendWithConversion() {
    Map map = new HashMap();
    map.put("Name", "Mark");
    map.put("Age", new Integer(47));
    jmsTemplate.convertAndSend("testQueue", map, new MessagePostProcessor() {
        public Message postProcessMessage(Message message) throws JMSException {
            message.setIntProperty("AccountID", 1234);
            message.setJMSCorrelationID("123-00001");
            return message;
        }
    });
}
```
将产生以下消息格式
```json
MapMessage={
	Header={
		... standard headers ...
		CorrelationID={123-00001}
	}
	Properties={
		AccountID={Integer:1234}
	}
	Fields={
		Name={String:Mark}
		Age={Integer:47}
	}
}
```

### 24.3.2 SessionCallback和ProducerCallback
虽然send操作适用于许多常见的使用场景，但是有时你需要在一个JMS Session 或者 MessageProducer 上执行多个操作。接口 SessionCallback 和 ProducerCallback 分别提供了JMS Session 和 Session / MessageProducer 对。JmsTemplate 上的 execute() 方法执行这些回调方法。

## 24.4接收消息
### 24.4.1同步接收
虽然JMS一般都和异步处理相关，但它也可以同步的方式使用消息。可重载的 receive(..) 方法提供了这种功能。在同步接收中，接收线程被阻塞直至获得一个消息，有可能出现线程被无限阻塞的危险情况。属性 receiveTimeout 指定了接收器可等待消息的延时时间。

### 24.4.2异步接收消息驱动pojo
>Spring还提供一个 @JmsListener 的注解，以非嵌入式的方式异步接受消息。

类似于EJB世界里流行的消息驱动Bean（MDB），消息驱动POJO（MDP）作为JMS消息的接收器。MDP的一个约束（但也请看下面的有关 javax.jms.MessageListener 类的讨论）是它必须实现 javax.jms.MessageListener 接口。另外当你的POJO将以多线程的方式接收消息时必须确保你的代码是线程-安全的。

以下是MDP的一个简单实现:

```java
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class ExampleListener implements MessageListener {

    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            try {
                System.out.println(((TextMessage) message).getText());
            }
            catch (JMSException ex) {
                throw new RuntimeException(ex);
            }
        }
        else {
            throw new IllegalArgumentException("Message must be of type TextMessage");
        }
    }

}
```

一旦你实现了 MessageListener 后就可以创建一个消息侦听容器。
请看下面例子是如何定义和配置一个随Sping发行的消息侦听容器的（这个例子用 DefaultMessageListenerContainer）

```xml
<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="jmsexample.ExampleListener" />

<!-- and this is the message listener container -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener" />
</bean>
```

关于各个消息侦听容器实现的功能请参阅相关的Spring Javadoc文档。

### 24.4.3 SessionAwareMessageListener接口
SessionAwareMessageListener 接口是一个Spring专门用来提供类似于JMS MessageListener 的接口，也提供了从接收 Message 来访问JMS Session 的消息处理方法。

```java
package org.springframework.jms.listener;

public interface SessionAwareMessageListener {

    void onMessage(Message message, Session session) throws JMSException;

}
```

如果你希望你的MDP可以响应所有接收到的消息（使用 onMessage(Message, Session) 方法提供的 Session）那么你可以选择让你的MDP实现这个接口（优先于标准的JMS MessageListener 接口)。所有随Spring发行的支持MDP的消息侦听容器都支持 MessageListener 或 SessionAwareMessageListener 接口的实现。要注意的是实现了 SessionAwareMessageListener 接口的类通过接口和Spring有了耦合。是否选择使用它完全取决于开发者或架构师。

请注意 SessionAwareMessageListener 接口的 'onMessage(..)' 方法会抛出 JMSException异常。和标准JMS MessageListener 接口相反，当使用 SessionAwareMessageListener 接口时，客户端代码负责处理任何抛出的异常。

### 24.4.4 MessageListenerAdapter
MessageListenerAdapter 类是Spring的异步支持消息类中的不变类（final class）：简而言之，它允许你几乎将 任意 一个类做为MDP显露出来（当然有某些限制）。

考虑如下接口定义。注意虽然这个接口既不是从 MessageListener 也不是从 SessionAwareMessageListener 继承来得，但通过 MessageListenerAdapter 类依然可以当作一个MDP来使用。同时也请注意各种消息处理方法是如何根据他们可以接收并处理消息的内容来进行强类型匹配的。

```java
public interface MessageDelegate {

    void handleMessage(String message);

    void handleMessage(Map message);

    void handleMessage(byte[] message);

    void handleMessage(Serializable message);

}
```

```java
public class DefaultMessageDelegate implements MessageDelegate {
    // implementation elided for clarity...
}
```

特别请注意，上面的 MessageDelegate 接口（上文中 DefaultMessageDelegate 类）的实现完全 不 依赖于JMS。它是一个真正的POJO，我们可以通过如下配置把它设置成MDP。

```xml
<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
    <constructor-arg>
        <bean class="jmsexample.DefaultMessageDelegate"/>
    </constructor-arg>
</bean>

<!-- and this is the message listener container... -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener" />
</bean>
```

下面是另外一个只能处理接收JMS TextMessage 消息的MDP示例。注意消息处理方法是如何实际调用 'receive' （在 MessageListenerAdapter 中默认的消息处理方法的名字是 'handleMessage'）的，但是它是可配置的（你下面就将看到）。注意 'receive(..)' 方法是如何使用强制类型来只接收和处理JMS TextMessage消息的。

```java
public interface TextMessageDelegate {

    void receive(TextMessage message);

}
```

```java
public class DefaultTextMessageDelegate implements TextMessageDelegate {
    // implementation elided for clarity...
}
```

辅助的 MessageListenerAdapter 类配置文件类似如下：

```xml
<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
    <constructor-arg>
        <bean class="jmsexample.DefaultTextMessageDelegate"/>
    </constructor-arg>
    <property name="defaultListenerMethod" value="receive"/>
    <!-- we don't want automatic message context extraction -->
    <property name="messageConverter">
        <null/>
    </property>
</bean>
```

请注意，如果上面的 'messageListener' 收到一个不是 TextMessage 类型的JMS Message，将会产生一个 IllegalStateException 异常（随之产生的其他异常只被捕获而不处理）。
MessageListenerAdapter 还有一个功能就是如果处理方法返回一个非空值，它将自动返回一个响应 消息。
请看下面的接口及其实现：

```java
public interface ResponsiveTextMessageDelegate {

    // notice the return type...
    String receive(TextMessage message);

}
```

```java
public class DefaultResponsiveTextMessageDelegate implements ResponsiveTextMessageDelegate {
    // implementation elided for clarity...
}
```

如果上面的 DefaultResponsiveTextMessageDelegate 和 MessageListenerAdapter 联合使用，那么任意从执行 'receive(..)' 方法返回的非空值都将（缺省情况下）转换成一个 TextMessage。这个返回的 TextMessage 将被发送到原来的 Message 中JMS Reply-To属性定义的 目的地（如果存在），或者是 MessageListenerAdapter 设置（如果配置了）的缺省 目的地；如果没有定义 目的地，那么将产生一个 InvalidDestinationException 异常（此异常将不会只被捕获而不处理，它 将沿着调用堆栈上传）。


### 24.4.5事务中的消息处理
在消息监听器的调用中使用事务只需要重新配置监听器容器
通过监听器容器定义中的 sessionTransacted 标记可以轻松的激活本地资源事务。每次消息监听器的调用都在激活的JMS事务中执行，执行失败时，消息接收将发生回滚。这个本地事务还将包含响应信息的发送（通过 SessionAwareMessageListener），但其它资源的操作（例如访问数据库）是独立的。经常会发生类似于数据库处理已提交但消息处理提交失败的情况，因此需要在监听器的实现中进行重复消息的检测。

```xml
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="sessionTransacted" value="true"/>
</bean>
```

当参与外部管理的事务时，你需要使用支持外来事务的监听器容器：通常是 DefaultMessageListenerContainer 来配置事务管理器。

参与XA事务时，消息监听器容器需要配置 JtaTransactionManager（默认会委托给J2EE服务器事务子系统）。注意以下JMS ConnectionFactory需要具有XA能力并注册JTA事务协调器！（参考你所使用的J2EE服务器中JNDI资源的配置。）这样，消息接收就像数据库访问一样作为同一个事务的一部分（具有统一提交的语义，仅仅增加了XA事务日志的额外开销）。

```xml
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

然后你只需要把它添加到早先配置好的容器中。这个容器将处理剩下的事情。

```xml
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destination" ref="destination"/>
    <property name="messageListener" ref="messageListener"/>
    <property name="transactionManager" ref="transactionManager"/>
</bean>
```

## 24.5JCA消息端点的支持
从Spring2.5版本开始，Spring也提供了基于JCA MessageListener 容器的支持。 JmsMessageEndpointManager 将根据供应者 ResourceAdapter 的类名自动地决定 ActivationSpec 类名。因此，通常它只提供如下例所示的Spring的通用 JmsActivationSpecConfig 。

```xml
<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
    <property name="resourceAdapter" ref="resourceAdapter"/>
    <property name="activationSpecConfig">
        <bean class="org.springframework.jms.listener.endpoint.JmsActivationSpecConfig">
            <property name="destinationName" value="myQueue"/>
        </bean>
    </property>
    <property name="messageListener" ref="myMessageListener"/>
</bean>
```

另外,你可以设置一个 JmsMessageEndpointManager 指定 ActivationSpec 对象。  ActivationSpec 对象可以通过JNDI查找来完成 (使用 `<jee:jndi-lookup>` )。

```xml
<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
    <property name="resourceAdapter" ref="resourceAdapter"/>
    <property name="activationSpec">
        <bean class="org.apache.activemq.ra.ActiveMQActivationSpec">
            <property name="destination" value="myQueue"/>
            <property name="destinationType" value="javax.jms.Queue"/>
        </bean>
    </property>
    <property name="messageListener" ref="myMessageListener"/>
</bean>
```

使用Spring的 ResourceAdapterFactoryBean， 目标 ResourceAdapter 可以像下例描述的那样本地配置。 在一些环境里（如WebLogic）也可以通过JNDI查找来完成。

```xml
<bean id="resourceAdapter" class="org.springframework.jca.support.ResourceAdapterFactoryBean">
    <property name="resourceAdapter">
        <bean class="org.apache.activemq.ra.ActiveMQResourceAdapter">
            <property name="serverUrl" value="tcp://localhost:61616"/>
        </bean>
    </property>
    <property name="workManager">
        <bean class="org.springframework.jca.work.SimpleTaskWorkManager"/>
    </property>
</bean>
```

请参考 JmsMessageEndpointManager、JmsActivationSpecConfig 和 ResourceAdapterFactoryBean 部分的JavaDoc，以获得更详细的信息。

Spring也提供了并不与JMS绑定的通用JCA消息端点管理器： org.springframework.jca.endpoint.GenericMessageEndpointManager。 它允许使用任何类型的消息监听器（例如CCI MessageListener）和任何提供者特定的ActivationSpec对象。从所涉及的JCA提供者的文档可以找到这个连接器的实际能力，从 GenericMessageEndpointManager 的JavaDoc中可以找到Spring特有的配置细节。

>基于JCA的消息端点管理器与EJB 2.1的Message-Driven Beans很相似，它使用了相同的资源提供者约定。像EJB 2.1 MDB一样，任何被JCA提供者支持的消息监听器接口都可以在Spring Context中使用。尽管如此，Spring仍为JMS提供了显式的“方便的”支持，很显然是因为JMS是JCA端点管理约定中最通用的端点API。

## 24.6注解驱动的监听器
异步接受消息最简单的方法是使用`@JmsListener`
```java
@Component
public class MyService {

    @JmsListener(destination = "myDestination")
    public void processOrder(String data) { ... }
}
```

示例中，收到destination为“myDestination”的消息后，processOrder方法将被执行。

带注解的监听方法，底层是使用JmsListenerContainerFactory容器

### 24.6.1启用端点侦听器的注解
Bean配置的方式：

为了使用@JmsListener注解，需要添加 @EnableJms 到 @Configuration 类。

```java
@Configuration
@EnableJms
public class AppConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory =
                new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        factory.setDestinationResolver(destinationResolver());
        factory.setConcurrency("3-10");
        return factory;
    }
}
```

使用DefaultJmsListenerContainerFactory作为消息监听容器，setConcurrency用于配置消费线程，这里是最小3个，最大10个。

XML的方式：
使用`<jms:annotation-driven>`

```xml
<jms:annotation-driven/>

<bean id="jmsListenerContainerFactory"
        class="org.springframework.jms.config.DefaultJmsListenerContainerFactory">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="destinationResolver" ref="destinationResolver"/>
    <property name="concurrency" value="3-10"/>
</bean>
```

### 24.6.2编程注册端点
JmsListenerEndpoint提供一个JMS端点模型和负责模型配置容器。允许我们以编程方式配置除了jmsListener外的端点。

```java
@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

    @Override
    public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
        SimpleJmsListenerEndpoint endpoint = new SimpleJmsListenerEndpoint();
        endpoint.setId("myJmsEndpoint");
        endpoint.setDestination("anotherQueue");
        endpoint.setMessageListener(message -> {
            // processing
        });
        registrar.registerEndpoint(endpoint);
    }
}
```

在上面的例子中，我们使用SimpleJmsListenerEndpoint提供要调用的实际消息监听者，但你也可以建立自己的端点变量描述自定义调用机制。

值得注意的是，你也可以完全跳过使用@JmsListener，只通过JmsListenerConfigurer以编程方式注册您的端点。

## 24.7JMS命名空间支持
Spring JMS引入了XML命名空间以简化JMS的配置。使用JMS命名空间元素时，需要引用如下的JMS Schema：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jms="http://www.springframework.org/schema/jms"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms.xsd">

    <!-- bean definitions here -->

</beans>
```

这个命名空间由两级元素组成：`<listener-container/>` 和 `<jca-listener-container/>`，它们都可以包含一个或这个多个 <listener/> 子元素。下面是一个基本配置的示例，包含两个监听器。

```xml
<jms:listener-container>

    <jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>

    <jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
```

上面的例子等同于在[24.4.4MessageListenerAdapter](#2444-messagelisteneradapter)的示例中，定义两个不同的监听器容器和两个不同的 MessageListenerAdapter 。除了上面的属性外，listener 元素还具有几个可选的属性。下面的表格列出了所有的属性：

**表24.1 JMS listener 元素的属性** 

|    属性     |    描述    |
| :-------- | :--------| 
| id    |   监听器容器的Bean名称。如果没有指定，将自动生成一个Bean名称。 | 
| destination （必须）    |   监听器目的地的名称，由 DestinationResolver 的策略决定。 | 
| ref （必须）    |   处理对象的Bean名称 | 
| method    |   处理器中被调用的方法名。如果 ref 指向 MessageListener 或者 Spring SessionAwareMessageListener，则这个属性可以被忽略。 | 
| response-destination    |   默认的响应目的地是发送响应消息抵达的目的地。 这用于请求消息没有包含"JMSReplyTo"域的情况。响应目的地类型被监听器容器的"destination-type"属性决定。记住：这仅仅适用于有返回值的监听器方法，因为每个结果对象都会被转化成响应消息。 | 
| subscription    |   持久订阅的名称，如果需要的话。 | 
| selector    |   监听器的一个可选的消息选择器。 | 
| concurrency    |   并发的数量，格式：5（最大），3-5（最小和最大） | 


`<listener-container/>`元素也有几个可选的属性。 这些属性允许像 基本的 JMS设置和资源引用一样来定义不同的策略 （例如 taskExecutor 和 destinationResolver）。 使用这些属性，可以定义很广泛的定制监听器容器，同时仍享有命名空间的便利。

这样的设置可以自动公开一个 JmsListenerContainerFactory
```xml
<jms:listener-container connection-factory="myConnectionFactory"
        task-executor="myTaskExecutor"
        destination-resolver="myDestinationResolver"
        transaction-manager="myTransactionManager"
        concurrency="10">

    <jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>

    <jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
```

下面的表格描述了所有可用的属性。参考 AbstractMessageListenerContainer 类和具体子类的Javadoc来了解每个属性的细节。这部分的Javadoc也提高那个了事务选择和消息传输场景的讨论。

**表24.2 JMS < listener-container >元素的属性** 

|    属性     |    描述    |
| :-------- | :--------| 
|	container-type		|	 监听器容器的类型。可用的选项是： default、simple、default102 或者 simple102 （默认值是 'default'）。 	|	
|	container-class	|	定制监听器容器实现类的完全限定类名。默认是Spring标准使用DefaultMessageListenerContainer或SimpleMessageListenerContainer		|
|	factory-id		|	公开此元素被定义为一个JmsListenerContainerFactory与指定id，以便它们可以与其他的端点被重新使用的设置。		|
|	connection-factory		|		JMS ConnectionFactory Bean的引用（默认的Bean名称是 'connectionFactory'）。	|
|	task-executor		|		JMS监听器调用者Spring TaskExecutor 的引用。	|
|	destination-resolver		|	DestinationResolver 策略的引用，用以解析JMS Destinations。		|
|	message-converter		|	MessageConverter 策略的引用，用以转换JMS Messages 成监听器方法的参数。默认值是 SimpleMessageConverter。		|
|	error-handler		|	异常处理的策略		|
|	destination-type		|		监听器的JMS目的地类型。可用的选项包含： queue、topic 或者 durableTopic （默认值是 'queue'）。	|
|	client-id		|	这个监听器容器在JMS客户端的id。		|
|	cache		|	The cache level for JMS resources:`none`,`connection`,`session`,`consumer` or`auto`. By default (`auto`), the cache level will effectively be "consumer", unless an external transaction manager has been specified - in which case the effective default will be`none` (assuming Java EE-style transaction management where the given ConnectionFactory is an XA-aware pool).		|
|	acknowledge		|	本地JMS应答模式。可用的选项包含： auto、client、dups-ok 或者 transacted （默认值是 'auto'）。 'transacted' 的值可激活本地事务性 Session。 也可以通过指定下面介绍的 transaction-manager 属性。		|
|	transaction-manager		|	Spring PlatformTransactionManager 的引用。		|
|	concurrency		|	每个监听器可激活的Session最大并发数。		|
|	prefetch		|	 加载进每个Session的最大消息数。记住增加这个值会造成并发空闲。		|
|	receive-timeout		|		接受消息的超时时间，单位是毫秒，默认是1000，-1表示没超时	|
|	back-off		|	发生冲突时的强制性重传延迟，如果 BackOffExecution 实现返回 `BackOffExecution #STOP` , 侦听器容器不会进一步尝试恢复，设置的recovery-interval 值将被忽略。默认是一个 FixedBackOff 与 5000毫秒的时间间隔	|
|	recovery-interval		|	指定时间间隔恢复的尝试,以毫秒为单位。 方便 方法创建一个 FixedBackOff 指定的时间间隔。 更多的复苏 选项,可以考虑指定一个`back-off`的实例。 默认值是5000毫秒。	|
|	phase		|	这个容器的生命周期阶段应该启动和停止。越低值在这个容器将开始和后来将停止。默认值是Integer.MAX_VALUE 这意味着容器尽可能晚地将开始和停止快越好。	|


使用“jms” Schema支持来配置基于JCA的监听器容器很相似

```xml
<jms:jca-listener-container resource-adapter="myResourceAdapter"
        destination-resolver="myDestinationResolver"
        transaction-manager="myTransactionManager"
        concurrency="10">

    <jms:listener destination="queue.orders" ref="myMessageListener"/>

</jms:jca-listener-container>
```

**表24.3 JMS < jca-listener-container / >元素的属性**

|    属性     |    描述    |
| :-------- | :--------| 
|	factory-id	|	公开此元素被定义为一个JmsListenerContainerFactory与指定id，以便它们可以与其他的端点被重新使用的设置。	|	
|	resource-adapter	|	JCA ResourceAdapter Bean 的一个引用（默认的Bean名称是'resourceAdapter'）		|	
|	activation-spec-factory	|	JmsActivationSpecFactory 的一个引用。 默认自动检测JMS提供者和它的 ActivationSpec 类 （参考 DefaultJmsActivationSpecFactory）		|	
|	destination-resolver	|	DestinationResolver 策略的引用，用以解析JMS Destinations。		|	
|	message-converter	|	MessageConverter 策略的引用，用以转换JMS Messages 成监听器方法参数。 默认值是 SimpleMessageConverter		|	
|	destination-type	|	监听器的JMS目的地类型。可用的选项包含 queue、topic 或者 durableTopic 默认是 'queue'）。		|	
|	client-id	|	这个监听器容器在JMS客户端的id。		|	
|	acknowledge	|	本地JMS应答模式。可用的选项包含：auto、client、dups-ok 或者 transacted （默认值是 'auto'）。 'transacted' 的值可激活本地事务性 Session。 也可以通过指定下面介绍的 transaction-manager 属性		|	
|	transaction-manager	|	Spring JtaTransactionManager 或者 javax.transaction.TransactionManager 的引用，用以为传进的消息应用XA事务。 如果没有指定，将使用本地应答模型（参见“acknowledge”属性）。		|	
|	concurrency	|	每个监听器可激活的Session最大并发数。	|	
|	prefetch	|	加载进每个Session的最大消息数。记住增加这个值会造成并发空闲。		|	
