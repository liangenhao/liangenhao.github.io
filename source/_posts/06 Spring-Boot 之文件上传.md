---
title: 06 Spring Boot 之文件上传
date: 2018-09-03
categories: Spring Boot
tags: [Spring Boot]
---

Spring Boot的文件上传，常使用`MultipartFile`类，该类源自SpringMVC。

一、【文件上传三要素】：

1. 表单提交方式：`method="post"`。
2. 表单类型：`enctype="multipart/form-data"`。
3. 表单项中有一个或者多个的`file`类型。

<!-- more -->

二、【编写HTML页面】：

```html
<form enctype="multipart/form-data" method="post" action="/upload">
    文件:<input type="file" name="head_img"/>
    姓名:<input type="text" name="name"/>
    <input type="submit" value="上传"/>
</form>
```

三、【编写Controller】：

```java
@RequestMapping(value = "upload")
@ResponseBody
public JsonData upload(@RequestParam("head_img") MultipartFile file,HttpServletRequest request) {

    //file.isEmpty(); 判断图片是否为空
    //file.getSize(); 图片大小进行判断

    String name = request.getParameter("name");
    System.out.println("用户名："+name);

    // 获取文件名
    String fileName = file.getOriginalFilename();	        
    System.out.println("上传的文件名为：" + fileName);

    // 获取文件的后缀名,比如图片的jpeg,png
    String suffixName = fileName.substring(fileName.lastIndexOf("."));
    System.out.println("上传的后缀名为：" + suffixName);

    // 文件上传后的路径
    fileName = UUID.randomUUID() + suffixName;
    System.out.println("转换后的名称:"+fileName);

    File dest = new File(filePath + fileName);

    try {
        file.transferTo(dest);

        return new JsonData(0, fileName);
    } catch (IllegalStateException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return  new JsonData(-1, "fail to save ", null);
}
```

> `MultipartFile`对象的`transferTo`方法，用于文件保存，效率和操作比原先使用`fileOutStream`方便和高效。

四、【Spring Boot 对上传文件大小的配置】：

Spring Boot 默认对上传文件的大小限制是：每个文件的配置最大为1Mb，单次请求的文件的总数不能大于10Mb 。

配置上传文件大小有两种方式：

1、方式一：配置文件配置：

```properties
spring.servlet.multipart.max-file-size=30Mb  
spring.servlet.multipart.max-request-size=30Mb
```

> 需要注意的是，如果是Spring Boot 1.5.x的配置是：
>
> ```properties
> spring.http.multipart.max-file-size=30Mb  
> spring.http.multipart.max-request-size=30Mb
> ```
>
> 

2、方式二：注入Bean：

```java
@Bean  
public MultipartConfigElement multipartConfigElement() {  
    MultipartConfigFactory factory = new MultipartConfigFactory();  
    //单个文件最大  
    factory.setMaxFileSize("10240KB"); //KB,MB  
    /// 设置总上传数据总大小  
    factory.setMaxRequestSize("1024000KB");  
    return factory.createMultipartConfig();  
}  
```

五、【自定义文件上传路径】：

在配置文件中，自定义文件需要上传的路径：

```properties
web.images-path=/Users/enhao/Desktop
spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/,classpath:/test/,file:${web.upload-path} 
```

> 上传路径的key可以自定义。然后将其添加到静态资源路径下（static-locations），这样可以直接访问到。
>
> 然后程序中直接引用配置文件中的路径即可。