# SpringMVC

> 基于Spring的框架,实际上是Spring的一个模块 ,  是控制器的容器    

MVC拥有一个servlet叫**DispatherServlet**  (中央调度器) ,这个调度器将请求转发给Controller对象(利用@Controller注解创建的对象)

## 使用

### 配置web.xml文件

```xml
	<!--在DispatcherServlet的init()方法中会执行读取web.xml配置文件的操作-->
    <!--并且会把获得的容器对象放置到ServletContext域中供以后使用-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!--springmvc创建对象的时候默认读取的文件是/WEB-INF/<servlet名字>-servlet.xml-->
        <!--可以自定义读取文件的位置-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!--classpath代表类路径,应该为资源路径-->
            <param-value>classpath:spring.xml</param-value>
        </init-param>

        <!--表示在服务器启动后自动创建对象, 期中的数字代表创建对象的顺序, 数字越小越优先, 为大于等于0的整数-->
        <!--默认的话是不会加载就创建,而是等用户发起请求才创建对象-->
        <load-on-startup>1</load-on-startup>

    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--在框架中可以使用两种方式写url-->
        <!--第一种是以扩展名的方式如, *.do, *.action *.mvc代表他们结尾的请求都又这个servlet处理-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

```

> 步骤
>
> 1. 配置组件扫描器
> 2. 配置中央调度器，需要配置访问spring文件地址，开启自动创建对象，访问url

### springMVC处理请求步骤

1. 发起请求给Tomcat服务器
2. Tomcat根据web.xml文件映射规则将请求发送给中央调度器
3. 中央调度器根据spring配置文件启动组件扫描器创建对象
4. 根据创建对象的注解寻找请求对应的处理方法
5. 将请求转发个控制器的对应方法
6. 框架执行方法，将返回对象进行处理转发到下一个页面

### 处理代码示例

```xml
<!--springmvc.xml配置文件是spring的配置文件用来配置Bean此处开启组件扫描-->    
<context:component-scan base-package="com"/>
```

```java
/**
 * Controller 创建处理器对象, 对象放在的SpringMVC的容器中
 * 使用位置是在类的上面
 * 和之前将的Service, Component一样都是用来创建对象只是对象的作用不同
 */
@Controller
public class Some {
    /**
     * 用来处理some.do的请求,返回值参数很多,名称自定义
     * RequestMapping请求映射注解,将请求和方法绑定,一个请求处理一个方法
     * 属性:
     * 1.value,是一个String表示请求的uri地址,属性唯一,可写多个请求
     *	需要在uri地址的前面添加"/"
     * @return model是给用户的数据, view是给用户的视图 jsp等
     */
    @RequestMapping(value = {"/some.do"})
    public ModelAndView doSome() {
        //创建返回对象
        ModelAndView mv = new ModelAndView();
        //添加数据,在请求的最后框架会讲数据放在request的域中
        mv.addObject("msg", "欢迎使用SpringMVC处理请求");

        //显示页面实际就是请求转发
        mv.setViewName("/show.jsp");
        return mv;
    }
}

```

```jsp
<!--show.jsp页面-->
<html>
<head>
    <title>这里是show.jsp页面</title>
</head>
<body>
<h3>这里是show.jsp</h3><br/>
    <!--El表达式-->
<h3>这个是msg的内容 ${msg}</h3>

</body>
</html>
</%--@elvariable id="msg" type="java.lang.String>
```

## springmvc框架底层原理

1. **tomcat**启动后根据**web.xml**中**DispatcherServlet**对象的**load-on-start**属性创建DispatcerServlet对象
   **DispatcerServlet**对象继承自**HttpServlet**是一个真实的**Servlet**对象
2. 在**DispatcerServlet**对象创建时会进行**init()**方法, 在方法中读取配置文件**stringmvc.xml** , 
   执行组件扫描器
3. 容器创建成功, 后创建注解类(**Controller对象**)，将对象放在springmwv容器中，容器是类似map，类似
   map.put(“controllerName”,Controller对象)
4. 收到请求后服务器判断请求分配给**DispatcherServlet**对象, **DispatcherServlet**对象根据请求的url将请求发送给对应
   **Controller**对象的对应方法
5. **Controller**对象处理完请求返回对应的对象(**ModelAndView**对象)
6. 框架处理返回对象, 将添加的对象添加到**request**域中, 处理请求转发等

## 视图解析器

> springmvc工具可以帮助开发人员设置视图文件的访问路径

### 配置

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<!--前缀属性:视图文件的路径,前后斜杠,前代表根,后代表是路径-->
	<property name="prefix" value="/WEB-INF/view/"/>
	<!--后缀属性:试图文件的扩展名-->
	<property name="suffix" value=".jsp"/>
</bean>
```

配置后使用视图需要这样

```java
mv.setViewName("show");
```



## RequestMapping注解解析

1. 将地址中共用的部分抽取出来,叫做**模块名称**可以放在类的前面统一定义

   ```java
   @Controller
   @RequestMapping("/user")		//统一模块名称对所有方法有效
   public class Some {}
   ```

2. 设置接收请求的请求方式

   ```java
   //method属性设置请求方式
   @RequestMapping(value = {"/some.do"},method = RequestMethod.GET)
   public ModelAndView doSome() {
   	//创建返回对象
   	ModelAndView mv = new ModelAndView();
   	//添加数据,在请求的最后框架会讲数据放在request的域中
   	mv.addObject("msg", "欢迎使用SpringMVC处理请求");
   	//显示页面实际就是请求转发
   	mv.setViewName("show");
   	return mv;
   }
   ```

   

## Controller获得请求参数

> 在处理请求的方法中定义形参参数,mvc会给他们赋值使用

### 常用请参数

  1. **HttpServletRequest request**
  2. **HttpServletResponse response**
  3. **HttpSession session**
  4. 请求中所携带的请求参数
     * 逐个接收方案
     * 对象接收方案

#### 逐个接收方案

> 在处理方法的形参中写入与请求参数同名的参数就可以直接使用,参数名必须一致

```java
public ModelAndView doSome(String name, String password) {
```

类型需要一一对应,转换不合法不会报错会400,然后在spring日志中会有一个记录

post方式的请求会造成中文乱码, get方式不会

##### 在过滤器中设置字符编码问题

过滤器可以使用框架定义的,也可以自己定义

###### 配置字符集过滤器对象

```xml
<filter>
    <!--配置字符集过滤器-->
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    
    <!--设置字符集编码-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
   
    <!--下面将请求会回应的字符强制使用设置为ture-->
    <init-param>
        <param-name>forceRequestEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>

<!--下面将过滤器设置为对所有访问有效-->
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



#### 对象接收方案

