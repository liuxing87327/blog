title: "spring4不兼容ibatis的解决方法 "
date: 2015-04-21 00:46
category: Java
tags: [Spring, 框架, ibatis]

---

因为spring升级到4.x后，orm包里面移除了对ibatis的支持，
如果orm使用4.x版本的话项目会启动失败。
查看orm-3.x版本的SqlMapClientFactoryBean类，里面说的很清楚，只支持到3.x。
如果可以升级到mybatis的话尽量升级，否则可以使用如下方法。
orm包单独使用3.x的版本，项目中正式在用，还没出现问题（或许还没爆出来哭）

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>3.2.13.RELEASE</version>
</dependency>
```