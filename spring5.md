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

  > 使用xml，创建对象时默认调用对象的无参构造方法，必须要有无参构造方法

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

> 都不需要set方法,已经封装了.

 ##### 完全注解开发

1. 创建一个配置类,代替xml配置文件 ,  要添加一个**@Configuration**注解标志注解类

2. 添加注解**@ComponentScan(basePackages={包路径})**代替开启自动扫描的配置

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

##### 底层实现代码


