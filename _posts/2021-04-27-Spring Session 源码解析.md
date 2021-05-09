---
layout: post
title:  "Spring Session 源码解析"
date:   2021-04-27 21:42:06 +0800--
categories: [分布式]
tags: [登陆验证, 分布式,]  

---

# 前言

之前项目中使用到了Spring Session来实现分布session，使用起来非常方便，只需要引入依赖并修改一下配置，在启动类上添加`@EnableRedisHttpSession`注解，之后采用平常的存储session方式即可 `session.setAttribute("loginUser",data);`。但是一直都没有很深入地了解过它的底层机制，比如：

- Session是什么时候创建，通过什么来创建的呢？
- 创建之后如何保存到Redis？
- 如何把SessionId设置到Cookie中的呢？



# 为什么要使用Spring Session

## 分布式场景下的Session一致性问题

当浏览器发起请求到服务器，服务器会生成SessionId保存在内存中，并将SessionId返回给浏览器，浏览器通过Cookie保存SessionId信息，用户每次通过浏览器访问服务器都会带上SessionId信息，这样就可以判断每次的请求是不是同一个用户，解决http协议无状态问题。

单机版的系统，比如一个Tomcat部署，是不存在Session一致性的问题，因为Session保存在一个Tomcat容器中。

但是如果在分布式的场景下，比如Nginx使用轮询方式，用户第一次请求，请求被分配到到了 A web服务器，用户在A web服务器上进行登录，并保存登录信息（Session信息），并将Session信息响应给浏览器。当用户第二次在请求的时候 通过轮询会定位到B web服务器，此时B web服务器上没有 用户的登录信息（Session信息），则会提示用户进行登录，此时便发生了session不一致的问题。


## 常见解决方案

### ip_hash负载均衡策略

> 根据Ip做hash计算，同一个ip的请求始终会定位到一台web服务器上。

优点:

- 只需要改nginx配置，不需要修改应用代码
- 负载均衡，只要hash属性的值分布是均匀的，多台服务器的负载是均衡的
- 可以支持web-server水平扩展（只需在配置中加上服务器即可,而session复制是不行的，受内存限制）

缺点

- session还是存在服务器中的，所以服务器重启可能导致部分session丢失，影响业务，如部分用户需要重新登录
- 如果服务器需要水平扩展，rehash后session重新分布，也会有一部分用户路由不到正确的session

### session复制

> Web服务器器创建Session后，会通过组播方式把Session发送到组播地址中的其他服务器上（tomcat集群自带的session复制）。

优点：

- 一个Tomcat宕机不会影响Session丢失问题，解决了ip_hash 的问题
- Nginx可以配置轮询和ip_hash，能够适应各种负载均衡策略

缺点：

- Session同步会有延迟，会影带宽
- 受限于内存资源，在大用户量，高并发，高流量场景，会占用大量内存
- 增加服务器不灵活，不易于扩展
  

### 统一存储

> 使用Session集中统一管理原理： Session不在由web服务器管理，而是统一放到一个地方集中式管理，读取和写入Session都依赖第三方软件。例如Redis、MongoDB、mysql等

优点：

- 扩展能力强，服务器宕机后Session不会丢失
- 是互联网项目，大型分布式、大并发场景下的首选方案

缺点：

- 对应用有侵入性，要增加一些配置文件
- 需要依赖第三方的库，需要搭建Redis的服务
- Spring Session + Redis实现分布式Session共享 有个非常大的缺陷, 无法实现跨域名共享session , 只能在同个域名下共享session , 因为是依赖cookie做的 , cookie 无法跨域。 spring Session一般是用于多台服务器负载均衡时共享Session的，都是同一个域名，不会跨域。你想要的跨域的登录，可能需要SSO单点登录。

如果要指定域名的话，我们只需要设置DomainName属性即可:

![image-20210225170335020](/assets/imgs/image-20210225170335020.png)



# Spring Session核心原理

## 关键类的说明

| 类名                                | 作用                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| RedisHttpSessionConfiguration       | 定义RedisOperationsSessionRepository等类的对象               |
| **SessionRepositoryFilter**         | ⭐️ 过滤器，操作session的入口类                                |
| **SessionRepositoryRequestWrapper** | **SessionRepositoryFilter内部类，包装HttpRequest请求，调用RedisOperationsSessionRepository类相关的方法都是通过其完成** |
| CookieHttpSessionIdResolver         | 这个类主要是调用DefaultCookieSerializer类的方法将sessionid存入cookie中，或者从cookie中读取sessionid，并返回给他的上一层 |
| DefaultCookieSerializer             | 这个类是真正的操作cookie的类，设置cookie的相关属性，只需要重新实例化这个类即可 |
| RedisOperationsSessionRepository    | 这个类的作用是生成session，并将session保存到redis中，另外就是根据sessionid查找session |
| RedisSession                        | 这个类就是Spring Session的真正的实例对象,这是原始的session   |



## SessionRepositoryFilter的注册

首先，我们从添加的SpringSession的配置类来看起，`@EnableRedisHttpSession`中的`@Import(RedisHttpSessionConfiguration.class)`导入了一个配置类：

- 配置类中给容器中添加了一个组件`RedisOperationsSessionRepository`：redis操作session（增删改查的封装类）
- 还添加了一个`SessionRepositoryFilter` ：session存储的过滤器，实现了Filter。
  - 每个请求过来都必须经过Filter
  - 创建的时候,就自动从容器中获取到了`sessionRepository`(存session的仓库,`RedisOperationsSessionRepository`是它的实现)
  - 还是实现了session的redis自动延期
  - 装饰者模式。



## RedisOperationsSessionRepository保存session的过程

首先大致了解一下保存session时的调用时序图(来自网络)：

![image-20210509105525068](/assets/imgs/image-20210509105525068.png)

1）调用的入口是`SessionRepositoryFilter`类的`doFilterInternal()`方法。Spring是通过责任链的模式来执行每个过滤器的`doFilterInternal()`方法。**SessionRepositoryFilter添加到FilterChain的具体步骤如下：**

- 在Servlet3.0规范中，Servlet容器启动时会自动扫描javax.servlet.ServletContainerInitializer的实现类，在实现类中我们可以定制需要加载的类。 通过注解@HandlesTypes(WebApplicationInitializer.class)，让Servlet容器在启动该类时，会自动寻找所有的WebApplicationInitializer实现类。
- insertSessionRepositoryFilter 方法通过filterName获取 SessionRepositoryFilter ，并创建了代理Filter，new DelegatingFilterProxy(filterName);（ DelegatingFilterProxy就是一个对于servlet filter的代理，用这个类的好处主要是通过Spring容器来管理servlet filter的生命周期，还有就是如果filter中需要一些Spring容器的实例，可以通过spring直接注入。）
- 然后将filter添加到FilterChain中

**SessionRepositoryFilter拦截请求的过程如下：**

- 请求被DelegatingFilterProxy : 拦截到，然后执行doFilter方法，在doFilter中找到执行的代理类。
- OncePerRequestFilter : 代理Filter执行doFilter方法，然后调用抽象方法doFilterInternal
- SessionRepositoryFilter 继承了OncePerRequestFilter，实现了doFilterInternal，这个方法一个封装一个wrappedRequest，通过执行commitSession保存session信息到redis



2）`doFilterInternal()`方法首先会对原始的请求和响应进行包装，包装类重写了getSession()方法来动态增加了redis的相关指责，然后会包装后的请求和响应放到之后的整个执行链。

![image-20210225172805192](/assets/imgs/image-20210225172805192.png)



3）SessionRepositoryRequestWrapper类的`getSession(true)`方法

> 判断是否是第一次请求应用，是的话需要创建session实例。

经过断点调试，并查看调用栈，发现执行链方法filterChain.doFilter(wrappedRequest, wrappedResponse)最终会调用到SessionRepositoryRequestWrapper类的getSession(true)方法。其中，SessionRepositoryRequestWrapper类是SessionRepositoryFilter类的一个私有的不可被继承，被重写的内部类。

getSession(boolean create) 方法主要有两块:

1. 获取session实例，如果请求头中带着sessionid，则表示不是第一次请求，是可以获取到session的。
2. 如果浏览器是第一次请求应用（没有sessionid）则获取不到session实例，需要创建session实例。在拿到生成的Session对象之后，紧接着会创建一个HttpSessionWrapper实例，并将前面生成的session传入其中，方便后面取用，然后将HttpSessionWrapper实例放入当前请求会话HttpServletRequest中，（Key是.CURRENT_SESSION，value是HttpSessionWrapper的实例）。

```java
public HttpSessionWrapper getSession(boolean create) {
  // 获取HttpSessionWrapper实例，如果可以获取到，则说明session已经生成了。就直接返回
  HttpSessionWrapper currentSession = getCurrentSession();
  if (currentSession != null) {
    return currentSession;
  }
  // 如果可以获取到session
  S requestedSession = getRequestedSession();
  // 如果HttpSessionWrapper实例为空，则需要将session对象封装到HttpSessionWrapper实例中，并设置到HttpRequestSerlvet中
  if (requestedSession != null) {
    if (getAttribute(INVALID_SESSION_ID_ATTR) == null) {
      requestedSession.setLastAccessedTime(Instant.now());
      this.requestedSessionIdValid = true;
      currentSession = new HttpSessionWrapper(requestedSession, getServletContext());
      currentSession.setNew(false);
      setCurrentSession(currentSession);
      return currentSession;
    }
  }
  // 如果获取不到session，则进入下面分支，创建session
  else {
   ...
  // 如果create为false，直接返回null
  if (!create) {
    return null;
  }
  ...
  // 如果create为true，则调用RedisOperationsSessionRepository类的createSession方法创建session实例
  S session = SessionRepositoryFilter.this.sessionRepository.createSession(); // ⚠️-> 4)中详解
  session.setLastAccessedTime(Instant.now());
  currentSession = new HttpSessionWrapper(session, getServletContext());
  setCurrentSession(currentSession);
  return currentSession;
}
```



4）RedisOperationsSessionRepository类的`createSession()`方法

从前面的代码分析我们可以知道如果获取不到session实例，则会调用createSession()方法进行创建。这个方法是在RedisOperationsSessionRepository类中，主要就是实例化RedisSession对象。其中RedisSession对象中包括了sessionid，creationTime，maxInactiveInterval和lastAccessedTime等属性。其中原始的sessionid是一段唯一的UUID字符串。

```java
@Override
public RedisSession createSession() {
  // 设置session的失效时间
  Duration maxInactiveInterval = Duration
    .ofSeconds((this.defaultMaxInactiveInterval != null)
               ? this.defaultMaxInactiveInterval
               : MapSession.DEFAULT_MAX_INACTIVE_INTERVAL_SECONDS);
  // 实例化RedisSession对象
  RedisSession session = new RedisSession(maxInactiveInterval);
  session.flushImmediateIfNecessary();
  return session;
}

RedisSession(Duration maxInactiveInterval) {
  this(new MapSession());
  this.cached.setMaxInactiveInterval(maxInactiveInterval);
  this.delta.put(CREATION_TIME_ATTR, getCreationTime().toEpochMilli());
  this.delta.put(MAX_INACTIVE_ATTR, (int) getMaxInactiveInterval().getSeconds());
  this.delta.put(LAST_ACCESSED_ATTR, getLastAccessedTime().toEpochMilli());
  this.isNew = true;
}
```



5）SessionRepositoryRequestWrapper类内部的`commitSession()`方法

doFilterInternal方法在调用完其他方法之后，在finally代码块中会调用SessionRepositoryRequestWrapper类内部的commitSession()方法，而commitSession()方法会保存session信息到Redis中，并将sessionid写到cookie中。我们接着来看看commitSession()方法。

第一步就是从当前请求会话中获取HttpSessionWrapper对象的实例，如果实例获取不到则向Cookie中写入一个空值。如果可以获取到实例的话，则从实例中获取Session对象。获取到Session对象之后则调用RedisOperationsSessionRepository类的save(session)方法将session信息保存到Redis中，其中redis的名称前缀是spring:session。将数据保存到Redis之后，紧接着获取sessionid，最后调用CookieHttpSessionIdResolver类的setSessionId方法将sessionid设置到Cookie中。

```java
private void commitSession() {
  // 当前请求会话中获取HttpSessionWrapper对象的实例
  HttpSessionWrapper wrappedSession = getCurrentSession();
  // 如果wrappedSession为空则调用expireSession写入一个空值的cookie
  if (wrappedSession == null) {
    if (isInvalidateClientSession()) {
      SessionRepositoryFilter.this.httpSessionIdResolver.expireSession(this,this.response);
    }
  }
  else {
    // 获取session
    S session = wrappedSession.getSession();
    clearRequestedSessionCache();
    SessionRepositoryFilter.this.sessionRepository.save(session);
    String sessionId = session.getId();
    if (!isRequestedSessionIdValid()
        || !sessionId.equals(getRequestedSessionId())) {
      SessionRepositoryFilter.this.httpSessionIdResolver.setSessionId(this, this.response, sessionId);//⚠️
    }
  }
}
```



6）CookieHttpSessionIdResolver类的`setSessionId`方法

setSessionId方法主要就是将生成的sessionid设置到请求会话中，然后调用DefaultCookieSerializer类的writeCookieValue方法将sessionid设置到cookie中。

```java
@Override
public void setSessionId(HttpServletRequest request, HttpServletResponse response,
                         String sessionId) {
  //如果sessionid等于请求头中的sessionid，则直接返回
  if (sessionId.equals(request.getAttribute(WRITTEN_SESSION_ID_ATTR))) {
    return;
  }
  //将sessionid设置到请求头中
  request.setAttribute(WRITTEN_SESSION_ID_ATTR, sessionId);
  //将sessionid写入cookie中
  this.cookieSerializer
    .writeCookieValue(new CookieValue(request, response, sessionId)); // ⚠️
}
```

7）DefaultCookieSerializer类的`writeCookieValue`方法

```java
@Override
public void writeCookieValue(CookieValue cookieValue) {
  HttpServletRequest request = cookieValue.getRequest();
  HttpServletResponse response = cookieValue.getResponse();
  //设置cookie的名称，默认是SESSION
  StringBuilder sb = new StringBuilder();
  sb.append(this.cookieName).append('=');
  String value = getValue(cookieValue);
  if (value != null && value.length() > 0) {
    validateValue(value);
    sb.append(value);
  }
  //设置cookie的失效时间
  int maxAge = getMaxAge(cookieValue);
  if (maxAge > -1) {
    sb.append("; Max-Age=").append(cookieValue.getCookieMaxAge());
    ZonedDateTime expires = (maxAge != 0)
      ? ZonedDateTime.now(this.clock).plusSeconds(maxAge)
      : Instant.EPOCH.atZone(ZoneOffset.UTC);
    sb.append("; Expires=")
      .append(expires.format(DateTimeFormatter.RFC_1123_DATE_TIME));
  }
  //设置Domain属性，默认就是当前请求的域名，或者ip
  String domain = getDomainName(request);
  if (domain != null && domain.length() > 0) {
    validateDomain(domain);
    sb.append("; Domain=").append(domain);
  }
  //设置Path属性，默认是当前项目名（例如：/spring-boot-session），可重设
  String path = getCookiePath(request);
  if (path != null && path.length() > 0) {
    validatePath(path);
    sb.append("; Path=").append(path);
  }
  if (isSecureCookie(request)) {
    sb.append("; Secure");
  }
  //设置在HttpOnly是否只读属性。
  if (this.useHttpOnlyCookie) {
    sb.append("; HttpOnly");
  }
  if (this.sameSite != null) {
    sb.append("; SameSite=").append(this.sameSite);
  }
  //将设置好的cookie放入响应头中
  response.addHeader("Set-Cookie", sb.toString());
}
```

---

## 读取session的过程

还是大致了解一下保读取session时的调用时序图(来自网络)：

首先代码入口还是SessionRepositoryFilter过滤器的doFilterInternal方法。这个方法里还是会调用到SessionRepositoryRequestWrapper类的getSession()方法，这个getSession方法是读取Session的开始，这个方法内部会调用getSession(true)方法。那我们就从SessionRepositoryRequestWrapper类的getSession(true)方法开始说起。

![在这里插入图片描述](/assets/imgs/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ1MzQ4MDg=,size_16,color_FFFFFF,t_70.png)

1）getSession(true)方法

这个方法首先获取HttpSessionWrapper对象，这个对象的作用是用于封装session，返回给其上一层，如果可以获取到则说明Session信息已经拿到了，就直接返回。如果获取不到则调用getRequestedSession()方法。这个方法就是获取session的主方法。接着让我们来看看这个方法吧。

```java
@Override
public HttpSessionWrapper getSession(boolean create) {
  //获取HttpSessionWrapper类，这个类会包装HttpSession
  HttpSessionWrapper currentSession = getCurrentSession();
  if (currentSession != null) {
    return currentSession;
  }
  //获取RedisSession
  S requestedSession = getRequestedSession(); // ⚠️->(2
  if (requestedSession != null) {
    if (getAttribute(INVALID_SESSION_ID_ATTR) == null) {
      requestedSession.setLastAccessedTime(Instant.now());
      this.requestedSessionIdValid = true;
      currentSession = new HttpSessionWrapper(requestedSession, getServletContext());
      currentSession.setNew(false);
      setCurrentSession(currentSession);
      return currentSession;
    }
  }
  //...
  S session = SessionRepositoryFilter.this.sessionRepository.createSession(); // ⚠️ 这边这是上文保存session的过程
  session.setLastAccessedTime(Instant.now());
  currentSession = new HttpSessionWrapper(session, getServletContext());
  setCurrentSession(currentSession);
  return currentSession;
}

```



2）getRequestedSession()方法

如上，这个方法主要有两步：

1. 从cookie中获取sessionid的集合，可能cookie中存在多个sessionid。
2. 循环sessionid的集合，分别根据sessionid到redis中获取session。获取sessionid是通过HttpSessionIdResolver接口的resolveSessionIds方法来实现的，SessionRepositoryFilter中定义了HttpSessionIdResolver接口的实例，其实现类是CookieHttpSessionIdResolver类。

```java
private S getRequestedSession() {
  if (!this.requestedSessionCached) {
    //从cookie中获取sessionid集合
    List<String> sessionIds = SessionRepositoryFilter.this.httpSessionIdResolver
      .resolveSessionIds(this); // ⚠️->(3
    //遍历sessionid集合，分别获取HttpSession
    for (String sessionId : sessionIds) {
      if (this.requestedSessionId == null) {
        this.requestedSessionId = sessionId;
      }
      //根据sessionid去redis中获取session
      S session = SessionRepositoryFilter.this.sessionRepository
        .findById(sessionId); // ⚠️->(4
      if (session != null) {
        this.requestedSession = session;
        this.requestedSessionId = sessionId;
        break;
      }
    }
    this.requestedSessionCached = true;
  }
  return this.requestedSession;
}
```



3）resolveSessionIds方法

接下来，我们就来到了`CookieHttpSessionIdResolver`类的`resolveSessionIds`方法，这个方法主要的作用就是从cookie中获取sessionid。

```java
@Override
public List<String> resolveSessionIds(HttpServletRequest request) {
	return this.cookieSerializer.readCookieValues(request);
}
```

看到这个方法之后，我们发现这个方法只是一个中转方法，内部直接把请求交给了readCookieValues方法。同样的在CookieHttpSessionIdResolver类内部也定义了cookieSerializer这个属性，它的实例对象是DefaultCookieSerializer。所以，真正的操作逻辑还是在DefaultCookieSerializer类中完成的。

这个从cookie中获取sessionid的方法也很简单，无非就是从当前的HttpServletRequest对象中获取所有的cookie，然后，提取name等于cookieName的cookie值。这个cookie值就是sessionid。

```java
@Override
public List<String> readCookieValues(HttpServletRequest request) {
	//从请求头中获取cookies
	Cookie[] cookies = request.getCookies();
	List<String> matchingCookieValues = new ArrayList<>();
	if (cookies != null) {
		for (Cookie cookie : cookies) {
			//获取存放sessionid的那个cookie，cookieName默认是SESSION
			if (this.cookieName.equals(cookie.getName())) {
				//默认的话sessionid是加密的
				String sessionId = (this.useBase64Encoding
						? base64Decode(cookie.getValue())
						: cookie.getValue());
				if (sessionId == null) {
					continue;
				}
				if (this.jvmRoute != null && sessionId.endsWith(this.jvmRoute)) {
					sessionId = sessionId.substring(0,
							sessionId.length() - this.jvmRoute.length());
				}
				matchingCookieValues.add(sessionId);
			}
		}
	}
	return matchingCookieValues;
}
```



4）findById方法

从cookie中那个sessionid之后会调用RedisOperationsSessionRepository类的`findById`方法，这个方法的作用就是从redis中获取保存的session信息。

1. 根据sessionid获取当前session在redis保存的所有数据
2. 传入数据并组装成MapSession
3. 将MapSession在转成RedisSession,并最终返回

```java
public RedisSession findById(String id) {
  //直接调用getSession方法
  return getSession(id, false);
}

private RedisSession getSession(String id, boolean allowExpired) {
  //获取当前session在redis保存的所有数据
  Map<Object, Object> entries = getSessionBoundHashOperations(id).entries();
  if (entries.isEmpty()) {
    return null;
  }
  //传入数据并组装成MapSession
  MapSession loaded = loadSession(id, entries);
  if (!allowExpired && loaded.isExpired()) {
    return null;
  }
  //将MapSession在转成RedisSession,并最终返回
  RedisSession result = new RedisSession(loaded);
  result.originalLastAccessTime = loaded.getLastAccessedTime();
  return result;
}

private BoundHashOperations<Object, Object, Object> getSessionBoundHashOperations(
  String sessionId) {
  //拿到key
  String key = getSessionKey(sessionId);
  //根据key获取值
  return this.sessionRedisOperations.boundHashOps(key);
}
//key是spring:session sessions:+sessionid
String getSessionKey(String sessionId) {
  return this.namespace + "sessions:" + sessionId;
}




---



# SpringSession中的设计模式

## 装饰者模式

**装饰者模式又名包装模式，以对客户端透明的方式拓展对象的功能，能够让我们在不修改底层代码的情况下，给我们的对象赋予新的职责。是继承关系的一个替代方案。**

![img](/assets/imgs/auto-orient,1.jpeg)

- **Component**：抽象组件，装饰者和被装饰者共同的父类，是一个接口或者抽象类，用来定义基本行为，可以给这些对象动态添加职责
- **ConcreteComponent**：具体的组件对象，实现类 ，即**被装饰者**，通常就是被装饰器装饰的原始对象，也就是可以给这个对象添加职责
- **Decorator**：**所有装饰器的抽象父类，一般是抽象类，实现接口；它的属性必然有个指向 Conponent 抽象组件的对象 ，其实就是持有一个被装饰的对象**。装饰模式的**核心在于抽象装饰类的设计**。
- **ConcreteDecorator**：**具体的装饰对象，实现具体要被装饰对象添加的功能。**每一个具体装饰类都定义了一些新的行为，它可以调用在抽象装饰类中定义的方法，并可以增加新的方法用以扩充对象的行为。

装饰者和被装饰者对象有相同的父类，因为装饰者和被装饰者必须是一样的类型，**这里利用继承是为了达到类型匹配，而不是利用继承获得行为**。

利用继承设计子类，只能在编译时静态决定，并且所有子类都会继承相同的行为；利用组合的做法扩展对象，就可以在运行时动态的进行扩展。**装饰者模式遵循开放-关闭原则：类应该对扩展开放，对修改关闭。利用装饰者，我们可以实现新的装饰者增加新的行为而不用修改现有代码，而如果单纯依赖继承，每当需要新行为时，还得修改现有的代码。**

类 `ServletRequestWrapper` 的代码如下：

```java
public class ServletRequestWrapper implements ServletRequest {
  
    private ServletRequest request; // 对其进行了包装
    
    public ServletRequestWrapper(ServletRequest request) {
        if (request == null) {
            throw new IllegalArgumentException("Request cannot be null");
        }
        this.request = request;
    }
    
    @Override
    public Object getAttribute(String name) {
        return this.request.getAttribute(name);
    }
    //...
}    
```
可以看到类`ServletRequestWrapper` 对 `ServletRequest` 进行了包装，这里是一个装饰者模式。

再看下图，spring session 中 `SessionRepositoryFilter` 的一个内部类 `SessionRepositoryRequestWrapper` 与 `ServletRequestWrapper` 的关系，可见 `ServletRequestWrapper` 是第一层包装，`HttpServletRequestWrapper` 通过继承进行包装，增加了 HTTP 相关的功能，`SessionRepositoryRequestWrapper` 又通过继承进行包装，增加了 Session 相关的功能。

![image-20210509140132559](/assets/imgs/image-20210509140132559.png)



## 责任链模式

责任链模式是一种对象的行为模式。责任链模式中的每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推，最典型的就是Servlet中的Filter。

责任链模式的优点大致有以下几点：

- 实现了请求发送者与请求处理者之间的松耦合
- 可动态添加责任对象、删除责任对象、改变责任对象顺序，非常灵活
- 每个责任对象专注于做自己的事情，职责明确

最佳应用场景：有多个对象可以处理同一个请求时，比如:多级请求、请假/加薪等审批流程、JavaWeb中Tomcat对 Encoding 的处理、拦截器。



参考：

[分布式Session一致性入门简介](https://blog.csdn.net/u010648555/article/details/80531679)

[Spring-Session实现Session共享实现原理以及源码解析](https://aflyun.blog.csdn.net/article/details/79491988?utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-1.vipsorttest&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-1.vipsorttest)

[SpringSession的源码解析（生成session，保存session，写入cookie全流程分析）](https://blog.csdn.net/u014534808/article/details/105479385?ops_request_misc=%7B%22request%5Fid%22%3A%22161952351116780261975901%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=161952351116780261975901&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-1-105479385.pc_search_result_no_baidu_js&utm_term=spring+session+怎么生成session)

