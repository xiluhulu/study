# JavaWeb

## 问题归纳

### Tomcat乱码问题

就是修改tomcat的conf下的logging.properties中的参数

```properties
java.util.logging.ConsoleHandler.encoding = GBK
```

### idea控制台乱码问题

需要在服务器配置中的==VM options==加入：

```
-Dfile.encoding=UTF-8
```

## web.xml的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--version:版本号-->
<!--encoding:xml的当前文件编码格式-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
<!--    用于配置servlet程序-->
    <servlet>
<!--        servlet的别名-->
        <servlet-name>Servlet</servlet-name>
<!--        servlet的全类名：包名到类名-->
        <servlet-class>com.Servlet</servlet-class>
<!--        初始化参数-->
        <init-param>
<!--            参数名-->
            <param-name>asdsss</param-name>
<!--            参数值-->
            <param-value>zxzxczxczxcxc</param-value>
        </init-param>
    </servlet>
<!--    用于配置servlet程序的访问地址-->
    <servlet-mapping>
<!--        用于给服务器识别，当前配置的地址给哪个servlet程序使用（与别名相同）-->
        <servlet-name>Servlet</servlet-name>
<!--        /斜杠在服务器解析的时候，表示地址为：http：//ip:port/工程路径-->
<!--        /asd 表示地址为：http://ip:port/工程路径/asd-->
        <url-pattern>/asd</url-pattern>
    </servlet-mapping>
</web-app>
```



## Servlet的生命周期

1. 执行Servlet构造器方法
2. 执行init初始化方法
   - 第一二步，是在第一次访问，的时候创建Servlet程序会调用
3. 执行service方法
   - 每次访问都会被调用
4. 执行destroy销毁方法
   - 只有在web服务停止的时候调用

## HttpServlet

- 会自动进行请求类型的分发，不需要手动分发

## ServletConfig

- Servlet配置信息类

- Servlet程序和ServletConfig对象有Tomcat负责创建，自己负责使用

- Servlet程序默认是第一次访问时创建的，ServletConfig是每一个Servlet程序创建时，就创建一个对应的ServletConfig对象

- 主要是在初始化中使用

- 也可以在其他方法中使用

  ```java
  getServletConfig();
  ```

  

### ServletConfig三大作用

1. 可以获取Servlet程序的别名servlet-name的值

2. 获取初始化参数init-param

3. 获取ServletContext对象

   ```java
       @Override
       public void init(javax.servlet.ServletConfig config) throws ServletException {
   //        必须调用父类
           super.init(config);
           //        获取servlet在web.xml中的别名
           System.out.println("ServletConfig别名为："+config.getServletName());
   //        获取servlet在web.xml中的初始化参数参数名为（xxx）的值
           System.out.println("初始化参数xxx的值为："+config.getInitParameter("xxx"));
   //        获取servletContext对象
           System.out.println("servletContext对象："+config.getServletContext());
       }
   ```

   

## SevletContext

- SevletContext是一个接口，表示Sevlet上下文对象

- 一个web工程只有一个SevletContext对象实例

- SevletContext对象是一个域对象

- 作用区域为整个web工程，所有的servlett都可调用

- SevletContext在web工程部署启动时创建。停止时销毁。

  |         | 存数据         | 取数据         | 删数据            |
  | ------- | -------------- | -------------- | ----------------- |
  | Map对象 | put()          | get()          | remove()          |
  | 域对象  | setAttritube() | getAttribute() | removeAttribute() |

  

### SevletContext类的四个作用

1. 获取web.xml中的上下文参数
   1. 这里的上下文指的是<context-param>中的值

2. 获取当前工程路径,格式为:/工程路径
3. 获取工程的绝对路径
4. 像MAP一样存储数据

实例：

```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//      获取servletContext对象
        javax.servlet.ServletContext context = getServletContext();
//        通过对象获取上下文参数context1的值
        System.out.println("context-param参数context1的值为："+context.getInitParameter("context1"));
//        通过对象获取上下文参数context2的值
        System.out.println("context-param参数context2的值为："+context.getInitParameter("context2"));
//        获取当前的工程路径，格式：/工程路径
        System.out.println("当前工程路径："+context.getContextPath());
//        获取工程部署后在服务器硬盘上的绝对路径
//        /斜杠被服务器解析为：http://ip:port/工程名/     映射到idea的web目录中
        System.out.println("获取当前工程部署路径："+context.getRealPath("/"));
//        通过上下文对象获取未保存参数名为key（未设置）的值，输出结果应为空
        System.out.println("key 设置之前的值为"+context.getAttribute("key"));
//        设置上下文对象设置参数名为key的值为value
        context.setAttribute("key","value");
//        再次通过上下文对象获取参数名为key（已设置）的值
        System.out.println("key 设置之后的值为"+context.getAttribute("key"));

    }
```

```xml
<!--    上下文参数工程中共享（位于<servlet>上方）-->
    <context-param>
        <param-name>context1</param-name>
        <param-value>c11111111</param-value>
    </context-param>
    <context-param>
        <param-name>context2</param-name>
        <param-value>c22222222</param-value>
    </context-param>
```

## Http协议

## Servlet-HttpServletRequest常用API

- 获取请求的资源路径
- 获取请求的统一资源定位符（绝对路径）
- 获取客户端请求的ip地址
- 获取请求头
- 获取请求的参数
- 获取请求的参数（多个值使用）
- 获取请求的方式GET或POST
- 获取请求转发对象

代码实例：

```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        获取请求的资源路径
        System.out.println("请求的资源路径："+req.getRequestURI());
//        获取请求的统一资源定位符（绝对路径）
        System.out.println("请求的统一资源定位符："+req.getRequestURL());
//        获取客户端请求的ip地址
        System.out.println("客户端的ip地址："+req.getRemoteHost());
//        获取请求头（参数为请求头的key）
        System.out.println("请求头："+req.getHeader("User-Agent"));
        System.out.println("请求头："+req.getHeader("Host"));
//        获取请求的方式GET或POST
        System.out.println("请求的方式："+req.getMethod());
//        获取请求的参数（参数为表单所对应的name）
        String username = req.getParameter("username");
//        获取请求的参数
        String password = req.getParameter("password");
//        获取请求的参数（多个值使用）
        String[] hobbies = req.getParameterValues("hobby");
        System.out.println("用户名："+username);
        System.out.println("密码："+password);
        System.out.println("爱好："+ Arrays.asList(hobbies));
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
//        设置字符编码不然中文会乱码
        req.setCharacterEncoding("UTF-8");
//        以下与get请求相同
        System.out.println("请求的方式："+req.getMethod());
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        String[] hobbies = req.getParameterValues("hobby");
        System.out.println("用户名："+username);
        System.out.println("密码："+password);
        System.out.println("爱好："+ Arrays.asList(hobbies));
    }
```

```html
 <body>
  <h1>get</h1>
    <form action="/JavaWeb01/RequestAPI" method="get">
      用户名：<input type="text" name="username"><br>
      密码：<input type="text" name="password"><br>
      爱好：<input type="checkbox" name="hobby" value="啊啊啊">啊啊啊
      <input type="checkbox" name="hobby" value="杀杀杀">杀杀杀
      <input type="checkbox" name="hobby" value="顶顶顶">顶顶顶<br>
      <input type="submit">
    </form>
    <h1>post</h1>
    <form action="/JavaWeb01/RequestAPI" method="post">
      用户名：<input type="text" name="username"><br>
      密码：<input type="text" name="password"><br>
      爱好：<input type="checkbox" name="hobby" value="啊啊啊">啊啊啊
      <input type="checkbox" name="hobby" value="杀杀杀">杀杀杀
      <input type="checkbox" name="hobby" value="顶顶顶">顶顶顶<br>
      <input type="submit">
    </form>
  </body>
```

### 请求转发

是指服务器受到请求后，从一次资源跳转到另一个资源的操作

特点：

- 浏览器地址没有变化
- 此次请求为一次请求并非多次
- 共享Request域中的数据
- 可以转发到WEB-INF目录下的资源
- 不可以访问工程以外的资源

代码实例：

```java
//1.
    public class RequestDispatcher1 extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//        拿到参数
        String username = request.getParameter("username");
        System.out.println("username:"+username);
//        设置域中参数
        request.setAttribute("key","asdasdasd");
//        请求转发到RequestDispatcher2中（给定目标对象）
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/RequestDispatcher2");
//        将带着请求参数走向指定目标，并且带回响应参数
        requestDispatcher.forward(request,response);
//        回来后可继续执行以下方法
        System.out.println("回来不？");
    }
}

//2.
public class RequestDispatcher2 extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//        这是1带来的请求参数
        String username = request.getParameter("username");
        System.out.println("我是2号，你的名字是："+username);
//        这是1设置的域参数
        Object key = request.getAttribute("key");
        System.out.println("我也得到了key:"+key);
    }
}
```

```html
<form action="/JavaWeb01/RequestDispatcher1" method="get">
    用户名：<input type="text" name="username"><br>
    <input type="submit">
</form>
```

## \<base>标签

为当前页面设置参照标签，这样就不会参照浏览器地址栏中的标签了

## /斜杠的不同意义

- 浏览器解析得到的地址是：http://ip:port/
- 服务器解析得到的地址是：http://ip:port/工程路径
- 特殊情况：response.sendRediect("/") 		把斜杠发给浏览器解析得到的地址是：http://ip

## Servlet-HttpServletResponse

### 两个输出流

| 输出流 | 方法              | 应用类型                     |
| ------ | ----------------- | ---------------------------- |
| 字节流 | getOutputStream() | 常用于下载（传递二进制数据） |
| 字符流 | getWriter()       | 常用于回传字符串（常用）     |

注意事项：两个流只能同时使用一个；使用了 字节流，就不能再使用字符流，否则报错，另一个同理。

### 响应乱码问题

```java
//GET与POST都适用
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//        通过响应头，设置浏览器地址也能使用UTF-8字符集
        response.setHeader("Content-Type","text/html;charset=UTF-8");
//        设置服务器字符集为UTF-8
        response.setCharacterEncoding("UTF-8");
        
//        同时设置服务器和客户端都使用UTF-8字符集，还设置了响应头（推荐使用）
//        需在获取流对象前使用
        response.setContentType("text/html;charset=UTF-8");
    }
```

### 请求重定向

- 响应的不是当前地址，而是定向后的地址
- 相当于多次请求
- 域对象中的参数不共享

代码实例：

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//        方法一：（不推荐使用）
//        设置响应状态码302，表示重定向
        response.setStatus(302);
//        设置响应头，标明新地址并转过去
        response.setHeader("Location","/JavaWeb01/SendRedirect2");
//        方法二：（推荐使用）
//        已经设置好状态码可直接使用
        response.sendRedirect("/JavaWeb01/SendRedirect2");
    }
```

## Cookie

- cookie是服务器通知客户端保存键值对的一种技术
- 客户端有了cookie，每次请求都会发给服务器
- 大小不能超过4kb
- cookie可以创建多个
- cookie的值不应该包含空格、方括号、圆括号、等号、逗号、双引号、斜杠、问号、@符号、冒号和分号。空值在所有浏览器上的行为不一定相同。

### cookie的创建

```java
//        创建cookie对象
        Cookie cookie = new Cookie("key1", "value1");
//        响应并通知通知浏览器保存cookie
        response.addCookie(cookie);
```

### cookie的获取

- 获取后不是一个特定的值，而是一个数组，但又不是数组
- 只能通过遍历寻找对应的键值对
- 有getName方法与getValue方法
- 可自行封装一个查找工具类

简单的获取cookie：

```java
//        1获取cookie数组
        Cookie[] cookies = request.getCookies();
//        2通过循环遍历找到指定的cookie
        for (Cookie cookie:cookies) {
            System.out.println(cookie.getName()+"="+cookie.getValue());
        }

```

获取指定的cookie：

```java
//        1获取cookie数组
        Cookie[] cookies = request.getCookies();
//        设置变量用于给cookie赋值
        String c = null;
//        2通过循环遍历找到指定的cookie
        for (Cookie cookie:cookies) {
            if (cookie.getName().equals("key1")){
                c=cookie.getValue();
            }
        }
        System.out.println(c);
```

cookie指定值查找方法的封装：（只能用于查找值）

```java
 public static String findCookie(String name, Cookie[] cookies){
     //判断数组，键名是否为空
        if(name==null || cookies==null || cookies.length==0){
            return null;
        }
     //遍历查找
        for (Cookie cookie : cookies) {
            if (name.equals(cookie.getName())){
                return cookie.getValue();
            }
        }
     //查找不到则返回空
        return null;
    }
```

cookie指定对象查找方法的封装：（推荐）

```java

```

### cookie的修改：

- 方案一（相当于覆盖）：

  1. 先创建一个要修改的同名的cookie对象
  2. 在构造器，同时赋予新的cookie值

  ```java
  //        1创建与需要修改cookie的同名对象
          Cookie cookie = new Cookie("key1", "newValue");
  //        通知浏览器保存cookie
          response.addCookie(cookie);
  
  ```

- 方案二（真正意义上的修改）：

  1. 先找到需要修改的cookie对象
  2. 调用setValue方法赋予新的值

  ```java
  //        1先查找cookie对象
          Cookie[] cookies = request.getCookies();
          Cookie key1Obj = CooieUtile.findCookieObj("key1", cookies);
  //        判断cookie对象是否为空
          if (key1Obj!=null) {
  //            设置cookie的值
              key1Obj.setValue("newValue2");
  //            通知浏览器修改cookie
              response.addCookie(key1Obj);
          }
  ```

  ### cookie的生命控制

  - 指的是管理cookie说明时候被销毁（删除）
  - 使用setMaxAge方法实现
    - 正值，表示cookie将在经过该表示的秒数后过期
    - 负值，表示浏览器一关，cookie就会被销毁
    - 0，表示马上删除cookie

```java
//        1.新建cookie对象
        Cookie cookie = new Cookie("life3600", "life3600");
//        通过cookie对象设置时间
        cookie.setMaxAge(60*60);
//        通知浏览器保存cookie
        response.addCookie(cookie);
```

### cookie有效路径path的设置

- path属性是通过请求的地址来进行有效的过滤

过滤条件如下：

```
cookieA：path=/工程路径
cookieB：path=/工程路径/abc
	请求地址如下：
		http://ip:port/工程路径/a.html
			cookieA	发送
			cookieB	不发送
		http://ip:port/工程路径/abc/a.html
			cookieA	发送
			cookieB	发送
	
```

## Session会话

- 用来维护一个客户端和服务器之间关联的一种技术
- 每个客户端都有自己的session会话
- session会话中，我们经常用来保存用户登录之后的信息
- session类似一个数组，当设置超时时长后所有的session的超时时长都会被统一，无法像cookie一样单独设置

### session的创建与获取

- request.getSession()：
  - 第一次调用是：创建session会话
  - 之后调用都是：获取前面创建好的session会话对象
- isNew()：判断是不是刚创建出来的
  - true：表示刚创建
  - false：表示获取之前创建
- 每个会话都有一个id，且这个id是唯一的
- getId()得到session的会话id值

```java
 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setHeader("Content-Type","text/html;charset=utf-8");
     //创建session
        HttpSession session = request.getSession();
     //判断session是否为第一次创建
        boolean aNew = session.isNew();
    // 获取session对象的id
        String id = session.getId();
        response.getWriter().write("得到的session，id："+id+"<br>");
        response.getWriter().write("是否为新创建：："+aNew+"<br>");
    }
```

session中存数据与取数据 ：

```java
//        往session中存储数据
        request.getSession().setAttribute("key1","value1");

//        获取session域中的数据
        Object key1 = request.getSession().getAttribute("key1");
        System.out.println(key1);
```

### session生命周期

- 客户端关闭session会被销毁
- setMaxInactiveInterval()：设置session的超时时长，以秒为单位，超过指定时常，session就会被销毁
  - 正值：设定超时时长
  - 负值：永不销毁（少用）
- getMaxInactiveInterval()：获取session的超时时间
- invalidate()：让当前session会话马上超时无效
- session默认超时时长为30分钟
- session的超时指的是，客户端两次请求的最大间隔时长

- 更改当前工程的所有session的默认超时时长可以在web.xml中修改

```xml
<!--与<servlet>同级-->
<session-config>
    <session-timeout>默认时间（单位：分钟）</session-timeout>
</session-config>
```

使用setMaxInactiveInterval()设置超时时长：

```java
//          创建session
        HttpSession session = request.getSession();
//        设置超时时长为3秒
        session.setMaxInactiveInterval(3);
```

使用invalidate()让session马上销毁：

```java
//          创建session
        HttpSession session = request.getSession();
//        session马上销毁
        session.invalidate();
```

## jsp（作用不大）

本质就是一个servlet程序

需要导入jsp的jar包，此包就在Tomcat内

### jsp头部的page命令（一般不做修改）

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

包含的属性如下：

- language：只支持java（不做修改）
- contentType：默认==text/html;charset=UTF-8==（不做修改）
- pageEncoding：设置当前页面的字符集，一般用utf-8
- import：用于导包、类
- autoFlush：缓冲区是否自动刷新,默认true（不做修改）
- buffer：默认8kb（不做修改）
- errorPage：当jsp页面运行时出错，自动跳转去指定页面（相当于404）
- isThreadSafe：是否可多线程访问，默认true
- isErrorPage：设置当前jsp是否为错误信息页面，默认为false，（一般不用）
- session：是否会创建HttpSession对象，默认true（不做修改）
- extends：jsp翻译出的Java类继承哪个Java类（不做修改）

### jsp常用脚本

#### 声明脚本

作用：可以给jsp翻译出来的Java类定义属性和方法甚至是静态代码块，内部类等等

可用于：

- 声明类属性
- 声明static静态代码块
- 声明类方法
- 声明内部类

格式：

```jsp
<%! 声明Java代码 %>
```

#### 表达式脚本(极少使用)

作用：用于在jsp页面输出数据

表达式脚本的特定：

- 所有表达式脚本都会被翻译到_jspService()方法中
- 表达式脚本都会被翻译成out.print()输出到页面上
- 由于表达式脚本翻译的内容都在\_jspService()方法中，所以\_jspService()方法中的对象都可以直接使用
- 表达式脚本中的表达式不能以分号结束

格式：

```jsp
<%=表达式%>
```

实例代码：

```jsp
<%--输出整型--%>
<%=12%>
<%--输出浮点型--%>
<%=12.132%>
<%--输出字符串--%>
<%="asd"%>
<%--输出对象--%>
<%=map%>
<%--输出对象--%>
<%=request......%>
```

#### 代码脚本

作用：可以在jsp页面中，写Java语句，定义变量、方法、循环、判断等等

特点：

- 代码脚本翻译之后都在\_jspService()方法中
- 代码脚本由于翻译到\_jspService()方法中，所以在\_jspService()中的现有对象可以直接使用
- 还可以由多个代码脚本块组合完成一个完整的java语句

格式：

```jsp
<% java语句 %>
```

#### 九大内置对象

jsp中的内置对象，是指Tomcat在翻译jsp页面成为Servlet源码后，内部提供的九大对象，叫内置对象。

- request				请求对象
- response			响应对象
- pageContext		jsp的上下文对象
- session				会话对象
- application		ServletContext对象
- config				ServletConfig对象
- out					jsp输出流对象
- page				指向当前jsp对象
- exception		异常对象

#### jsp四大域对象

- pageContext			 （PageContextlmpl类）			当前jsp页面内有效
- rerquest					（HttpServletRequest类）		一次请求内有效
- session					 （HttpSession类）					一个会话范围内有效（访问服务到浏览器关闭）
- application				（ServletContext类）				整个WEB工程范围内都有效（只要web工程不停止，数据都在）

有效范围：

- 当1jsp页面请求转发2jsp页面：
  - 1jsp：pageContext：!null			 rerquest：!null					session：!null				application：!null	 
  - 2jsp：pageContext：null			 rerquest：!null					session：!null				application：!null
- 当2jsp再次单独请求，非请求转发：
  - 2jsp：pageContext：null			 rerquest：null					session：!null				application：!null
- 当关闭浏览器2jsp再次单独请求，非请求转发：
  - 2jsp：pageContext：null			 rerquest：null					session：null				application：!null
- 当关闭或重启服务2jsp再次单独请求，非请求转发：
  - 2jsp：pageContext：null			 rerquest：null					session：null				application：null

使用的优先顺序：

pageContext\==》rerquest\==》session\==》application

(在某个功能中尽量选择范围小的使用，可以更快的释放资源，从而达到减轻服务器压力的作用)

#### jsp的out输出和response.getWriter输出的区别

- response：经常用于设置返回给客户端的内容（输出）

- out：也是给用户做输出使用

  - 当两个一起使用是会使用response进行输出，out追加到response后
  - 可以使用out.flush()进行提前追加
  - 一般情况下使用out输出，因为底层源码都是使用out进行输出，可以避免输出顺序打乱

- out的两个输出方法

  - out.write()：适合输出字符串，当输出整型是会被输出对应ASCII的字符
  - out.print()：适合所有数据

  ==使用out.print()就行==

#### jsp常用的标签

静态包含

- 特点：1、静态包含不会翻译被包含的jsp页面；2、静态包含其实是把被包含的jsp页面的代码拷贝到包含的位置执行输出
- 格式：\<%@ include file="路径" %\>

动态包含

- 特点：1、动态包含会把包含的jsp页面翻译成java代码；2、动态包含底层代码使用如下代码去调用被包含的jsp页面执行输出JspRuntimeLibrary（requst，response，"被包含的jsp路径"，out，false）；3、动态包含，还可以传递参数
- 格式：

```jsp
<jsp:include page="被包含的jsp路径">
    <jsp:param name="参数名1" value="参数值1"/>
    <jsp:param name="参数名2" value="参数值2"/>
</jsp:include>
```

标签转发

- 特点：就是请求转发
- 格式：

```jsp
<jsp:forward page="被包含的jsp路径"></jsp:forward>
```

#### Listener监听器

- Listener监听器是JavaWeb的三大组件之一。JavaWeb的三大组件分别是：Servlet程序、Filter过滤器 、Listener监听器
- Listener它是JavaEE的规范，就是接口
- 监听器的作用是，监听某种事物的变化。通过回调函数，反馈给客户（程序）去做一些响应的处理。

##### ServletContextListener监听器

- ServletContextListener可以监听ServletContext对象的创建和销毁
- ServletContext对象在web工程启动的时候创建，在web工程停止的时候销毁。

使用步骤：

1. 编写一个类实现ServletContextListener

   1. ```java
      public class SerletContextListener implements ServletContextListener {
          @Override
          public void contextInitialized(ServletContextEvent servletContextEvent) {
              System.out.println("ServletContext对象被创建");
          }
          @Override
          public void contextDestroyed(ServletContextEvent servletContextEvent) {
              System.out.println("ServletContext对象被销毁");
          }
      }
      ```

2. 实现其两个回调方法

3. 到web.xml中配置监听器

   1. ```xml
      <!--与servlet标签同级-->
      <listener>
              <listener-class>com.SerletContextListener</listener-class>
          </listener>
      ```

      

## jsp中的EL表达式

- 全称：Expression Language。为表达式语言。
- 作用：代替jsp中的表达式脚本在jsp页面中进行数据输出。因EL表达式在输出数据的时候，比jsp的表达式脚本简洁。
- 格式：\${ 表达式 }
- 当表达式在输出null值的时候，输出的是空串。jsp表达式脚本输出null的时候，输出的是null字符串。

### EL表达式搜索域数据的顺序

- EL表达式主要是在jsp中输出数据‘
- 主要是输出域对象中的数据
- 当四个域中都有相同的key的数据的时候，EL表达式会按照四个域的从小到大的顺序进行搜索，找到就输出。
  - pageContext\==》rerquest\==》session\==》application

```jsp
//域对象的设置
<%    pageContext.setAttribute("key","pageContext");
    request.setAttribute("key","request");
    session.setAttribute("key","session");
    application.setAttribute("key","application");
%>
//域对象的输出，从小到大搜索，搜到即输出。只需要填写域对象的key即可。
${key}
```

### EL表达式输出Bean的普通属性，数组属性。List集合属性，map属性

需求：输出Person类中普通属性，数组属性。List集合属性和map集合属性。

Person（bean类）

```java
public class Person {
    private String name;
    private String[] phones;
    private List<String> cities;
    private Map<String,Object> map;
    private int age=18;
    
    public Person() {}
    public Person(String name, String[] phones, List<String> cities, Map<String, Object> map) {
        this.name = name;this.phones = phones;this.cities = cities;this.map = map;
    }
    public int getage() {return age;}
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    public String[] getPhones() {return phones;}
    public void setPhones(String[] phones) {this.phones = phones;}
    public List<String> getCities() {return cities;}
    public void setCities(List<String> cities) {this.cities = cities;}
    public Map<String, Object> getMap() {return map;}
    public void setMap(Map<String, Object> map) {this.map = map;}
    @Override
    public String toString() {return "Person{" +"name='" + name + '\'' +", phones=" + 
        Arrays.toString(phones) +", cities=" + cities +", map=" + map +'}';
	}
}
```

jap中：

```jsp
<%
    //用于往Javabean添加数据
    Person person = new Person();
    person.setName("阿萨的");
    person.setPhones(new String[]{"11111111111","222222222","333333333333","43434343434343434"});
    List<String> cities = new ArrayList<>();
    cities.add("北京");
    cities.add("上海");
    cities.add("深圳");
    person.setCities(cities);
    Map<String,Object> map = new HashMap<>();
    map.put("key1","value1");
    map.put("key2","value2");
    map.put("key3","value3");
    map.put("key4","value4");
    person.setMap(map);
	//将数据添加到域中
    pageContext.setAttribute("p",person);
%>
<!--输出格式与js调用json类似，使用键名点上对应的内部数据键名，如果为数组集合格式则使用下标。-->
<!--点的作用实质就是调用bean类中的get方法-->
<br>输出person:${p}
<br>输出person的name属性:${p.name}
<br>输出person的phones数组属性值:${p.phones}
<br>输出person的cities集合中的元素值:${p.cities}
<br>输出person的List集合中个别元素值:${p.cities[1]}
<br>输出person的Map集合:${p.map}
<br>输出person的Map集合中某个key的值:${p.map[2]}
<br>输出person的age属性:${p.age}
<br>
```

### EL表达式的运算

- 可直接在格式内进行对数值的关系运算，算术运算，逻辑运算，empty运算（判断是否为null）

格式：${ 运算表达式 }

|   关系运算符   |   说明   |                      示例                      | 结果  |
| :------------: | :------: | :--------------------------------------------: | :---: |
|     ==或eq     |   等于   |            ${ 5 == 5 }或${ 5 eq 5 }            | true  |
|     !=或ne     |  不等于  |            ${ 5 != 5 }或${ 5 ne 5 }            | false |
|     <或lt      |   小于   |            ${ 3 < 5 }或${ 3 lt 5 }             | true  |
|     >或gt      |   大于   |            ${ 3 > 5 }或${ 3 gt 5 }             | false |
|     <=或le     | 小于等于 |            ${ 3 <= 5 }或${ 3 le 5 }            | true  |
|     >=或ge     | 大于等于 |            ${ 3 >= 5 }或${ 3 ge 5 }            | false |
| **逻辑运算符** |          |                                                |       |
|    &&或and     |  与运算  | ${ 12\==12 && 12<11 }或${ 12\==12 and 12<11 }  | false |
|    \|\|或or    |  或运算  | ${ 12\==12 \|\| 12<11 }或${ 12\==12 or 12<11 } | true  |
|     !或not     | 取反运算 |           ${ !true }或${ not true }            | false |
| **算数运算符** |          |                                                |       |
|       +        |   加法   |                   ${ 1 + 1 }                   |   2   |
|       -        |   减法   |                   ${ 2 - 1 }                   |   1   |
|       *        |   乘法   |                   ${ 3 * 3 }                   |   9   |
|     /或div     |   除法   |            ${ 9 / 3 }或${ 9 div 3 }            |   3   |
|     %或mod     |   取模   |          ${ 18 % 12 }或${ 18 mod 12}           |   6   |

#### empty运算

此运算可以判断一个数据是否为空，如果为空输出true，不为空输出false。

以下几种情况为空：

1. 值为null的时候
2. 值为空串的时候
3. 值是Object类型数组，长度为零的时候
4. list集合，元素个数为零
5. map集合，元素个数为零

#### 三元运算

&{表达式1 ? 表达式2 : 表达式3}

如果表达式1的值为真，返回表达式2的值，如果表达式1的值为假，返回表达式3的值。

#### “.”点运算和[]中括号运算符

- .点运算 ，可以输出Bean对象中某个属性的值

- []中括号运算，可以输出有序集合中某个元素的值，并且[]中括号运算，还可以输出map集合中key含有特殊字符的key的值

  示例：

  ```jsp
  <%
      Map<String, Object> mapp = new HashMap<>();
      mapp.put("a.a.a","aaaValue");
      mapp.put("a-a-a","aaaValue");
      mapp.put("a+a+a","aaaValue");
      request.setAttribute("map",map);
  %>
  ${map["a.a.a"]}
  ${map['a-a-a']}
  ${map["a+a+a"]}
  ```

### EL表达式的11个隐含对象

| 变量             | 类型                   | 作用                                                 |
| ---------------- | ---------------------- | ---------------------------------------------------- |
| pageContext      | PageContextmpl         | 可以获取jsp中的九大内置对象                          |
| pageScope        | Map\<String,Object\>   | 可以获取pageContext域中的数据                        |
| requestScope     | Map\<String,Object\>   | 可以获取Request域中的数据                            |
| sessionScope     | Map\<String,Object\>   | 可以获取Session域中的数据                            |
| applicationScope | Map\<String,Object\>   | 可以获取ServletContext域中的数据                     |
| param            | Map\<String,String\>   | 可以获取请求参数的值                                 |
| paramValues      | Map\<String,String[]\> | 可以获取请求参数的值，获取多个值的时候使用           |
| cookie           | Map\<String,Cookie\>   | 可以获取当前请求的Cookie信息                         |
| initParam        | Map\<String,String\>   | 可以获取在web.xml中配置的\<context-param\>上下文参数 |

当四个域的键名一样时，EL表达式获取四个域的值：

```jsp
<%
    //  对四个域进行设置值
    pageContext.setAttribute("key1","pageContext");
    request.setAttribute("key1","request");
    session.setAttribute("key1","session");
    application.setAttribute("key1","applocaton");
%>
<%//使用EL表达式获取值%>
${pageScope.key1}
${requestScope.key1}
${sessionScope.key1}
${applicationScope.key1}
```

在EL表达式中pageContext对象的使用：

```jsp
1.协议：${pageContext.request.scheme}<br>
2.服务器ip：${pageContext.request.serverName}<br>
3.服务器端口：${pageContext.request.serverPort}<br>
4.获取工程路径：${pageContext.request.contextPath}<br>
5.获取请求方法：${pageContext.request.method}<br>
6.获取客户端ip地址：${pageContext.request.remoteHost}<br>
7.获取会话id编号：${pageContext.session.id}<br>
<%//对pageContext.request封装成req%>
<% pageContext.setAttribute("req",request); %><br>
1.协议：${req.scheme}<br>
2.服务器ip：${req.serverName}<br>
3.服务器端口：${req.serverPort}<br>
4.获取工程路径：${req.contextPath}<br>
5.获取请求方法：${req.method}<br>
6.获取客户端ip地址：${req.remoteHost}<br>
7.获取会话id编号：${pageContext.session.id}<br>
```

其他对象的使用：

```jsp
<!--请求地址?username=阿斯顿&password=1212312&hobby=ooo&hobby=ppp-->
输出请求参数username的值：${param.username}<br>
输出请求参数password的值：${param.password}<br>
输出请求参数username的值：${paramValues.username[0]}<br>
输出请求参数hobby的值：${paramValues.hobby[0]}<br>
输出请求参数hobby的值：${paramValues.hobby[1]}<br>
<!------------------------------------------------>
输出请求头【User-Agent】的值：${header['User-Agent']}<br>
输出请求头【Connection】的值：${header.Connection}<br>
输出请求头【User-Agent】的值：${headerValues['User-Agent'][0]}<br>
<!------------------------------------------------>
获取cookie的名称：${cookie.JSESSIONID.name}<br>
获取cookie的值：${cookie.JSESSIONID.value}
<!------------------------------------------------>
输出&lt;Context&gt;username的值：${initParam.username}<br>
输出&lt;Context&gt;url的值：${initParam.url}
```

web.xml中的设置：

```xml
    <context-param>
        <param-name>username</param-name>
        <param-value>root</param-value>
    </context-param>
    <context-param>
        <param-name>url</param-name>
        <param-value>www.xxx.com</param-value>
    </context-param>
```

## JSTL标签库

- 导入两个包：分别为jstl.jar与standard.jar
- 需要在jsp中引入<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
- 为一个标签库

```jsp

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
以上为需要导入的语句
----------------------------------------------------------------------------
以下为用法各种标签的用法
	<c:forEach>：遍历循环，begin为初始值，end为最大值，var为输出的变量，items为需要遍历的变量
    <c:forEach begin="0" end="${all.size()}" var="a" items="${all}">
        <p>${a}</p>
    </c:forEach>

```

## 文件的上传和下载



## Filter过滤器

- 三大组件之一。
- 作用：拦截请求，过滤响应。
- 应用：1、权限检查；2、日记操作；3、事物管理等。

示例：

```java
public class Filter01 implements Filter {
    public void destroy() {
    }

    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        //放行方法
        chain.doFilter(req, resp);
    }

    public void init(FilterConfig config) throws ServletException {

    }

}
```

web.xml配置：

```xml
<filter>
    <filter-name>Filter01</filter-name>
    <filter-class>com.filter.Filter01</filter-class>
</filter>
<filter-mapping>
    <filter-name>Filter01</filter-name>
    <!--过滤的路径；当前路径映射到当前工程的web目录下-->
    <!--在这目录下的所有资源都必须经过Filter过滤器的放行才能访问-->
    <url-pattern>/admin/*</url-pattern>
</filter-mapping>
```



### 生命周期

- 生命周期的方法：
  1. 构造器方法
  2. init初始化方法
     - web工程启动时第一、二步执行（Filter已经创建）
  3. doFilter方法
     - 每次拦截到请求都会执行
  4. destroy销毁
     - 停止web工程时执行（停止web工程销毁Filter过滤器）

```java
//顺序如下：
    public Filter01() {
        System.out.println("Filter01构造器方法执行");
    }
	public void init(FilterConfig config) throws ServletException {
        System.out.println("Filter01初始化方法执行");
    }
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        System.out.println("Filter01请求拦截方法执行");
		//放行方法
        chain.doFilter(req,resp);
    }
 	public void destroy() {
        System.out.println("Filter01销毁方法执行");
    }
```

### FliterConfig类

- Tomcat每次创建Filter的时候，也会同时创建一个FilterConfig类，其中包含了Filter配置文件的配置信息
- FilterConfig类的作用是获取filter过滤器的配置内容
  1. 获取Filter的名称filter-name的内容
  2. 获取Filter中配置的init-param初始化参数
  3. 获取ServletContext对象

```java
    public void init(FilterConfig config) throws ServletException {
        System.out.println("Filter01初始化方法执行");
//        获取过滤器名称
        System.out.println("filter-name的值时："+config.getFilterName());
//        获取web.xml中的init-param初始化参数值
        System.out.println("初始化参数ddddd的值："+config.getInitParameter("dddddd"));
        System.out.println("初始化参数panme的值："+config.getInitParameter("pname"));
//        获取ServletContext对象
        System.out.println("ServletContext对象："+config.getServletContext());
    }
```

### FilterChain过滤链

- 多个过滤器一起工作
- 在多个Filter过滤器执行的时候，执行的优先顺序是由他们在web.xml中从上到下配置的顺序决定的
- 多个过滤器执行的特点：
  - 所有过滤器和目标资源默认都执行在同一线程中
  - 多个过滤器共同执行的时候，他们使用同一个Request对象。
- FilterChain.doFilter()方法的作用：
  - 执行下一个Filter过滤器（如果有Filter）
  - 执行目标资源（没有Filter）

### Filter的拦截路径

精确匹配

```xml
<url-pattern>/admin/文件名.后缀</url-pattern>
```

目录匹配

```xml
<url-pattern>/admin/*</url-pattern>
```

后缀名匹配

```xml
<url-pattern>*.后缀名</url-pattern>
```

## JSON

- 是一种轻量级的数据交换格式
- 轻量级指的是与xml作比较
- 数据交换指的是客户端和服务器之间业务数据的传递格式

### JSON在JavaScript中的使用

- json本身是一个对象

- json中的key我们可以理解为是对象中的一个属性

- json中的key访问就跟访问对象的属性一样：json对象.key（指在JavaScript中）

- JavaScript中转换json格式等方法有两种

  - 一：对象的形式存在，叫json对象

  - 二：字符串的形式存在，叫json字符串

  - 一般操作json中的数据的时候，需要json对象的格式

  - 一般在客户端和服务器之间进行数据交换的时候，使用json字符串

    JSON.stringify(json对象)	把json对象转换成为json字符串

    JSON.parse(json字符串)		把json字符串转换成为json对象

### JSON在Java中的使用

- 使用谷歌开发的Gson将JSON数据在java中转换
- 主要是转化数据

javaBean和json的互转

JavaBean：

```java
public class JsonBean {
    private  Integer id;
    private String name;
    public JsonBean() {}
    public JsonBean(Integer id, String name) {this.id = id;	this.name = name;}
    public Integer getId() {return id;}
    public void setId(Integer id) {this.id = id;}
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    @Override
    public String toString() {
        return "JsonBean{" +"id=" + id +", name='" + name + '\'' +'}';
    }
}
```

java测试方法：

```java
//    json与JavaBean的互转
    @Test
    public void jsonTest01(){
        JsonBean jsonBean = new JsonBean(1, "阿斯顿");
        Gson gson = new Gson();
//        javaBean转成json字符串
//        toJson()可以将java对象转成json字符串
        String s = gson.toJson(jsonBean);
        System.out.println(s);
//        fromJson()可以将json字符串转成Java对象
//        第一个参数为：json字符串
//        第二个参数为：转换回去的JavaJava对象类型
        JsonBean jsonBean1 = gson.fromJson(s, JsonBean.class);
        System.out.println(jsonBean1);
    }
```

List和json的互转

```java
//    list和json的互转
    @Test
    public void jsonTest02(){
        List<JsonBean> list = new ArrayList<>();
        list.add(new JsonBean(1,"第一"));
        list.add(new JsonBean(2,"第二"));
        Gson gson = new Gson();
//        把list转化成json字符串
        String s = gson.toJson(list);
        System.out.println(s);


//        把json字符串转换成List集合
//		 第二个参数为Gson的TypeToken类【不推荐】
        List<JsonBean> list1 = gson.fromJson(s, new JsonListType().getType());
        System.out.println(list1);
        JsonBean jsonBean = list1.get(1);
        System.out.println(jsonBean);
    }
```

继承了Gson的TypeToken类的类

```java
//只需要继承，并将泛型填写成需要转换的类型
public class JsonListType extends TypeToken<List<JsonBean>>{}
```

map和json的互转

```Java
    @Test
    public void jsonTest03(){
        Map<Integer, JsonBean> map = new HashMap<>();
        map.put(1,new JsonBean(1,"第一"));
        map.put(2,new JsonBean(2,"第二"));

        Gson gson = new Gson();
//        将map转化成json字符串
        String s = gson.toJson(map);
        System.out.println(s);
//        使用了匿名内部类的方法new TypeToken<Map<Integer, JsonBean>>() {}【推荐】
//		 TypeToken内的泛型填写需要转换的类型即可
        Map<Integer, JsonBean> o = gson.fromJson(s, new TypeToken<Map<Integer, JsonBean>>() {}.getType());
        System.out.println(o);
    }

}
```

## AJAX请求

- ajax使用一种浏览器通过异步发起请求，局部更新页面的技术。
- ajax请求的局部更新，浏览器地址栏不会发生变化
- 局部更新不会舍弃原来页面的内容

以下请求所用到的通用servlet接口【此请求可以返回JSON数据】：

```java
public class AjaxServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("ajax:post");
        request.setCharacterEncoding("UTF-8");
//        此处请求头不设置可能会有跨域问题
        response.setHeader("Access-Control-Allow-Origin","*");
        System.out.println(request.getParameter("action"));
        JsonBean asd = new JsonBean(1, "asd");
        Gson gson = new Gson();
        String s = gson.toJson(asd);
        response.getWriter().write(s);
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("ajax:get");
        request.setCharacterEncoding("UTF-8");
//        此处请求头不设置可能会有跨域问题
        response.setHeader("Access-Control-Allow-Origin","*"); 
        System.out.println(request.getParameter("action"));
        JsonBean asd = new JsonBean(1, "asd");
        Gson gson = new Gson();
        String s = gson.toJson(asd);
        response.getWriter().write(s);
    }
}
```

以下请求所用到的通用JavaBean类：

```java
public class JsonBean {
    private  Integer id;
    private String name;
    public JsonBean() {}
    public JsonBean(Integer id, String name) {
        this.id = id;
        this.name = name;
    }
    public Integer getId() {return id;}
    public void setId(Integer id) {this.id = id;}
    public String getName() {return name;}
    public void setName(String name) {this.name = name;}
    @Override
    public String toString() {
        return "JsonBean{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

### 原生Ajax请求

html部分：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <button onclick="q()">Ajax GET请求</button>
    <!-- <button onclick="q()">Ajax POST请求</button> -->
</body>
<script>
    function q() {      
        // 1、首先创建XMLHttpRequest对象
        var xmlHttpRequset=new XMLHttpRequest();        
        // 2、调用open方法设置请求参数
        xmlHttpRequset.open("GET","http://localhost:8080/JavaWeb01/AjaxServlet",true);
        // 4、在send方法前绑定onreadystatechange事件，处理请求完成后的操作
        xmlHttpRequset.onreadystatechange=function(){
            // 判断响应状态和响应码
            if (xmlHttpRequset.readyState == 4 && xmlHttpRequset.status == 200) {
                console.log(JSON.parse(xmlHttpRequset.responseText));
            }
        }
        // 3、调用send方法发送请求
        xmlHttpRequset.send();        
    }
</script>
</html>
```

### jQuery中的Ajax请求

- $.ajax方法
  - url：表示请求的地址
  - type：表示请求的数据类型GET或POST请求
  - data：表示发送给服务器的数据
    - 数据的格式分为两种：一：name=value&name=value；二：{key:value}
  - success：请求成功，响应的回调函数
  - dataType：响应的数据类型
    - 常用的响应数据类型有text（纯文本）、xml（xml数据）、json（json对象）
- $.get方法和$.post方法
  - url：表示请求的地址
  - data：表示发送给服务器的数据
  - callback：成功的回调函数
  - type：响应的数据类型
- $.getJSON方法
  - url：表示请求的地址
  - data：表示发送给服务器的数据
  - callback：成功的回调函数
- 表单提交参数序列化serialize()
  - serialize()可以把表单中所有表单的内容都获取到，并以name=value&name=value的形式进行拼接

以下为所有方法的示例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="./jquery-1.7.2.js"></script>
</head>
<body>
    <button id="ajaxBtn">$.ajax请求</button>
    <button id="getBtn">$.get请求</button>
    <button id="postBtn">$.post请求</button>
    <button id="getJsonBtn">$.getJSON请求</button><br>
    <form id="f">
        用户名：<input type="text" name="username"><br>
        密码：<input type="text" name="password"><br>
        下拉单选：<select name="xldanx" id="">
            <option value="asd">asd1</option>
            <option value="asd">asd2</option>
            <option value="asd">asd3</option>
        </select><br>
        下拉多选：<select name="xlduox" id="" multiple="multiple">
            <option value="zxc1">zxc1</option>
            <option value="zxc2">zxc2</option>
            <option value="zxc3">zxc3</option>
        </select><br>
        复选：<input type="checkbox" name="check" value="check1">check1 <input checked type="checkbox" name="check" value="check2">check2 <br>
        单选：<input checked type="radio" name="radio" value="radio1">radio1 <input type="radio" name="radio" value="radio2">radio2 <br>
        <input type="button" id="submit" value="提交---serialize()"/>
    </form>  
</body>
    <script>
        $("#ajaxBtn").click(function(){
            console.log(1)
            $.ajax({
                // 请求的url地址
                url:"http://localhost:8080/JavaWeb01/AjaxServlet",
                // 发送到服务器的数据
                data:"action=jQueryAjax",
                // 请求方式
                type:"GET",
                // 得到的返回数据（data为返回的数据）
                success:function(data){
                    // console.log(typeof(data))
                    console.log(data)
                },
                // 返回的数据转换成的类型
                dataType:"json"
            });
        });
        $("#getBtn").click(function(){
            console.log(2);
            // GET方法请求的参数分别为url（请求的url地址）、datta（发送的数据）、callback（成功返回的回调函数）、type（返回的数据类型）
            $.get("http://localhost:8080/JavaWeb01/AjaxServlet","action=jQueryGET",function(data){
                console.log(data)
            },"json");
        });
        $("#postBtn").click(function(){
            console.log(3);
            // POST方法请求的参数分别为url（请求的url地址）、datta（发送的数据）、callback（成功返回的回调函数）、type（返回的数据类型）
            $.post("http://localhost:8080/JavaWeb01/AjaxServlet","action=jQueryPOST",function(data){
                console.log(data)
            },"json");
        }); 
        $("#getJsonBtn").click(function(){
            console.log(4);
            // getJSON方法请求的参数分别为url（请求的url地址）、datta（发送的数据）、callback（成功返回的回调函数）
            // 返回的数据直接转换成json
            $.getJSON("http://localhost:8080/JavaWeb01/AjaxServlet","action=jQueryGETjson",function(data){
                console.log(data)
            })
        });
       $("#submit").click(function(){
            console.log(5);
            // $("#f").serialize()：用于序列化表单所提交的参数，使其拼接只提交的参数中或者直接使用
            $.getJSON("http://localhost:8080/JavaWeb01/AjaxServlet","action=jserialize()&"+$("#f").serialize(),function(data){
                console.log(data)
                console.log($("#f").serialize())
            })
       });
    </script>
</html>
```





















































































































































































































































































































































