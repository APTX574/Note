# 设计模式

## 单例模式
### 介绍
>这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。
1. 单例类只能有一个实例。
2. 单例类必须自己创建自己的唯一实例。
3. 单例类必须给所有其他对象提供这一实例。

### 实现方式



**创建私有化示例**

```java
private static Singleton singleton;
```
**私有化构造方法**

```java
public class Singleton {
    private Singleton() {
        singleton = new Singleton();
    }
}
```

**提供获取对象的public static get方法**

```java
public static Singleton newInstance() {
	return singleton;
}
```





### 单例模式的两种实现方式



**饿汉模式** 

> 上面示例的写法 类加载时便创建对象



**懒汉模式**   *延迟加载模式*

在get方法体中判断对象引用是否为null,若不为null则创建对象

```java
public static Singleton newInstance() {
	if (singleton==null){
		singleton = new Singleton();
	}
	return singleton;
}
```

###### 线程不安全问题

```Java
if (singleton==null){				//此处在多线程运行时可能会多个线程满足if从而创建多个对象
	singleton = new Singleton();
}
```

###### 线程安全的懒汉模式

```java
public class Singleton {
    private static Singleton singleton;
    private Singleton() {
    }
    public static Singleton newInstance() {
        if (singleton==null){				//在未实例化时多个线程都将进去这个if代码块
            synchronized (Class.class) {	//只有一个线程拿到同步锁进去第二个if实例化一次
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

+++

### 示例

1. java Runtime类

   ```java
   	private static final Runtime currentRuntime = new Runtime();
   
   	private static Version version;
   
       /**
        * Returns the runtime object associated with the current Java application.
        * Most of the methods of class {@code Runtime} are instance
        * methods and must be invoked with respect to the current runtime object.
        *
        * @return  the {@code Runtime} object associated with the current
        *          Java application.
        */
       public static Runtime getRuntime() {
           return currentRuntime;
       }
   
       /** Don't let anyone else instantiate this class */
       private Runtime() {}
   
   ```

2. Windows任务管理器





## 工厂模式

### 介绍

> 它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。
> 
> 定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

### 示例

1. druid数据库连接池技术中的连接池工厂

   ```java
   private static DruidDataSource dataSource;
       static {
           try {
               //定义资源变量
               Properties prop = new Properties();
               //通过类的加载器获取资源输入流
               InputStream res = JdbcUtil.class.getClassLoader().getResourceAsStream("jdbc.properties");
               prop.load(res);     //载入资源
               //通过数据库连接池工厂获得数据库连接池
               dataSource = (DruidDataSource) DruidDataSourceFactory.createDataSource(prop);   
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   
       public static Connection getConnection() {
           Connection conn = null;
           try {
               conn = dataSource.getConnection();      //通过数据库连接池获得连接
               return conn;
           } catch (SQLException throwables) {
               throwables.printStackTrace();
           }
           return null;
       }
   ```

   DruidDataSourceFactory源码

   ```java
   public static DataSource createDataSource(Properties properties) throws Exception {
   	return createDataSource((Map)properties);
   }
   public static DataSource createDataSource(Map properties) throws Exception {
   	DruidDataSource dataSource = new DruidDataSource();			//生产对象
   	config(dataSource, properties);								//将对象与资源文件关联
   	return dataSource;											//返回对象
   }
   ```

2. Apache Commons FileUpload

   ```java
   public interface FileItemFactory {
       FileItem createItem(String var1, String var2, boolean var3, String var4);
   }
   
   
   public FileItem createItem(String fieldName, String contentType, boolean isFormField, String fileName) {
   	DiskFileItem result = new DiskFileItem(fieldName, contentType, isFormField, 
        fileName,this.sizeThreshold, this.repository);
   	result.setDefaultCharset(this.defaultCharset);
   	FileCleaningTracker tracker = this.getFileCleaningTracker();
   	if (tracker != null) {
   		tracker.track(result.getTempFile(), result);
   	}
   	return result;
   }
   ```

   