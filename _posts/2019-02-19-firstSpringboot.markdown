---
layout: post
title: 创建第一个SpringBoot项目
date: 2016-02-19 20:30:30.000000000 +08:00
---

### 创建项目
- `Create New Project`
- 选择`Spring Initializr`

	![](http://xbqn.nbshk.cn/20190219100300_jaLdDE_Screenshot.jpeg)
- 根据需求配置Project Metadata

	![](http://xbqn.nbshk.cn/20190219100608_oGtSGc_Screenshot.jpeg)
- 直接`Next` - 暂时不用勾选其他
- 下一步配置项目名称与项目路径，然后`Finish`，项目就创建好了
> **注意**：创建项目后，`SpringBoot`会自动配置从`Maven`下载依赖，所以需要点时间，可以看到底部有进度条

### 启动项目
> 配置`pom.xml`，这个配置就是项目的配置文件，看下是否有这些依赖
> 
> ![](http://xbqn.nbshk.cn/20190219101941_beyQLu_Screenshot.jpeg)
> 在`src`目录里找到有一个类`FirstspringbootApplication`，添加注解`@RestController`
> 
> ![](http://xbqn.nbshk.cn/20190219102424_Ui4vLw_Screenshot.jpeg)
> 右键`Run FirstspringbootApplication`，看到以下日志，就代表Springboot配置完毕，端口是8080
> 
> ![](http://xbqn.nbshk.cn/20190219102453_zIKPBu_Screenshot.jpeg)
> 然后，我们访问下看看
> 
> ![](http://xbqn.nbshk.cn/20190219102600_FSPpnp_Screenshot.jpeg)
> `No message available`报错了，那是因为我们没有配置这个地址的接收入口；我们来添加下
> 
> ![](http://xbqn.nbshk.cn/20190219103132_FCgDi9_Screenshot.jpeg)
> 然后在启动访问下，ok，成功了
> 
> ![](http://xbqn.nbshk.cn/20190219103549_ICrMy6_Screenshot.jpeg)
> 

### 下一节继续介绍三层访问设计（Service+Biz+DAL）

