title: "引入公共头部时，控制引入js、css、title"
date: 2012-08-16 00:10
category: [java]
tags: []
---

引入公共头部时，控制引入js、css、title，避免重复造轮子，能少敲点代码就少敲点

1.头部或底部文件中引入jstl的标签库

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>  
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>  
```

2.这一部分放入公共的头或底部文件中，放置在待引入css和js的位置，两种方式具体应用中有细微差别

```xml
<!-- 1.可以分开引入 -->
<c:if test="${ not empty param.css}">
	<c:forEach var="css" items="${fn:split(param.css, ',')}">
        <c:if test="${not empty css}">
			<link type="text/css" rel="stylesheet" href="/static/css/${css}">
        </c:if>
	</c:forEach>
</c:if>

<c:if test="${ not empty param.js || not empty param.ajs}">
	<c:forEach var="js" items="${fn:split(param.js, ',')}">
        <c:if test="${not empty js}">
		    <script type="text/javascript" src="/static/js/${js }"></script>
        </c:if>
	</c:forEach>

    <c:forEach var="js" items="${fn:split(param.ajs, ',')}">
        <c:if test="${not empty js}">
			<script type="text/javascript" src="${js }"></script>
        </c:if>
	</c:forEach>
</c:if>

<!-- 2.或者可以一起引入 -->
<c:if test="${not empty param.css || not empty param.js || not empty param.ajs}">
	<c:forEach var="css" items="${fn:split(param.css, ',')}">
        <c:if test="${not empty css}">
			<link type="text/css" rel="stylesheet" href="/static/css/${css}">
        </c:if>
	</c:forEach>

	<c:forEach var="js" items="${fn:split(param.js, ',')}">
        <c:if test="${not empty js}">
		    <script type="text/javascript" src="/static/js/${js }"></script>
        </c:if>
	</c:forEach>

    <c:forEach var="js" items="${fn:split(param.ajs, ',')}">
        <c:if test="${not empty js}">
			<script type="text/javascript" src="${js }"></script>
        </c:if>
	</c:forEach>
</c:if>
```

3.这一部分放置在引入公共头部或底部的页面中

```xml
<!-- ajs表示引入外部的js，在实现中是直接src=ajs；js表示引入系统内部的js，在实现中是直接src="公共的路径"+ajs -->
<!-- 这里还可以声明头部的title等其他的参数,参数是自定义的 在头部或底部文件里直接使用”${param.参数 }“就可以拿到 -->
<jsp:include page="../common/header.jsp">
    <jsp:param name="css" value="test.css"/>
    <jsp:param name="js" value="test.js,test2.js"/>
    <jsp:param name="ajs" value="http://test.js, http://test2.js"/>
</jsp:include>
```

css也可以使用类似js的方式。