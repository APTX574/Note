## java

###### 利用反射设置javabean

```java
    User user = null;							//创建对象的引用
    Class<User> clazz = User.class;				//获取类的字节码对象
    user = clazz.getConstructor().newInstance();	//利用反射创建实例对象
    Field name = clazz.getDeclaredField("name");	//获取需要获取的属性域
    name.setAccessible(true);						//将获取的域的限权打开
    name.set(user, "123");							//设置需要设置对象的值
            // 获得需要的方法的域,需要方法名和参数的字节码
    Method setName = clazz.getDeclaredMethod("setName", String.class);
    setName.setAccessible(true);				//将获取的域的限权打开
    setName.invoke(String.class, "123");		//调用方法
```
wanc 



