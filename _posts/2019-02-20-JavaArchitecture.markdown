---
layout: post
title: Java后端开发之基于SpringBoot三层架构
date: 2019-02-19 20:00:00
tags: Java
---

### 创建SpringBoot项目
> 上一章已经讲过如何创建一个`SpringBoot`的项目，回顾请[点击这里](../firstSpringboot)

### 三层架构介绍
> 三层架构(3-tier architecture) 通常意义上的三层架构就是将整个业务应用划分为：界面层（User Interface layer）、业务逻辑层（Business Logic Layer）、数据访问层（Data access layer）。
> 
- **数据访问层**：主要看数据层里面有没有包含逻辑处理，实际上它的各个函数主要完成各个对数据文件的操作。而不必管其他操作。
- **业务逻辑层**：主要负责对数据层的操作。也就是说把一些数据层的操作进行组合。
- **表示层**：主要对用户的请求接受，以及数据的返回，为客户端提供应用程序的访问。
>
> 依赖关系：ULL -> BLL -> DAL，ULL不可直接引用DAL层

### 代码结构
> 下图是上一章创建的`SpringBoot`项目
> 
> ![](http://xbqn.nbshk.cn/20190220103735_6PnErj_Screenshot.jpeg)
> 
> 我们先把`src`删除，`target`是编译之后产生的目录，这边可以处理
> 
> 然后在项目上`右键`，点击`New Moudle`，按照创建上一章`SpringBoot`项目一样即可
> 
> ![](http://xbqn.nbshk.cn/20190220104512_wXrVTb_Screenshot.jpeg)
> 
> 创建3个`Module`：`web=界面层(这边用来做启动)`-`biz=业务层`-`dal=数据层`
> 
> 还需要建一个`Module`：`share`=用于暴露接口
> 
> 每一层只要留下`src`、`*.iml`、`pom.xml`即可
> 
> 因为是`Module`所以每层里面自动会有一个`Application.java`的启动类，把`Biz`与`Dal`层的删除，只留下`web`层来启动就好了
> 
> ![](http://xbqn.nbshk.cn/20190220105318_CNogOr_Screenshot.jpeg)
> 
> 然后我们需要去各层项目中的`pom.xml`配置模块依赖
> 
> 首先是最外层的`pom.xml`，这边添加的是模块属性，代表这个根项目中包含3个模块
> 
> ![](http://xbqn.nbshk.cn/20190220171240_3vwbNT_Screenshot.jpeg)
>
> 然后是去`web`层，这边添加的不是模块属性了，因为是在模块中配置了，此时应该配置依赖了，`web`层应该是依赖`biz`；然后同理`biz`去配置依赖`dal`就好了
> 
> ![](http://xbqn.nbshk.cn/20190220171342_RjFP8m_Screenshot.jpeg)
>  
> 然后将`biz`与`dal`层中的`application.properties`重命名各层项目的名字，如`application-biz.properties`
> 
> ok，此时配置已经告一段落，接下来就是如何把三层访问连接起来...

### 三层访问
> #### DAL层
> *前言：数据层我们是选用`Mybatis`框架，在后面会使用到，不熟悉`Mybatis`的暂时可以先`google`了解下*
> > 在`dal`中的`application-biz.properties`配置数据库信息(`MySQL`的安装之前的文章也有提过)，因为我们是用`Mybatis`框架，所以同时需要配置`Mybatis`
> > 
```
//mysql
spring.datasource.url=jdbc:mysql://localhost:3306/TestDB
spring.datasource.username=root
spring.datasource.password=@ASDasd123
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
//mybatis config
mybatis.mapper-locations=classpath:/mappering/*mapper.xml
mybatis.configuration.use-column-label=true
mybatis.configuration.use-generated-keys=false
mybatis.configuration.auto-mapping-behavior=FULL
mybatis.configuration.default-fetch-size=100
mybatis.configuration.local-cache-scope=SESSION
mybatis.configuration.lazy-loading-enabled=false
mybatis.configuration.jdbc-type-for-null=OTHER
mybatis.configuration.lazy-load-trigger-methods=equals,clone,hashCode,toString
mybatis.configuration.safe-row-bounds-enabled=false
mybatis.configuration.default-statement-timeout=30
mybatis.configuration.default-executor-type=SIMPLE
mybatis.type-aliases-package=cn.jinxuebin.demodal.model
mybatis.configuration.map-underscore-to-camel-case=true
mybatis.configuration.multiple-result-sets-enabled=true
mybatis.configuration.cache-enabled=true
```
> > 在pom.xml中需要加入依赖
> >
> > ![](http://xbqn.nbshk.cn/20190220171441_IazMtU_Screenshot.jpeg)
> >
> > 由于`datasource`的配置是在`dal`层的`application-biz.properties`里，项目启动的时候不会读取这个配置，所以需要加一个配置类`DalConfig.java`
> > 
```java
package cn.jinxuebin.demodal.config;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
/**
 * @author Peter
 * @description DAL配置类
 * @data 2019-02-20
 */
@Configuration
@MapperScan(basePackages = "cn.jinxuebin.demodal.dao")
@PropertySource(value= {"classpath:application-dal.properties"})
public class DalConfig {
}
```
> >
> > 我已经在本地的数据库简单建了一张测试的`Products`表
> > 
> > ![](http://xbqn.nbshk.cn/20190220140838_yZBIOV_Screenshot.jpeg)
> > 
> > 然后利用[`Mybatis-generator`](http://www.mybatis.org/generator/)工具生成相应的`Dao/Model/Mapper`
> > 
> > 关键文件就这么3个
> >  
> > ![](http://xbqn.nbshk.cn/20190220141608_AVuVBC_Screenshot.jpeg)
> > 
- **mybatis-generator-core-1.3.7.jar** 它就是生成工具`jar`包 
- **mysql-connector-java-8.0.15.jar** 它就是`mysql`连接`jar`包，版本需一致
- **generatorConfig.xml** 这个就是生成器的配置文件了
- **src** 这个是空的就可以，代码会自动生成到这边
> > 
> > `generatorConfig.xml`根据需求配置，注意`namespace/type/parameterType`检查是否正确
> > 
> > 执行命令`java -jar mybatis-generator-core-1.3.7.jar -configfile generatorConfig.xml -overwrite`就生成成功了，然后把这些类放到工程里
> > 
> > `DAL`层的代码类如下图，具体代码里的注解请直接看源码
> > 
![](http://xbqn.nbshk.cn/20190220163911_d33AgV_Screenshot.jpeg)
> 
> #### Share
> *前言*： `Share`不属于三层的范围，它只是`BLL`层对外开放的接口层，当其他应用需要调我们这个服务的时候就会用到`Share`这个`Module`；`Share`还需要提供`BO`实体类，也是用来提供给外部
> > 本章就简单一点，建一个`interface`，里面就一个方法
> >
```java
package cn.jinxuebin.demoshare;
import cn.jinxuebin.demoshare.bo.ProductsBO;
/**
 * @author Peter
 * @description Products业务接口类
 * @data 2019-02-20
 */
public interface IProductsService {
    ProductsBO getProductById(int id);
}
``` 
> 
> #### BLL
> *前言*： `BLL`是业务逻辑层，基本上应该都是业务代码，所以不会像`DAL`那么多配置，相对比较明朗
> > 首先当然也是`BLL`的配置类：`BizConfig.java`
> > 
```java
package cn.jinxuebin.demodal.config;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.mybatis.spring.annotation.MapperScan;
/**
 * @author Peter
 * @description DAL配置类
 * @data 2019-02-20
 */
@Configuration
@MapperScan(basePackages = "cn.jinxuebin.demodal.dao")
@PropertySource(value= {"classpath:application-dal.properties"})
public class DalConfig {
}
```
> > 然后就是定义一个Imp实现类来实现IProductsService
> > 
```java
package cn.jinxuebin.demobiz.imp;
import cn.jinxuebin.demobiz.ProductsManager;
import cn.jinxuebin.demoshare.IProductsService;
import cn.jinxuebin.demoshare.bo.ProductsBO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
/**
 * @author Peter
 * @description ProductsService实现类
 * @data 2019-02-20
 */
@Service
public class ProductsImp implements IProductsService {
    @Autowired
    private ProductsManager productsManager;
    @Override
    public ProductsBO getProductById(int id) {
        return productsManager.getProductById(id);
    }
}
```
> > 
> > 然后需要一个实现业务逻辑的Manager类
> > 
```java
package cn.jinxuebin.demobiz;
import cn.jinxuebin.demodal.dao.ProductsMapper;
import cn.jinxuebin.demodal.model.Products;
import cn.jinxuebin.demoshare.bo.ProductsBO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
/**
 * @author Peter
 * @description 商品业务类
 * @data 2019-02-20
 */
@Service
public class ProductsManager {
    @Autowired
    private ProductsMapper productsMapper;
    public ProductsBO getProductById(int id) {
    	//将DO实体类转化成BO实体类，这边就不封装函数了，简单实现下
        Products dao = productsMapper.selectByPrimaryKey(id);
        ProductsBO bo = new ProductsBO();
        bo.setItemid(dao.getItemid());
        bo.setDesc(dao.getItemdesc());
        bo.setName(dao.getItemname());
        bo.setPrice(dao.getPrice());
        bo.setShopid(dao.getShopid());
        return bo;
    }
}
```
> 
> #### ULL
> *前言*：这边我们就是一个`Web`的`Module`用来做启动
> > 创建`Application.java`
> > 
```java
package cn.jinxuebin.demoweb;
import cn.jinxuebin.demobiz.config.BizConfig;
import cn.jinxuebin.demodal.config.DalConfig;
import cn.jinxuebin.demoshare.IProductsService;
import cn.jinxuebin.demoshare.bo.ProductsBO;
import com.fasterxml.jackson.databind.util.JSONPObject;
import com.mysql.cj.xdevapi.JsonArray;
import net.minidev.json.JSONObject;
import net.minidev.json.JSONValue;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Import;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
@SpringBootApplication
@RestController
@Import({BizConfig.class, DalConfig.class})
public class DemoWebApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoWebApplication.class, args);
	}
	@Autowired
	private IProductsService service;
	@RequestMapping("/book")
	public String home(@RequestParam("id") int id) {
		ProductsBO bo = service.getProductById(id);
		return JSONValue.toJSONString(bo);
	}
}
```

### OK，这样就基本完成了，然后启动下看看，若没报错就成功了
> 访问地址：`http://127.0.0.1:8080/book?id=1`，可以看到数据了
> 
![](http://xbqn.nbshk.cn/20190220170125_FRbzEi_Screenshot.jpeg)


