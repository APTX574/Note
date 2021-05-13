## fileupload.jar

> 用于在servlet中下载接受客户端传来的文件,依赖于io的包

***Maven配置脚本***

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
```



***示例***

```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) {
    //利用ServletFileUpload.isMultipartContent(request)判断req是否是字节流传输给后端
	if (ServletFileUpload.isMultipartContent(request)) {
        
        //创建工厂
		FileItemFactory fileItemFactory = new DiskFileItemFactory();
        
        //利用工厂获取ServletFileUpload对象
		ServletFileUpload fileUpload = new ServletFileUpload(fileItemFactory);
		try {
            
            //通过请求对象获取FileItem集合
			List<FileItem> fileItems = fileUpload.parseRequest(request);
			for (FileItem item : fileItems) {
                
                //判断Fileitem是否是文件，是文件则返回false
				if (item.isFormField()) {
					System.out.println("字段名" + item.getFieldName());
					System.out.println("值" + item.getString("UTF-8"));
				} else {
					System.out.println("文件名" + item.getFieldName());
                    
                    //利用字节输出流将文件输出到本地磁盘
					item.write(new File("D:\\" + item.getName()));
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

