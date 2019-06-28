title: "HttpClient参数配置"
date: 2018-04-10 18:00
category: Java
tags: [RestTemplate, httpClient, Spring, OkHttp, 参数配置, 最佳实践]
---

HttpClient参数配置、HttpClient最佳实践

---

## OkHttp
参数配置项

```java
import okhttp3.*;
import okhttp3.internal.Util;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManager;
import java.io.File;
import java.io.IOException;
import java.security.SecureRandom;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
 
/**
 * OkHttpUtil
 *
 * @author liuxing
 * @summary OkHttpUtil
 * @since 2017-11-10 17:42
 */
public class OkHttpUtil {
 
    private static final Logger LOGGER = LoggerFactory.getLogger(OkHttpUtil.class);
 
    public static OkHttpClient client;
 
    static {
        OkHttpClient.Builder build = new OkHttpClient.Builder()
                // 失败后重试，默认true
                .retryOnConnectionFailure(true)
                // 读取超时 默认 10秒
                .readTimeout(10, TimeUnit.SECONDS)
                // 写入超时 默认 10秒
                .writeTimeout(10, TimeUnit.SECONDS)
                // 连接超时 默认 10秒
                .connectTimeout(5, TimeUnit.SECONDS)
                // 连接池 默认是ConnectionPool的构造设置
                .connectionPool(new ConnectionPool(100, 30, TimeUnit.SECONDS))
                // 心跳时间 默认0，不检测
                .pingInterval(5, TimeUnit.SECONDS);
 
        ExecutorService executor = new ThreadPoolExecutor(50, 200, 30, TimeUnit.SECONDS,
                new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
 
        // 异步调度线程池
        Dispatcher dispatcher = new Dispatcher(executor);
        dispatcher.setMaxRequests(1000);
        dispatcher.setMaxRequestsPerHost(1000);
 
        build.dispatcher(dispatcher);
 
        // ssl
        build.sslSocketFactory(createSSLSocketFactory(), new TrustAllManager());
        build.hostnameVerifier(new TrustAllHostnameVerifier());
 
        client = build.build();
    }
 
    public static OkHttpClient client() {
        return client;
    }
 
    /**
     * 默认信任所有的证书
     * TODO 最好加上证书认证，主流App都有自己的证书
     *
     * @return
     */
    private static SSLSocketFactory createSSLSocketFactory() {
 
        SSLSocketFactory sSLSocketFactory = null;
 
        try {
            SSLContext sc = SSLContext.getInstance("TLS");
            sc.init(null, new TrustManager[]{new TrustAllManager()}, new SecureRandom());
            sSLSocketFactory = sc.getSocketFactory();
        } catch (Exception e) {
        }
 
        return sSLSocketFactory;
    }
}
```

## HttpClient
常用配置项

```java
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.conn.ssl.TrustStrategy;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.InputStreamEntity;
import org.apache.http.impl.client.*;
import org.apache.http.ssl.SSLContexts;
import org.apache.http.util.EntityUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
import java.util.concurrent.TimeUnit;
 
/**
 * HttpClientUtil
 *
 * @author liuxing
 * @summary HttpClientUtil
 * @since 2017-12-01 14:43
 */
public class HttpClientUtil {
 
    private static final Logger LOGGER = LoggerFactory.getLogger(HttpClientUtil.class);
 
    private static CloseableHttpClient httpClient;
 
    static {
        HttpClientBuilder builder = HttpClients.custom();
        builder.setConnectionTimeToLive(30L, TimeUnit.SECONDS);
        builder.setMaxConnTotal(1000);
        builder.setMaxConnPerRoute(1000);
 
        builder.setRetryHandler(new DefaultHttpRequestRetryHandler(2, true));
 
        // 保持长连接配置，需要在头添加Keep-Alive
        // 该策略依赖响应头的Keep-Alive参数：如 Keep-Alive: timeout=5, max=100 单位秒
        // 否则则依赖ConnectionTimeToLive的设置
        builder.setKeepAliveStrategy(DefaultConnectionKeepAliveStrategy.INSTANCE);
 
        try {
            SSLContext sslContext = SSLContexts.custom().loadTrustMaterial(null, (TrustStrategy) (chain, authType) -> true).build();
            builder.setSSLContext(sslContext);
        } catch (Exception e) {
            LOGGER.error(e.getMessage(), e);
        }
 
        try {
            SSLContext sc = SSLContext.getInstance("TLS");
            sc.init(null, new TrustManager[]{new TrustAllManager()}, new SecureRandom());
            builder.setSSLContext(sc);
        } catch (Exception e) {
            LOGGER.error(e.getMessage(), e);
        }
 
        RequestConfig requestConfig = RequestConfig.custom()
                // 读写超时
                .setSocketTimeout(10000)
                // 请求超时
                .setConnectTimeout(10000)
                // 从连接池中获取连接的时间，非请求时间
                .setConnectionRequestTimeout(200)
                // 最大重定向次数
                .setMaxRedirects(10)
                // 自动gzip
                .setContentCompressionEnabled(true)
                .build();
 
        builder.setDefaultRequestConfig(requestConfig);
 
        httpClient = builder.build();
 
        LOGGER.info("requestConfig: {}", requestConfig.toString());
    }
 
    public static CloseableHttpClient client() {
        return httpClient;
    }
 
    /**
     * 信任所有证书
     *
     * @author liuxing
     * @summary 信任所有证书
     * @since 2017-11-30 23:36
     */
    public static class TrustAllManager implements X509TrustManager {
 
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
 
        }
 
        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
 
        }
 
        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    }
}
```

## RestTemplate
RestTemplate本质只是一个模板，具体的请求需要根据初始化配置的RequestFactory

从Spring 4.3.x开始，可以选择OkHttp、HttpClient、Netty

默认是使用Java URLConnection（未池化连接，不推荐使用）。


HttpClient、OkHttp、RestTemplate都只需要配置一个全局单例即可。

常用的超时时间配置5-10秒即可，最大并发连接500-1000，都默认开启的Gzip压缩

## 示例
以HttpClient为例

**maven引入jar包**
```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.4</version>
</dependency>

<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpmime</artifactId>
    <version>4.5.4</version>
</dependency>

```

**普通的get请求**

```java
/**
 * 普通的get请求
 */
@Test
public void testDoGet() {
    HttpGet httpGet = new HttpGet("https://www.baidu.com");
    try {
        try (CloseableHttpResponse response = client.execute(httpGet)) {
            if (Objects.equals(response.getStatusLine().getStatusCode(), HttpStatus.SC_OK)) {
                LOGGER.info("GET 请求成功:\n {}", EntityUtils.toString(response.getEntity()));
            } else {
                LOGGER.error("GET 请求失败:\n {}", EntityUtils.toString(response.getEntity()));
            }
        }
    } catch (Exception e) {
        LOGGER.error(e.getMessage(), e);
    } finally {
        httpGet.releaseConnection();
    }
}
```

**普通的表单请求**

```java
/**
 * 普通的表单请求
 */
@Test
public void testDoPost() {
    HttpPost httpPost = new HttpPost("http://localhost:3000/api");
    HttpEntity entity = EntityBuilder.create()
            .setParameters(
                    new BasicNameValuePair("errFlag", "false"),
                    new BasicNameValuePair("name", "xxx"),
                    new BasicNameValuePair("code", "123456")
            )
            .setContentType(ContentType.create("application/x-www-form-urlencoded", "utf-8"))
            .build();

    httpPost.setEntity(entity);
    try {
        try (CloseableHttpResponse response = client.execute(httpPost)) {
            if (Objects.equals(response.getStatusLine().getStatusCode(), HttpStatus.SC_OK)) {
                LOGGER.info("POST 请求成功:\n {}", EntityUtils.toString(response.getEntity()));
            } else {
                LOGGER.error("POST 请求失败:\n {}", EntityUtils.toString(response.getEntity()));
            }
        }
    } catch (Exception e) {
        LOGGER.error(e.getMessage(), e);
    } finally {
        httpPost.releaseConnection();
    }
}
```

**Json数据post请求**

```java
/**
 * json数据post请求
 * <p>
 * put、delete、patch同post
 * 使用不同的method实现 @see {@link org.apache.http.client.methods.HttpEntityEnclosingRequestBase}
 */
@Test
public void testDoJsonPost() {
    Map<String, Object> params = new HashMap<>();
    params.put("name", "xxx");
    params.put("code", "123456");
    params.put("errFlag", "false");

    HttpPost httpPost = new HttpPost("http://localhost:3000/api");
    HttpEntity entity = EntityBuilder.create()
            .setText(JSON.toJSONString(params))
            .setContentType(ContentType.create("application/json", "utf-8"))
            .build();

    httpPost.setEntity(entity);
    try {
        try (CloseableHttpResponse response = client.execute(httpPost)) {
            if (Objects.equals(response.getStatusLine().getStatusCode(), HttpStatus.SC_OK)) {
                LOGGER.info("POST JSON 请求成功:\n {}", EntityUtils.toString(response.getEntity()));
            } else {
                LOGGER.error("POST JSON 请求失败:\n {}", EntityUtils.toString(response.getEntity()));
            }
        }
    } catch (Exception e) {
        LOGGER.error(e.getMessage(), e);
    } finally {
        httpPost.releaseConnection();
    }
}
```