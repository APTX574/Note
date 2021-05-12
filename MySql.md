## MySql学习笔记
### SQL语句
#### DQL数据库查询语言
​	用于对数据库的增删改查操作

***代码示例***

##### 单表查询和自查询

```sql
SELECT DISTINCT * FROM `user` AS u	//查询的字段, 查询的表格和对应的命重名, 多个列之间逗号间隔 DISTINCT 代表去重
WHERE u.id=(
	SELECT id FROM `user`	//子查询, 在这一查询结果的基础上进行上一查询
	WHERE `user`.`name`='wqs'
)   
HAVING birthday=''		//HAVING为在前一查询结果的基础上查询
ORDER BY column DESC	//ORDER BY为按column排序, desc为逆序, 默认为esc顺序
LIMIT index,num;		//index为显示的第一条数据的下标(从0开始), num为显示的条数
```

###### 显示函数

```sql
SUM(column)				//计算所写列满足条件的总值
MAX(*)					//查询字段的最大值
MIN(*)					//查询字段的最小值
COUNT(DISTINCT ,column)	//计算所写列满足条件的个数 DISTINCT 代表去重
AVG(column)				//计算所写列满支条件的均值
CONCAT(str+column)		//将查询到的结果拼接显示
```

###### 查询表达式

```sql
IN(1,2,3)				//结果在后面列表中
BETWEEN 1 AND 10		//结果在1和10之间, 数字必须前小后大
LIKE	'%like_'		//结果与后面的字符型匹配, %代表任意字符, _代表单个字符
IS NULL 				//结果不是null
```

---

##### 多表联合查询

1. 多表内连接

   > 只显示笛卡尔积中满足连接条件的项,  不区分主表和从表

```sql
SELECT * FROM `user` AS u		//显示字段, 连接的第一个表
INNER JOIN IN column AS c		//内连接的表
ON u.id=c.id					//内连接的连接条件. 满足条件的行将被显示
HAVING birthday=''
ORDER BY column DESC 
LIMIT index,num
```

2. 多表外连接

   > 以主表为主, 显示主表和从表和主表的笛卡尔积后主表所有的项,  包括不符合连接条件的项
   >
   > > 左外连接,  左边的表为主表
   >
   > >右外连接,  右边的表为主表
   >
   > > 全外连接,  显示左边和右边的所有结果,包括不满足连接条件的行

```sql
SELECT * FROM `user` AS u		//外连接的左表
RIGHT JOIN IN column AS c		//外连接的右表, RIGHT/LEFT表示主表
ON u.id=c.id					//外连接的连接条件,主表的所有行将被显示
WHERE id IS NULL				//可以去掉连接条件中满足内连接的行
HAVING birthday=''
ORDER BY column DESC 
LIMIT index,num
```

---

##### 分组查询

> 可以将数据按指定字段分组

#### DDL据库定义语言



#### DML数据库修改语言
#### TCL事物控制语言

#### druid数据库连接池
***操作步骤***

``` java
import com.alibaba.druid.pool.*; //载入数据库连接池资源包

import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

public class JdbcUtil {
    /*定义数据库连接池常量并且通过静态代码块初始化,类似单列*/
    private static DruidDataSource dataSource;
    static {
        try {
            //定义资源变量
            Properties prop = new Properties();
            //通过类的加载器获取资源输入流
            InputStream res = JdbcUtil.class.getClassLoader().getResourceAsStream("jdbc.properties");
            prop.load(res);     //载入资源
            dataSource = (DruidDataSource) DruidDataSourceFactory.createDataSource(prop);   //通过数据库连接池工厂获得数据库连接池
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

    public static void close(Connection connection) {
        if (connection != null) {
            try {
                connection.close();               //关闭数据库连接
            } catch (SQLException throwables) {
                throwables.printStackrace();
            }
        }
    }
}
```

***资源文件properties格式***

```html
driverClassName=com.mysql.cj.jdbc.Driver	<-->连接驱动类的路径</-->
url=jdbc:mysql://localhost:3306/shop		<-->连接数据库的url</-->
username=root								<-->连接用户名</-->
password=APTX								<-->连接密码</-->
```

***注意事项***

1. 应该导入最新的jar包,  且`driverClassName=com.mysql.cj.jdbc.Driver`


+++

#### dbutils的使用

***操作步骤***

```java
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;		//处理返回一个对象
import org.apache.commons.dbutils.handlers.BeanListHandler;	//处理返回一组对象
import org.apache.commons.dbutils.handlers.ScalarHandler;	//处理返回一个数值
import org.apache.commons.dbutils.ResultSetHandler;			//上面类的接口

import utils.JdbcUtil;

import java.sql.Connection;
import java.sql.SQLException;
import java.util.List;

/**
 * 利用获得德鲁伊数据库连接，利用JbUtils实现增删改查工作
 *
 * @author aptx
 */
public abstract class BaseDao<T> {
    /*获得dbutils处理数据库操作的类QueryRunner*/
    private final QueryRunner runner = new QueryRunner();
    //利用runner进行查询, 没有返回值
    public int update(Connection conn, String sql, Object... args) {
        try {
            conn = JdbcUtil.getConnection();
            return runner.update(conn, sql, args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return -1;
    }
	//返回一个javabean, 利用BeanHandler处理器,需要传入处理的类
    public T queryForOne(Connection conn,String sql,Class<T> type, Object... args)  {
        try {
            conn = JdbcUtil.getConnection();
            return runner.query(conn, sql, new BeanHandler<>(type), args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return null;
    }
	//返回一组javabean, 利用BeanListHandler处理器,需要传入处理的类
    public List<T> queryForList(Connection conn,String sql, Class<T> type, Object... args) {
        try {
            conn = JdbcUtil.getConnection();
            return runner.query(conn, sql, new BeanListHandler<>(type),args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return null;
    }
	//返回一个值, 利用ScalarHandlerr处理器,需要传入输出的类
    public int queryForSingleValue(Connection conn, String sql, Object... args) {

        try {
            conn = JdbcUtil.getConnection();
            return runner.query(conn, sql, new ScalarHandler<Integer>(), args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return -1;
    }
}
```

***注意事项***

1. sql语句编写时应该将字段名和javabean的属性名相对应, 若不一样, 应该在sql语句中给字段取别名,

   ​	别名应该与属性名对应, 字段与属性类型也应该一一对应


---

### 杂项

### 遇到的错误和解决方法





