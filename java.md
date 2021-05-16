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
-----

###### Cookie

1. 概念

   > 1. 是服务器创建的传递给客户端保存的一对键值对,  name--->value
   >
   > 2. 服务器指定服务端保存的时间,  其后每次客户端访问此cookie对应
   >
   >    的网页时,  客户端将这个键值对发给服务器,   服务器根据此cookie
   >
   >    给出判断传递相应页面,一般用于保存用户名等操作

2. 常用用法
	```java
	Cookie cookie = new Cookie("name", user1.getName()); //创建Cookie对象
	
	cookie.setMaxAge(60*60*24*7);	//设置Cookie的生命时长, 正数单位为秒, 负数代表为此次会话(默认), 0代表立刻删除
									//一般利用设置时长为0删除Cookie对象
	cookie.setPath(request.getContextPath() + "xxx");	//设置访问路径,只有满足此路径可以获得这个Cookie
	
   Cookie[] cookies = request.getCookies();		//获得请求提供的Cookie数组
   
   Cookie cookie1;
   
   for (Cookie cook : cookies) {					//便利数组获取需要的元素
		if (cook.getName() == "xxx") {
			cookie1 = cook;							//给对象重新赋值
   	}
	}
	response.addCookie(cookie);						//将对象传递给客户端
	```
	
	
	
	-----

   
   
   



