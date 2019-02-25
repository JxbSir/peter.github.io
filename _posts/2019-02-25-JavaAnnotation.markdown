---
layout: post
title: Java注解说明（持续更新）
date: 2019-02-25 08:00:00
tags: Java
---

## 什么是注解？
> Java注解又称Java标注，是Java语言5.0版本开始支持加入源代码的特殊语法元数据。
> 
> Java语言中的类、方法、变量、参数和包等都可以被标注。和Javadoc不同，Java标注可以通过反射获取标注内容。在编译器生成类文件时，标注可以被嵌入到字节码中。Java虚拟机可以保留标注内容，在运行时可以获取到标注内容。 当然它也支持自定义Java标注。

## 内置的注解
> Java 定义了一套注解，共有 7 个，3 个在 java.lang 中，剩下 4 个在 java.lang.annotation 中

#### 作用在代码的注解是

- @Override - 检查该方法是否是重载方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
- @Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
- @SuppressWarnings - 指示编译器去忽略注解中声明的警告。

#### 作用在其他注解的注解(或者说 元注解)是:
- @Retention - 标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。
- @Documented - 标记这些注解是否包含在用户文档中。
- @Target - 标记这个注解应该是哪种 Java 成员。
- @Inherited - 标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)

#### 从 Java 7 开始，额外添加了 3 个注解:
- @SafeVarargs - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
- @FunctionalInterface - Java 8 开始支持，标识一个匿名函数或函数式接口。
- @Repeatable - Java 8 开始支持，标识某注解可以在同一个声明上使用多次。

## Spring注解
- @Controller：用于标记在一个类上，使用它标记的类就是一个SpringMVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法。通俗来说，被Controller标记的类就是一个控制器，这个类中的方法，就是相应的动作。
- @RestController：只返回页面，显示用
    - @Controller和@RestController的区别？
        - @RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。
        - 如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，或者html，配置的视图解析器 InternalResourceViewResolver不起作用，返回的内容就是Return 里的内容。 
        - 如果需要返回到指定页面，则需要用 @Controller配合视图解析器InternalResourceViewResolver才行。 如果需要返回JSON，XML或自定义mediaType内容到页面，则需要在对应的方法上加上@ResponseBody注解。
- @RequestMapping：是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径
- @RequestBody：
	- 该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；
	- 再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。
- @ResponseBody：
	- 该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。
	- 返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用； 
- @Service("userService”)：注解是告诉Spring，当Spring要创建UserServiceImpl的的实例时，bean的名字必须叫做"userService"，这样当Action需要使用UserServiceImpl的的实例时,就可以由Spring创建好的"userService"，然后注入给Action。
    - 声明了一个bean，其他类才可以使用@Autowired来自动注入
    - 在bean中的id是userService
- @Repository(value="userDao”)：
    - 告诉Spring，让Spring创建一个名字叫“userDao”的UserDaoImpl实例。
    - 用来表明该类是用来执行与数据库相关的操作（即dao对象），并支持自动处理数据库操作产生的异常
- @Autowired：它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过 @Autowired的使用来消除 set ，get方法。
- @Resource
    - @Resource后面没有任何内容，默认通过name属性去匹配bean，找不到再按type去匹配
    - 指定了name或者type则根据指定的类型去匹配bean
    - 指定了name和type则根据指定的name和type去匹配bean，任何一个不匹配都将报错
    - @Autowired和@Resource两个注解的区别：
        - @Autowired默认按照byType方式进行bean匹配，@Resource默认按照byName方式进行bean匹配
        - @Autowired是Spring的注解，@Resource是J2EE的注解，这个看一下导入注解的时候这两个注解的包名就一清二楚了
- @Component：泛指各种组件，就是说当我们的类不属于各种归类的时候
- @Scope：Spring默认产生的bean是单例的，假如我不想使用单例怎么办，xml文件里面可以在bean里面配置scope属性。注解也是一样，配置@Scope即可，默认是"singleton"即单例，"prototype"表示原型即每次都会new一个新的出来。