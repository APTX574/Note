# spring5

## IOC容器

### 概念

> 利用 xml/注解+反射+工厂设计模式，利用spring管理创建java对象



### IOC对bean的管理的内容

1. 利用IOC创建javaBean对象
2. 利用IOC进行javaBean属性的注入
   * 构造器注入
   * set方法注入

---



### IOC对bean管理的方式

####  基于xml文件

##### 创建方式

  ```java
  ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
  User user = context.getBean("User", User.class);
  ```



  ```xml
  <!--bean1.xml-->
  <beans >
    	<bean 
            id="user" 			<!--对象的唯一表示, pring去区分对象, 还有name属性差不多,现在不用了-->
            class="com.example.spring.User"		<!--所要创建的对象的类的全路径有包路径-->
            />
  </beans>
  ```



在其他类中只要写这个就会自动注入

```java
@AutoWired
private User user;
```



  > 使用xml，创建对象时默认调用对象的无参构造方法，必须要有无参构造方法
  >
  > 在注入时有参构造方法会与注入冲突不能有有参构造函数
  >
  > xml与注解只能二选一

---



##### 属性注入

###### xml注入属性

> DI 依赖注入就是注入属性，是！IOC的一种具体实现，在创建对象的基础完成

  * 使用set方法注入

    1. 先创建带有参构造的对象

    2. 利用spring配置创建对象

    3. 利用property标签进行参数注入

	   ```xml
	       <bean id="user" class="com.example.spring.User">
	           <property name="name" value="tom"/>		<!--底层应该是使用反射去调用set方法-->
	       </bean>
	   ```
	
  * 使用有参构造注入(对象可以没有无参构造)

	1. 先创建带有参构造的对象

    2. 利用spring配置创建对象

    3. 利用property标签进行参数注入
    
       ```xml
           <bean id="user" class="com.example.spring.User">
               <constructor-arg name="name" value="tom"/>
           </bean>
       ```

++++



###### xml注入空值

  ```xml
      <bean id="user" class="com.example.spring.User">
          <property name="name">
              <null/>				<!--为name设置空值-->
          </property>
      </bean>
  ```

###### xml注入特殊字符

  ```xml
  &lt;&gt;	<!--转义'<'与'>'-->
  <![CDATA[	使用cdata标签
      
      ]]>
  
  ```
###### 使用内外部bean注入

* 外部bean

  ```xml
  <!--xml配置文件-->
  <beans>
      
      <bean id="userService" class="service.UserService">  <!--创建一个service-->
          <property name="userDao" ref="userDao"/>		<!--将dao注入到他的属性中-->
      </bean>												<!--外部注入使用ref属性-->
      <bean id="userDao" class="Dao.UserDaoImpl"/>		<!--创建上面注入所用到的dao对象-->
      
  </beans>
  ```

  ```java
  //Userservice类的内部结构
  
  public class UserService {
      private UserDao userDao;	//需要外部注入对象的引用
      
      public void setUserDao(UserDao userDao) {	//设置外部注入的set方法
          this.userDao = userDao;
      }
  
      public void userUpdate() {
          userDao.update();
  
      }
  }
  
  ```

  

* 内部bean和级联赋值

  > 一个部门对应多员工,一个部门对象对应多个员工对象

  ```java
  public class Employee {
      private String name;
      private Department department;
      //getset方法
  }
  
  public class Department {
      String name;
  }
  ```

  ```xml
  	<!--内部类在属性中嵌套一个类--->
  	<!--或者也可以ref另一个创建好的类-->
  	<bean id="employee" class="bean.Employee">
          <property name="name" value="tom"/>
          <property name="department">
              <bean class="bean.Department">
                  <property name="name" value="保安"/>
              </bean>
          </property>
      </bean>
  ```

  > 内部bean与级联赋值还可以使用外部bean的方式

  ___

  ###### xml注入数组集合

  

  ```xml
  	<!--注入数组-->
  	<property name="arr">
  		<array>
  			<value type="int">10</value>
  			<value type="int">23</value>
  		</array>
  	<property>
          
      <!--注入map-->   
  	<property name="map">
  		<map>
  			<entry key="key1" value="niq"/>
  			<entry key="key2" value="ni2q"/>
  		</map>
  	</property>
          
      <!--注入list-->    
  	<property name="list">
  		<list>
  			<value type="java.lang.String">1ad</value>
  			<value type="java.lang.String">1aqw</value>
  		</list>
  	</property>
         
  ```

  > 可以使用util空间将集合提取,将提取出来的集合注入

  ---

  

#### 基于注释

##### 创建方式

1. 导入aop包

2. 开启xml自动扫描

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"	<!--引入空间-->
          xsi:schemaLocation="http://www.springframework.org/schema/beans 													http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/context																	http://www.springframework.org/schema/context/spring-context.xsd">
       <!--开启自动扫描,扫描文件包为com下-->
       <context:component-scan base-package="com" use-default-filters="false">
           <!--关闭默认的过滤器使用下面我们给出的过滤器-->
           <context:include-filter type="annotation" expression="注解类型"/>
           <!--包括或忽略的过滤器, 按类或注解等筛选,表达式是-->
           <!--也可以是使用默认的过滤器, 但是可以忽略掉满足条件的类-->
   	</context:component-scan> 			
   	
   </beans>
   ```



##### 基于注解方式实现注入

1. **@AutoWired**根据类型自动注入
   * 直接在要注入的声明上添加注解,会自动根据类型注入
2. **@Qualifier**根据属性名称进行注入
   * 和AutoWired一起使用
   * 一个接口多个实现类,根据名称指定对象    
3. **@Resource**可以根据类型  也可以根据名称
   * 可以使用上面两个注 解(不建议使用)
4. **@Value**注入普通类型属性 
   * 给一些普通的类型直接用value属性注入值

```
//相当于xml中的bean标签
@Repository
public class UserDaoImpl implements UserDao {
	//表示会自动寻找合适得类给jdbctl赋值
    @Autowired
    private JdbcTemplate jdbcTemplate;

}
```



> 都不需要set方法,已经封装了.



 ##### 完全注解开发

1. 创建一个配置类,代替xml配置文件 ,  要添加一个**@Configuration**注解标志注解类

2. 添加注解**@ComponentScan(basePackages={包路径})**代替开启自动扫描的配置

   ```java
   @Configuration
   @ComponentScan(basePackages = {"com"})
   public class Config {
   }
   ```

   

3.   加载配置类

     ```javascript
     ApplicationContext context = new 												AnnotationConfigApplicationContext(Config.class);
     	//Config.calss为配置类的字节码
     ```

     

### IOC对bean管理中的FactoryBean

> spring中有两种bean对象,一种是普通的我们创建的bean
>
> 另一种是FectoryBean 工厂Bean

#### 普通Bean

**在配置文件中定义类型与返回类型一致(class属性和返回类型一致)**

#### 工厂Bean

**spring内置的Bean类型**

1. 定义一个类,让其实现FactoryBean接口

2. 实现接口的抽象方法,在抽象方法中设置返回的类型
   * getObject--->获取对象示例
   * getObjectType--->获取对象类型
   * isSingle--->对象是否为单例



### Bean的一些属性

#### Bean的作用域

> 设置创建的Bean是单示例还是多实例

​	**单实例**通过context.getBean()方法获得的bean都是一个对象,  只创建一次

​	**多示例**每次getBean都会创建一个对象

***通过scope属性设置***

+ **singleton**为单实例	默认状态	配置文件一加载就会创建对象
+ **prototype**为多实例   原型模式   每次调用函数时创建实例
+ **request**会将每次的对象存放在请求域中
+ **session**会将每次的对方存放在会话域中

#### Bean的生命周期

##### 生命周期

1. 使用无参构造方法创建对象
2. 利用set方法进行属性注入DI
3. 执行初始化方法 **init-method**属性指定
4. 正常使用
5. 容器关闭指向销毁方法 **destroy-method**属性指定
6. **context.close**销毁 ,  接口强转到实现类进行方法调用

##### Bean的后置处理器

1. 在初始化之前将Bean的示例传递给Before方法
2. 在初始化之后将Bean的示例传递给After方法

##### 创建Bean的后置处理器

1. 创建类实现**BeanPostProcessor**接口

2. 实现**After**和**Before**两个方法
3. 在配置文件中配置后置处理器





## AOP面向切面开发

### 概念

> 面向切面(方面)编程,将业务逻辑之间的耦合度 
> 不通过修改源代码的方式 ,  在主干中添加新的功能

### 原理

**底层使用动态代理模式**

1. 有接口时增强一个类的方法 ,  **JDK**动态代理

   创建接口实现类的代理对象 ,  通过代理对象去增强实现类的方法

2. 没有接口时增强一个类的方法 ,  **CGLIB**动态代理
   ~~利用当前类的子类去增强高耦合~~
   
   创建当前类的**子类**的代理对象

#### AOP操作术语

1. **连接点**
   + 类中**理论上能被增强的方法**
2. **切入点**
   + 类中实际**真正被增强的方法**(有一些方法能被增强但是并未被增强)
3. **通知(增强)**
   + **实际被增强**的部分**逻辑**(增加逻辑判断)
     1. 前置通知
     2. 后置通知
     3. 环绕通知
     4. 异常通知(当方法出现异常会执行)
     5. 最终通知(finally不管有没有异常都会执行)
4. **切面**  
   + 把通知应用到切入点的过程(是一个**动作**)

#### 底层实现代码

使用类Proxy创建代理对象(**JDK**动态代理)

```java
//代理模式的使用main方法
public class JDKProxy {
    public static void main(String[] args) {
        
        //将被代理对象所实现的接口封着成Class的数组作为newProxyInstance 的参数
        Class[] clazz = {UserDao.class};
        
        //创建被代理的对象
        UserDao userDao = new UserDaoImpl();
        
        //创建代理模式Handler的具体实现并传入被代理对象,在Handler中实现方法的增强
        InvocationHandler invocationHandler = new Handler(userDao);
        
        //创建代理对象,代理对象是一个被代理对象实现接口的实现类
        //传入类的加载器,实现接口字节码数组,增强方法Handler的实现类
        UserDao dao = (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), 
                                                       clazz, invocationHandler);
        
        //调用代理类的方法,这个方法就是被增强后的方法,返回值为Handler中involve方法中的返回值
        int update = dao.update(12, 24);
        
        System.out.println("这个是被代理对象"+userDao);
        System.out.println("这个是代理对象" + dao);
    }
}
```


```java
//代理模式Handler的具体实现
public class Handler implements InvocationHandler {
    private Object object;
    //私有的对象,是被代理的对象.需要一个带参构造函数
    //重写的involve方法,在之中增强方法
    //proxy为真实的代理对象,method为被代理的方法, args为被代理方法的参数
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //通过merhod.getName()获得被代理的方法名, 进而进行区分代理
        if (method.getName().equals(toString())) {
            return method.invoke(object, args);
        }
        
        //这个东西不能toString不知道为啥
        ~~System.out.println("输出invoke方法参数proxy"+proxy);~~
            
        //调用原来的被代理对象,可以在这行代码的前后添加增强的方法
        Object invoke = method.invoke(object, args);
        
        //对于一个代理对象, 其中的被代理对象也就是这个object都是唯一的
        System.out.println("这个是Handler里面的object对象" + object);
		
        //返回值,可以在原先的返回值上进行增强
        return invoke;
    }
}

```

#### AOP操作(准备)

>  Spring一般基于**AspectJ**实现AOP

+  AspectJ是一个单独框架 , 但是一般一起使用

##### 基于注解

###### 切入点表达式

> 知道对哪个类的哪个方法进行增强

**execution**( [**权限修饰符**] [**返回类型**] [**类的全路径**] [**方法名称**] (**参数列表**) )

* 权限修饰符  *****  代表任意修饰符
* 返回类型可以不写  ( 奇奇怪怪我感觉权限修饰符才应该可以省略 ,  返回类型不要省略 )
* 全路径和方法之间用**点  “.”  **连接
* 参数列表可以用两个点代表 **“..”**
* 对类的所有方法进行增强则用  **“*”** 代替
* 对包下所有类的所有方法进行增强用 **“*”**  代替类名和方法名

###### 代码实现步骤

1. 编写配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   
   <!--导入命名空间contex和aop-->
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"			
          xmlns:context="http://www.springframework.org/schema/context"	
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/aop 
          http://www.springframework.org/schema/aop/spring-aop.xsd
          http://www.springframework.org/schema/context 
          http://www.springframework.org/schema/context/spring-context.xsd">
   
       <context:component-scan base-package="Dao"/>	<!--开启注解自动扫描进行创建对象-->
       <aop:aspectj-autoproxy/>						<!--开启Aspect的使用声明创建代理-->
   </beans>
   ```

2. 编写代理类与被代理类

   ```java
   //被代理类,是UserDao的一个实现类
   @Component					//注解没有声明value时为类名(首字母小写)
   public class UserDaoImpl implements UserDao {
       @Override
       public int update(int a, int b) {
           System.out.println(a + b);
           return a + b;
       }
   }
   
   //代理类
   @Component
   @Aspect     //生成代理对象
   public class UserBefore {
       //注解和切入点表达式
       @Before("execution(*  Dao.UserDaoImpl.update(..))")
       //增加的逻辑表达式
       public void before() {
           System.out.println("before ,,,,,,");
       }
   }
   ```

> 在编写测试类是传给getBean的字节码文件必须是UserDaoImpl实现的接口的字节码文件,
> 使用UserDaoImpl.class会给出类型不匹配的错误,猜想创建出来的代理对象不是
> UserDaoImpl的子类,而是他实现接口UserDao的另一个实现类

###### 通知的不同类型的不同注解

* **@After**   最终通知 ,  不会一定会执行

* **@AfterReturning**   返回通知 ,  如果抛出异常则不会执行

* **@AfterThrowing **  在抛出异常之后

* **@Around **  环绕通知

  * 有参数**ProceedingJoinPoint的proceed**方法代表执行原方法

    ```java
    @Around("execution(*  Dao.UserDaoImpl.update(..))")
    public int around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
    	System.out.println("环绕通知的前面的部分");			//增加的在前面的业务逻辑
    	Object proceed = proceedingJoinPoint.proceed();		//执行原先的函数
    	System.out.println("环绕通知的后面的部分");			//增加的在后面的业务逻辑
    	return (int) proceed;								//原先的返回值
    }
    ```

* 几种通知的方式的执行顺序是随机的,但是可以强制限定执行顺(事务?好像后面会讲)
  也有说是因为底层逻辑更新了所以不同不知道,以后再说

###### 相同切入点的抽取

1. 定义函数添加切入点注解**@Pointcut**
2. 添加属性value ,  属性值写公用的切入点表达式
3. 直接使用函数代替切入点表达式  ( 需要带括号 )  

###### 设置增强类的优先级

在增强类添加注解**@Order**里面设置数字类型的优先级 ,  数字越小优先级越高

##### 基于xml配置文件

太傻了 ,  我才不学这个东西,这么复杂我还是用注解吧 ,  这个跳过啦



## JDBC Template操作数据库

> Spring中对jdbc的封装
> 可以进行对数据库的增删改查
> 单独操作或整合模板

### 底层实现

#### 配置数据库连接池

```xml
<!--配置创建druid连接池对象, 设置关闭连接参数-->
<bean id="dataSoure" class="com.alibaba.druid.pool.DruidDataSource"
		destroy-method="close">
    <!--注入连接池配置参数-->
	<property name="url" value="jdbc:mysql://localost:8080:shop"/>
	<property name="username" value="root"/>
	<property name="password" value="APTX"/>
	<property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
</bean>

```

#### 配置创建数据库模板对象

```xml
<!--配置创建数据库连接池-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSoure"/>
</bean>

```

#### 使用函数进行增删改操作

> 没啥新东西就是个方法,跟之前的jdbcUtils基本一致,就是函数名字啥的不一样  



### Spring声明式事务管理

#### 配置文件

```xml
<!--配置事务管理接口实现类-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--注入数据源-->
	<property name="dataSource" ref="dataSoure"/>
</bean>
<!--开启事务管理, 指定事务管理器-->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

#### 添加注解

将注解**@Transactiona**l添加到类或者方法上

#### 注解参数配置

* **propagation()**    事务的传播行为
* **isolation()**    事务的隔离级别
* **Timeout**    超时时间
* **readOnly()**    是否只读 
* **rollbackFor()**    回滚
* **noRollbackFor()**     不回滚

 ##### 事物的传播行为

* 多事务方法之间调用,事务间的管理方式
* 事务方法指对数据库表中数据进行更改的操作
* 多事务方法指在一事务方法中调用另一事务方法
* 或无事务调用有事务,或有事务调用无事务

<img src="C:\Users\aptx\AppData\Roaming\Typora\typora-user-images\image-20210520091044112.png" alt="image-20210520091044112" style="zoom:75%;" />

###### required  必须的

> 方法必须在事物中运行,如果当前上下文中没有事物则就自己开启事物
> 如果当前上下文存在事物,则就在当前存在的事物中运行

```java
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
	methodB();
	// do something
}
@Transactional(propagation = Propagation.REQUIRED)
public void methodB() {
    // do something
}
```

单独调用methodB方法时，因为当前上下文不存在事务，所以会开启一个新的事务。 
调用methodA方法时，因为当前上下文不存在事务，所以会开启一个新的事务。当执行到methodB时，methodB发现当前上下文有事务，因此就加入到当前事务中来。

###### required_new

> 方法必须在事物中运行,如果当前上下文中没有事物则就自己开启事物
> 但是如果上下文存在事物,则将该事物暂停挂起,开启自己的事物,
> 自己的事物结束后,继续原先的事物

使用PROPAGATION_REQUIRES_NEW,需要使用 JtaTransactionManager作为事务管理器。 
它会开启一个新的事务。如果一个事务已经存在，则先将这个存在的事务挂起。

```java
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
	doSomeThingA();
	methodB();
	doSomeThingB();
	// do something else
}
// 事务属性为REQUIRES_NEW
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {
    // do something
}
```

当调用

```java
main{  
	methodA();
} 
```

相当于调用

```java
main(){
	TransactionManager tm = null;
	try{
		//获得一个JTA事务管理器
		tm = getTransactionManager();
		tm.begin();//开启一个新的事务
		Transaction ts1 = tm.getTransaction();
		doSomeThing();
		tm.suspend();//挂起当前事务
		try{
			tm.begin();//重新开启第二个事务
			Transaction ts2 = tm.getTransaction();
			methodB();
			ts2.commit();//提交第二个事务
		} Catch(RunTimeException ex) {
			ts2.rollback();//回滚第二个事务
		} finally {
			//释放资源
		}
        
		//methodB执行完后，恢复第一个事务
		tm.resume(ts1);
		doSomeThingB();
		ts1.commit();//提交第一个事务
	} catch(RunTimeException ex) {
		ts1.rollback();//回滚第一个事务
	} finally {
		//释放资源
	}
	
}
```

在这里，我把ts1称为外层事务，ts2称为内层事务。从上面的代码可以看出，ts2与ts1是两个独立的事务，互不相干。Ts2是否成功并不依赖于 ts1。如果methodA方法在调用methodB方法后的doSomeThingB方法失败了，而methodB方法所做的结果依然被提交。而除了 methodB之外的其它代码导致的结果却被回滚了

###### 超时时间

> 设定超时时间,超过时间自动回滚,默认为-1,以秒为单位

###### readOnly

> 默认**FALSE**可读可写, 值为**TRUE**时只能做查询操作不能进行更改

###### rollback

> 设置哪些异常回滚,哪些异常不回滚

#### 完全注解开发

1. 新建类添加**@Configuration**注解, 标明注解类
2.  添加注解**@ComponentScan**指明组件扫描范围
3. 添加注解**@EnableTransactionManagement**开启事物
4. 在配置类中创建返回数据库连接池方法添加**@Bean**注解
5. 在返回函数中设置配置参数,**@Bean**注解代表将返回值交给IOC管理
6. 配置**JdbcTemplate** 返回函数跟数据库连接池类似
7. 在返回函数中给**JdbcTenplate**配置数据库连接池对象,使用参数构造注入 (自动注入)
8. 创建事物管理器对象 ,  相同方式注入数据库连接池



  
