![](https://img.shields.io/travis/perwendel/spark.svg) 
![](https://img.shields.io/github/license/perwendel/spark.svg)
![](https://img.shields.io/maven-central/v/com.sparkjava/spark-core.svg)
# Spark-Web 一个基于Java8函数式编程精巧的web框架
* [开始](#开始)
* [停止服务器](#停止服务器)
* [路由](#路由)
* [Request](#request)
* [Response](#response)
* [查询参数](#查询参数)
* [Cookies](#cookies)
* [Sessions](#sessions)
* [Halting](#halting)
* [Filters](#filters)
* [Redirects](#redirects)
* [重定向](#重定向)
* [异常映射](#异常映射)
* [静态文件](#静态文件)
	* [缓存过期时间](#缓存过期时间)
* [设置自定义的请求头](#设置自定义的请求头)
* [ResponseTransformer](#responsetransformer)
* [视图和模板引擎](#视图和模板引擎)
	* [Velocity](#velocity)
	* [Freemarker](#freemarker)
	* [Mustache](#mustache)
	* [Handlebars](#handlebars)
	* [Jade](#jade)
	* [Thymeleaf](#thymeleaf)
	* [Jetbrick](#jetbrick)
	* [Pebble](#pebble)
	* [Water](#water)
* [嵌入式Web服务器](#嵌入式Web服务器)
	* [端口](#端口)
	* [安全](#安全)
	* [线程池](#线程池)
	* [等待初始化](#等待初始化)
	* [WebSockets](#websockets)
* [其他服务器](#其他服务器)
* [GZIP](#gzip)
* [Javadoc](#javadoc)
* [Examples And FAQ](#examples and faq)
* [如何自动刷新静态文件](#如何自动刷新静态文件)

###开始
##1、创建一个maven工程，然后在pom.xml中添加下面的依赖

```xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-core</artifactId>
    <version>2.5.3</version>
</dependency>
```
[对Maven不熟悉？看更多的介绍](https://sparktutorials.github.io/2015/04/02/setting-up-a-spark-project-with-maven.html)

使用其他工具

```Java
Gradle : compile "com.sparkjava:spark-core:2.5.3" //add to build.gradle
Ivy : <dependency org="com.sparkjava" name="spark-core" rev="2.5.3" conf="build" /> //ivy.xml
SBT : libraryDependencies += "com.sparkjava" % "spark-core" % "2.5.3" //build.sbt
```
####2、开始编码

```Java
import static spark.Spark.*;

public class HelloWorld {
    public static void main(String[] args) {
        get("/hello", (req, res) -> "Hello World");
    }
}
```

####3、运行，并且在浏览器中打开

	http://localhost:4567/hello

为了在控制台看到更多的输出，你可以在你的工程中[添加日志](#添加日志)

###停止服务器
通过调用stop()方法停止服务器，同时路由信息被清空

###路由
一个应用程序最主要的构建模块是一些路由的集合，一个路由由3个基本的组件组成：

* verb (HTTP请求方式)(get, post, put, delete, head, trace, connect, options)
* path (请求资源路径)(/hello, /users/:name)
* callback (回调方法)(request, response) -> { }

路由按照它们定义的顺序进行匹配，第一个匹配上的路由将会被调用

```Java
get("/", (request, response) -> {
    // Show something
});

post("/", (request, response) -> {
    // Create something
});

put("/", (request, response) -> {
    // Update something
});

delete("/", (request, response) -> {
    // Annihilate something
});

options("/", (request, response) -> {
    // Appease something
});
```

路由模式中可以包含命名参数， 是通过在请求对象的方法中添加参数

```Java
// 匹配 "GET /hello/foo" 和 "GET /hello/bar"
// request.params(":name") 得到的结果是 'foo' 或者 'bar'
get("/hello/:name", (request, response) -> {
    return "Hello: " + request.params(":name");
});
```

路由模式同样可以包含模式或者正则匹配参数

```Java
// 匹配 "GET /say/hello/to/world"
// request.splat()[0] 得到的是 'hello' 以及 request.splat()[1] 得到'world'
get("/say/*/to/*", (request, response) -> {
    return "Number of splat parameters: " + request.splat().length;
});
```

####路由概述
在Spark2.4中我们添加了实验特性
```Java
RouteOverview.enableRouteOverview(); // overview available at /debug/routeoverview/
RouteOverview.enableRouteOverview("/my/overview/path"); // available at specified path
```
>>  这些特性有很多反射的魔力，它有可能在某些版本的JDK上不能很好的运行

###Request
Request对象信息和它本身所提供的方法
```Java
	request.attributes();             // 属性的集合
	request.attribute("foo");         // foo参数的值
	request.attribute("A", "V");      // A和V参数的值
	request.body();                   // 客户端请求body中的值
	request.bodyAsBytes();            // 请求中的二进制文件
	request.contentLength();          // 请求中body的长度
	request.contentType();            // request.body的类型
	request.contextPath();            // 请求服务器路径
	request.cookies();                // cookie
	request.headers();                // HTTP请求头列表
	request.headers("BAR");           // 请求头BAR值
	request.host();                   // 主机地址
	request.ip();                     // client IP address
	request.params("foo");            // foo参数值
	request.params();                 // map类型值
	request.pathInfo();               // 路径信息
	request.port();                   // 服务器端口
	request.protocol();               // 使用的协议, e.g. HTTP/1.1
	request.queryMap();               // 请求map
	request.queryMap("foo");          // map中foo的值
	request.queryParams();            // 查询参数列表
	request.queryParams("FOO");       // value of FOO query param
	request.queryParamsValues("FOO")  // all values of FOO query param
	request.raw();                    // raw request handed in by Jetty
	request.requestMethod();          // HTTP 方式 (GET, ..etc)
	request.scheme();                 // "http"
	request.servletPath();            // servlet请求令, e.g. /result.jsp
	request.session();                // session 管理
	request.splat();                  // splat (*) parameters
	request.uri();                    // URI, e.g. "http://example.com/foo"
	request.url();                    // URL "http://example.com/foo"
	request.userAgent();              // 用户代理
```

###Response
Response对象信息和它本身所提供的方法
```Java
	response.body();               // 获取响应内容信息
	response.body("Hello");        // content中Hello值集合
	response.header("FOO", "bar"); // 请求头中FOO包含bar值的集合
	response.raw();                // raw response handed in by Jetty
	response.redirect("/example"); // 服务器重定向到 /example
	response.status();             // 响应状态
	response.status(401);          // HTTP状态码
	response.type();               // 获取响应类型
	response.type("text/xml");     // 响应类型 text/xml
```

###查询参数
查询参数的集合允许你通过它们的前缀进行分成map，例如，它可以允许你把参数分成像user[nam]和user[age]到一个user对象的map
```Java
	request.queryMap().get("user", "name").value();
	request.queryMap().get("user").get("name").value();
	request.queryMap("user").get("age").integerValue();
	request.queryMap("user").toMap();
```
###Cookies
```Java
	request.cookies();                         // 获取请求中cookies map集合
	request.cookie("foo");                     // 通过名称访问cookie
	response.cookie("foo", "bar");             // 带有某个值的cookie
	response.cookie("foo", "bar", 3600);       // set cookie with a max-age
	response.cookie("foo", "bar", 3600, true); // 安全的cookie
	response.removeCookie("foo");              // 移除 cookie
```
###Sessions
每个请求都会访问在服务器上创建的session对象，Session对象提供了一下的方法
```Java
	request.session(true)                      // 创建并返回 session
	request.session().attribute("user")        // 获取session中的 'user'对象
	request.session().attribute("user", "foo") // session中 'user'对象的foo属性
	request.session().removeAttribute("user")  // 移除session中某个对象
	request.session().attributes()             // 获取session中所有的信息
	request.session().id()                     // 获取session的id
	request.session().isNew()                  // 检查session是不是新的
	request.session().raw()                    // 返回servlet对象
```

###Halting
在使用过滤器或者路由是立即停止请求

	halt();
你可以指定特定的状态进行停止

	halt(401);
或者是添加body内容

	halt("This is the body");

或者是两项都有

	halt(401, "Go away!");

###Filters
在每个请求之前将会被调用，它可以读取到请求中的信息并读取或者修改响应信息

为了停止执行，使用halt：

```Java
before((request, response) -> {
    boolean authenticated;
    // ... check if authenticated
    if (!authenticated) {
        halt(401, "You are not welcome here");
    }
});
```
在每个请求之后将会被调用，它可以读取到请求中的信息并读取或者修改响应信息

	after((request, response) -> {
	    response.header("foo", "set by after filter");
	});

拦截器使用模式匹配，只有请求的路径匹配才会被拦截
	before("/protected/*", (request, response) -> {
	    // ... check if authenticated
	    halt(401, "Go Away!");
	});
###Redirects
你可以使用Response的redirect方法触发浏览器重定向到另一个页面

	response.redicrect("/bar");

你也可以使用特定的HTTP状态码3XX触发重定向

	response.redirect("/bar", 301);

###重定向
同时这里有非常方便的redirects API来进行重定向任务，它可以在没有Response对象的状态下使用

	// redirect a GET to "/fromPath" to "/toPath"
	redirect.get("/fromPath", "/toPath");
	
	// redirect a POST to "/fromPath" to "/toPath", with status 303
	redirect.post("/fromPath", "/toPath", Redirect.Status.SEE_OTHER);
	
	// redirect any request to "/fromPath" to "/toPath" with status 301
	redirect.any("/fromPath", "/toPath", Redirect.Status.MOVED_PERMANENTLY);

记得静态引入redirects的前缀，Spark.redirect

###异常映射
为了解决所有的路由和拦截器等的异常：

	get("/throwexception", (request, response) -> {
	    throw new YourCustomException();
	});
	
	exception(YourCustomException.class, (exception, request, response) -> {
	    // Handle the exception here
	});

###静态文件
你可以在服务器的类路径下使用staticFileLocation方法指定一个文件夹，注意这个公共的目录名称不包括URL路径，一个像`/public/css/style.css`是通过`http://{host}:{port}/css/style.css`获取到的

	staticFiles.location("/public"); // 静态文件

你也可以通过使用externalStaticFileLocation 方法指定一个额外的文件(不在类路径下)

	staticFiles.externalLocation(System.getProperty("java.io.tmpdir"));	

####缓存过期时间
你可以指定特定的缓存时间(秒),默认的这儿是没有使用缓存的

	staticFiles.expireTime(600); // ten minutes

####设置自定义的请求头
	staticFiles.header("Key-1", "Value-1");
	staticFiles.header("Key-1", "New-Value-1"); // Using the same key will overwrite value
	staticFiles.header("Key-2", "Value-2");
	staticFiles.header("Key-3", "Value-3");

###ResponseTransformer
路由映射使用handle方法把输出进行转换，可以通过集成ResponseTransformer类来转换输出格式，下面是使用Gson格式化树池JSON数据：
```Java
import com.google.gson.Gson;
public class JsonTransformer implements ResponseTransformer {
    private Gson gson = new Gson();
    @Override
    public String render(Object model) {
        return gson.toJson(model);
    }
}
```
它怎么使用的？(MyMessage是一个带有一个参数的bean)
	
	get("/hello", "application/json", (request, response) -> {
	    return new MyMessage("Hello World");
	}, new JsonTransformer());

你也可以使用Java8的引用，因为ResponseTransformer 是一个只有一个方法的接口

	Gson gson = new Gson();
	get("/hello", (request, response) -> new MyMessage("Hello World"), gson::toJson);

###视图和模板引擎
TemplateViewRoute 是通过路径(url匹配)和模板引擎实现了render方法来构建的，同时它调用了TemplateViewRoute的render的方法作为返回值来替代使用toString()方法来返回body中的数据。

这个路由的主要目的是使用模板引擎来提供一个创建通用的，可重复使用的组件来渲染输出结果

####Velocity
使用Velocity引擎把对象渲染成HTML

Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-velocity</artifactId>
    <version>2.3</version>
</dependency>
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-velocity)

####Freemarker
使用Freemarker引擎把对象渲染成HTML
Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-mustache</artifactId>
    <version>2.3</version>
</dependency>
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-mustache)


####Handlebars
使用Handlebars引擎把对象渲染成HTML
Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-handlebars</artifactId>
    <version>2.3</version>
</dependency
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-handlebars)

####Jade
使用Jade引擎把对象渲染成HTML
Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-jade</artifactId>
    <version>2.3</version>
</dependency>
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-jade)

####Thymeleaf
使用Thymeleaf引擎把对象渲染成HTML
Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-thymeleaf</artifactId>
    <version>2.3</version>
</dependency>
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-thymeleaf)


####Jetbrick
使用Jetbrick引擎把对象渲染成HTML
Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-jetbrick</artifactId>
    <version>2.3</version>
</dependency>
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-jetbrick)

####Pebble
使用Jetbrick引擎把对象渲染成HTML
Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-pebble</artifactId>
    <version>2.3</version>
</dependency>
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-pebble)

####Water
使用Water引擎把对象渲染成HTML
Maven依赖：
```Xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-water</artifactId>
    <version>2.3</version>
</dependency>
```
源代码在[Github](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-water)

###嵌入式Web服务器
单机的Spark运行在[jetty](http://eclipse.org/jetty/)嵌入式的服务器上

####端口
默认的，Spark运行的端口是4567，如果你想设置其他的端口，你需要在路由和拦截器之前设置

	port(9090);

####安全
(HTTPS/SSL)你可以使用secure方法设置安全的连接，这个应该在路由之前设置

	secure(keystoreFilePath, keystorePassword, truststoreFilePath, truststorePassword);

如果你需要更多的帮助，查看[FAQ](http://sparkjava.com/documentation.html#enable-ssl)

####线程池
你可以设置最大的线程数

	int maxThreads = 8;
	threadPool(maxThreads);

你同样可以配置最少的线程数目，以及IDE的超时时间

	int maxThreads = 8;
	int minThreads = 2;
	int timeOutMillis = 30000;
	threadPool(maxThreads, minThreads, timeOutMillis);

####等待初始化
你可以使用awaitInitialization()方法来检查服务器是否准备好接受请求，这个通常是通过一个分离的线程来做的，例如在你的服务器启动后运行一个安全检查模块。

这个方法导致当前的线程等待知道嵌入式的Jetty服务器被初始化.初始化被通过定义路由或拦截器所触发。因此，如果你只使用一个线程，不要把它放在你定义的路由和拦截器前面。

	awaitInitialization(); // Wait for server to be initialized

###WebSockets
WebSockets提供了一个在TCP连接上全双工的信息交流协议，意味着你可以在同一个连接上发送和接收信息

WebSockets只在嵌入式的Jetty服务器上工作，而且必须在常规的HTTP路由前定义。为了创建一个WebSocket路由，你需要提供一个路径和处理的类：
	
	webSocket("/echo", EchoWebSocket.class);
	init(); // Needed if you don't define any HTTP routes after your WebSocket routes

```Java
import org.eclipse.jetty.websocket.api.*;
import org.eclipse.jetty.websocket.api.annotations.*;
import java.io.*;
import java.util.*;
import java.util.concurrent.*;

@WebSocket
public class EchoWebSocket {

    // Store sessions if you want to, for example, broadcast a message to all users
    private static final Queue<Session> sessions = new ConcurrentLinkedQueue<>();
    @OnWebSocketConnect
    public void connected(Session session) {
        sessions.add(session);
    }
    @OnWebSocketClose
    public void closed(Session session, int statusCode, String reason) {
        sessions.remove(session);
    }
    @OnWebSocketMessage
    public void message(Session session, String message) throws IOException {
        System.out.println("Got: " + message);   // Print message
        session.getRemote().sendString(message); // and send it back
    }
}
```

###其他服务器
为了在Web服务器上运行Spark(替代嵌入式的Jetty服务器)，你必须实现spark.servlet.SparkApplication这个接口，你需要在init()方法中初始化路由，并且，下面的配置信息需要配置在你的web.xml中

```Xml
<filter>
    <filter-name>SparkFilter</filter-name>
    <filter-class>spark.servlet.SparkFilter</filter-class>
    <init-param>
        <param-name>applicationClass</param-name>
        <param-value>com.company.YourApplication</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>SparkFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

###GZIP
如果请求和响应头中带有gzip压缩方式，那么gzip压缩方式可以自动完成，这意味着你需要在你的响应头中设置改属性

如果你想压缩单个response,你可以手动添加到你的路由中：
	
	get("/some-path", (request, response) -> {
	    // code for your get
	    response.header("Content-Encoding", "gzip");
	});

如果你需要压缩所有的数据，你可以在拦截器后后设置

	after((request, response) -> {
	    response.header("Content-Encoding", "gzip");
	});
	
###Javadoc

从[Github](https://github.com/perwendel/spark)下载，运行：

	mvn javadoc:javadoc

在/target/site/apidocs中生成文件

###Examples And FAQ
在[Github](https://github.com/perwendel/spark/blob/master/README.md#examples)上可以找到例子

####文件上传
创建一个POST请求的文件表单

	<form method='post' enctype='multipart/form-data'>
	    <input type='file' name='uploaded_file'>
	    <button>Upload picture</button>"
	</form>

对于Spark是可以提取上传文件的，这里你必须设置一个特定的请求参数，它允许通过getPart方法从压缩的请求中获取数据

	post("/yourUploadPath", (request, response) -> {
	    request.attribute("org.eclipse.jetty.multipartConfig", new MultipartConfigElement("/temp"));
	    try (InputStream is = request.raw().getPart("uploaded_file").getInputStream()) {
	        // Use the input stream to create a file
	    }
	    return "File uploaded";
	});

[文件上传例子](https://github.com/tipsy/spark-file-upload)

###如何自动刷新静态文件
如果你使用staticFiles.location(...)方法，意味着你在你的类路径下保持你的静态文件，当你构建你的应用时静态资源被复制到target文件夹下。这就意味着你为了刷新静态文件不得不每次make\build你的工程，一个可行的方法是告诉Spark从全局路径下读取静态文件，当你刷新时这些静态文件将会自动加载。
	
	if (localhost) {
	    String projectDir = System.getProperty("user.dir");
	    String staticDir = "/src/main/resources/public";
	    staticFiles.externalLocation(projectDir + staticDir);
	} else {
	    staticFiles.location("/public");
	}

</br>
###添加日志
当你启动运行时你可能在控制台上输出以下信息

	SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
	SLF4J: Defaulting to no-operation (NOP) logger implementation
	SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
为了解决这个问题，你可以在你的项目中添加以下的依赖
```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.21</version>
</dependency
```
