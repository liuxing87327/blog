title: "使用spring4.x的websocket支持"
date: 2015-04-21 00:48
category: [Java]
tags: [spring,websocket]
---

J2EE7版（JSR-356） 
http://liuxing87327.github.io/2015/02/09/websocket-demo

相关依赖请参考上文，spring需要4.x


##websocket处理器

```java
import org.apache.commons.collections.MapUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.BinaryWebSocketHandler;
import org.springframework.web.socket.handler.TextWebSocketHandler;

/**
 * 功能说明：WebSocket处理器
 * 可以继承 {@link TextWebSocketHandler}/{@link BinaryWebSocketHandler}，
 * 或者简单的实现{@link WebSocketHandler}接口
 * 作者：liuxing(2015-01-25 03:42)
 */
public class TelWebSocketHandler extends TextWebSocketHandler {

    private static final Logger LOGGER = LoggerFactory.getLogger(TelWebSocketHandler.class);

    /**
     * 建立连接
     * @param session
     * @throws Exception
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        String inquiryId = MapUtils.getString(session.getAttributes(), "inquiryId");
        int empNo = MapUtils.getInteger(session.getAttributes(), "empNo");
        TelSocketSessionUtils.add(inquiryId, empNo, session);
    }

    /**
     * 收到客户端消息
     * @param session
     * @param message
     * @throws Exception
     */
    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String inquiryId = MapUtils.getString(session.getAttributes(), "inquiryId");
        int empNo = MapUtils.getInteger(session.getAttributes(), "empNo");
        TelSocketSessionUtils.sendMessage(inquiryId, empNo, "【来自服务器的复读机】：" + message.getPayload().toString());
    }

    /**
     * 出现异常
     * @param session
     * @param exception
     * @throws Exception
     */
    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        String inquiryId = MapUtils.getString(session.getAttributes(), "inquiryId");
        int empNo = MapUtils.getInteger(session.getAttributes(), "empNo");

        LOGGER.error("websocket connection exception: " + TelSocketSessionUtils.getKey(inquiryId, empNo));
        LOGGER.error(exception.getMessage(), exception);

        TelSocketSessionUtils.remove(inquiryId, empNo);
    }

    /**
     * 连接关闭
     * @param session
     * @param closeStatus
     * @throws Exception
     */
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
        String inquiryId = MapUtils.getString(session.getAttributes(), "inquiryId");
        int empNo = MapUtils.getInteger(session.getAttributes(), "empNo");
        TelSocketSessionUtils.remove(inquiryId, empNo);
    }

    /**
     * 是否分段发送消息
     * @return
     */
    @Override
    public boolean supportsPartialMessages() {
        return false;
    }

}
```

##websocket连接的拦截器
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Map;

/**
 * 功能说明：websocket连接的拦截器
 * 有两种方式
 *          一种是实现接口HandshakeInterceptor，实现beforeHandshake和afterHandshake函数
 *          一种是继承HttpSessionHandshakeInterceptor，重载beforeHandshake和afterHandshake函数
 * 我这里是参照spring官方文档中的继承HttpSessionHandshakeInterceptor的方式
 * 作者：liuxing(2015-01-25 03:46)
 */
public class TelWebSocketHandshakeInterceptor extends HttpSessionHandshakeInterceptor {

    private static final Logger LOGGER = LoggerFactory.getLogger(TelWebSocketHandshakeInterceptor.class);

    /**
     * 从请求中获取唯一标记参数，填充到数据传递容器attributes
     * @param serverHttpRequest
     * @param serverHttpResponse
     * @param wsHandler
     * @param attributes
     * @return
     * @throws Exception
     */
    @Override
    public boolean beforeHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        if (getSession(serverHttpRequest) != null) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) serverHttpRequest;
            HttpServletRequest request = servletRequest.getServletRequest();
            attributes.put("inquiryId", request.getParameter("inquiryId"));
            attributes.put("empNo", request.getParameter("empNo"));
        }

        super.beforeHandshake(serverHttpRequest, serverHttpResponse, wsHandler, attributes);

        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception ex) {
        super.afterHandshake(request, response, wsHandler, ex);
    }

    private HttpSession getSession(ServerHttpRequest request) {
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest serverRequest = (ServletServerHttpRequest) request;
            return serverRequest.getServletRequest().getSession(false);
        }
        return null;
    }

}
```

##session工具类
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;

import java.io.IOException;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 功能说明：TelSocketSessionUtils
 * 作者：liuxing(2014-12-26 02:32)
 */
public class TelSocketSessionUtils {

    private static final Logger LOGGER = LoggerFactory.getLogger(TelSocketSessionUtils.class);

    private static Map<String, WebSocketSession> clients = new ConcurrentHashMap<>();

    /**
     * 保存一个连接
     * @param inquiryId
     * @param empNo
     * @param session
     */
    public static void add(String inquiryId, int empNo, WebSocketSession session){
        clients.put(getKey(inquiryId, empNo), session);
    }

    /**
     * 获取一个连接
     * @param inquiryId
     * @param empNo
     * @return
     */
    public static WebSocketSession get(String inquiryId, int empNo){
        return clients.get(getKey(inquiryId, empNo));
    }

    /**
     * 移除一个连接
     * @param inquiryId
     * @param empNo
     */
    public static void remove(String inquiryId, int empNo) throws IOException {
        clients.remove(getKey(inquiryId, empNo));
    }

    /**
     * 组装sessionId
     * @param inquiryId
     * @param empNo
     * @return
     */
    public static String getKey(String inquiryId, int empNo) {
        return inquiryId + "_" + empNo;
    }

    /**
     * 判断是否有效连接
     * 判断是否存在
     * 判断连接是否开启
     * 无效的进行清除
     * @param inquiryId
     * @param empNo
     * @return
     */
    public static boolean hasConnection(String inquiryId, int empNo) {
        String key = getKey(inquiryId, empNo);
        if (clients.containsKey(key)) {
            return true;
        }

        return false;
    }

    /**
     * 获取连接数的数量
     * @return
     */
    public static int getSize() {
        return clients.size();
    }

    /**
     * 发送消息到客户端
     * @param inquiryId
     * @param empNo
     * @param message
     * @throws Exception
     */
    public static void sendMessage(String inquiryId, int empNo, String message) throws Exception {
        if (!hasConnection(inquiryId, empNo)) {
            throw new NullPointerException(getKey(inquiryId, empNo) + " connection does not exist");
        }

        WebSocketSession session = get(inquiryId, empNo);
        try {
            session.sendMessage(new TextMessage(message));
        } catch (IOException e) {
            LOGGER.error("websocket sendMessage exception: " + getKey(inquiryId, empNo));
            LOGGER.error(e.getMessage(), e);
            clients.remove(getKey(inquiryId, empNo));
        }
    }

}
```

##初始化配置
```xml
<!--websocket配置-->
<bean id="telWebSocketHandler" class="包.websocket.handler.TelWebSocketHandler"/>

<websocket:handlers allowed-origins="*">
    <websocket:mapping path="webSocketStatus" handler="telWebSocketHandler"/>
    <websocket:handshake-interceptors>
        <bean class="包.websocket.interceptor.TelWebSocketHandshakeInterceptor"/>
    </websocket:handshake-interceptors>
</websocket:handlers>

<bean class="org.springframework.web.socket.server.standard.ServletServerContainerFactoryBean">
    <property name="maxTextMessageBufferSize" value="8192"/>
    <property name="maxBinaryMessageBufferSize" value="8192"/>
    <property name="maxSessionIdleTimeout" value="900000"/>
    <property name="asyncSendTimeout" value="5000"/>
</bean>
```

spring官方文档已经写得很齐全了，更多场景和说明请参阅下文大笑
http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket
