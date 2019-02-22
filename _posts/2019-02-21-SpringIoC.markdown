---
layout: post
title: Spring之IoC容器
date: 2019-02-21 20:00:00
tags: Java
---


## IoC是什么？
> **全称**：Inversion of Control，它是Spring框架的核心之一。
> 
> IoC容器就是具有依赖注入功能的容器，负责实例化、定位、配置对象以及建立对象之间的依赖关系，应用程序无需在代码中new相关的对象，都由IoC来完成。
> 
> 它并不是什么技术点，而已一种设计模式。一般情况下，我们自己来控制对象，反转那么久很好理解了，久是我们只需设计好对象，由容器来帮我们控制。我们不需要通过new来创建对象，也不需要去管理这个对象的生命周期，这一切都有容器来帮我们完成。
> 
> #### 为什么要用IoC
> > IoC最大的好处：如果通过Java程序类来管理的话，那么势必会出现类与类之间的各种依赖，出现大量的耦合代码；而IoC就帮我解掉了这个耦合关系。

## 三种依赖注入方式
> Martin Fowler的那篇文章“Inversion of Control Containers and
the Dependency Injection pattern”，其中提到了三种依赖注入的方式，即构造方法注入(constructor injection)、setter方法注入(setter injection)以及接口注入(interface injection)

- **构造方法注入**：

```
public FXNewsProvider(IFXNewsListener newsListner,IFXNewsPersister newsPersister) {
	this.newsListener = newsListner;
	this.newPersistener = newsPersister; 
}
 
```
- **Setter方法注入**：

```
public class FXNewsProvider
{
	private IFXNewsListener newsListener;
	private IFXNewsPersister newPersistener;
	public IFXNewsListener getNewsListener() { 
		return newsListener;
	}
	public void setNewsListener(IFXNewsListener newsListener) {
		this.newsListener = newsListener; 
	}
	public IFXNewsPersister getNewPersistener() { 
		return newPersistener;
	}
	public void setNewPersistener(IFXNewsPersister newPersistener) {
		this.newPersistener = newPersistener; 
	}
}
```
- **接口注入**：

> 从注入方式的使用上来说，接口注入是现在不甚提倡的一种方式，基本处于“退 役状态”。因为它强制被注入对象实现不必要的接口，带有侵入性。而构造方法注入和setter 方法注入则不需要如此；这种接口注入的方式需要调用者必须实现一个指定的接口，这种方式使用比较少，一般不推荐使用

## IoC Service Provider
> Spring 的IoC容器就是一个提供依赖注入服务的`IoC Service Provider`，它的职责相对来说比较简单，主要有两个：
> 
- 业务对象的构建管理
	- 在IoC场景中，业务对象无需关心所依赖的对象如何构建如何取得，但 这部分工作始终需要有人来做。所以，`IoC Service Provider`需要将对象的构建逻辑从客户端对 象1那里剥离出来，以免这部分逻辑污染业务对象的实现。 
- 业务对象间的依赖绑定
	- 对于`IoC Service Provider`来说，这个职责是最艰巨也是最重要的，这 是它的最终使命之所在。如果不能完成这个职责，那么，无论业务对象如何的“呼喊”，也不 会得到依赖对象的任何响应(最常见的倒是会收到一个NullPointerException)。`IoC Service Provider`通过结合之前构建和管理的所有业务对象，以及各个业务对象间可以识别的依赖关系，将这些对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪状态。 
> 
> ### 如何管理
> > - **直接编码方式**：
> >  	- 就是在代码中提前注册/绑定类与对象的关系到一个单例类中，需要使用对象的时候直接从这个类中获取对象
> > - **配置文件方式**：
> > 	- 配置文件的类型并不固定，可以是文本、properties、xml文件等，都可以使用，不过一般还是采用xml的方式来管理依赖关系
> > 	- Spring是在xml中通过bean节点类配置
> > - **元数据方式**：
> > 	- 通过@Inject注解来指明需要通过构造方法注入方式；当然，注解最终也要通过代码处理来确定最终的注入关系，从这点儿来说，注解方式可以算作编
码方式的一种特殊情况。


## 两种IoC容器详解
> Spring提供了两种容器类型:`BeanFactory`和`ApplicationContext`

### 1.BeanFactory
> 基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延
迟初始化策略(lazy-load)。只有当客户端对象需要访问容器中的某个受管对象的时候，才对 该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需 要的资源有限。对于资源有限，并且功能要求不是很严格的场景，`BeanFactory`是比较合适的 IoC容器选择。
>
> 可以看下`BeanFactory`的接口定义，getBean就是我们常用的获取某个对象的方法
> 
```java
public interface BeanFactory {
    String FACTORY_BEAN_PREFIX = "&";
    Object getBean(String var1) throws BeansException;
    <T> T getBean(String var1, Class<T> var2) throws BeansException;
    Object getBean(String var1, Object... var2) throws BeansException;
    <T> T getBean(Class<T> var1) throws BeansException;
    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;
    <T> ObjectProvider<T> getBeanProvider(Class<T> var1);
    <T> ObjectProvider<T> getBeanProvider(ResolvableType var1);
    boolean containsBean(String var1);
    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;
    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;
    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;
    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;
    @Nullable
    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;
    String[] getAliases(String var1);
}
```
>
> 我们来创建一个`Product`的类，然后创建bean.xml的配置文件，增加bean节点：
> 
> `<bean id="product" class="cn.jinxuebin.demoioc.Product"></bean>`
> 
> #### **三种绑定方式**
> - 配置文件
> 	- Spring3.1之前可以直接通过`XmlBeanFactory`来获取bean，不过Spring3.1之后已经过时了，新版本需要通过下面的方式获取
> ![](http://xbqn.nbshk.cn/20190222100507_IUejLA_Screenshot.jpeg)
> **XmlBeanFactory废除原因**：`XmlBeanFactory`本身是继承`DefaultListableBeanFactory`的；Spring官方并没有该给明确的原因，只是推荐使用`DefaultListableBeanFactory`和`XmlBeanDefinitionReader`
> ![](http://xbqn.nbshk.cn/20190222102422_yKdzbM_Screenshot.jpeg)
> - 直接编码
> 
```
public static void main(String[] args) {
	DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory(); 
	BeanFactory container = (BeanFactory)bindViaCode(beanRegistry); 
	FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("djNewsProvider"); 
	newsProvider.getAndPersistNews();
}
public static BeanFactory bindViaCode(BeanDefinitionRegistry registry) {
	AbstractBeanDefinition newsProvider = new RootBeanDefinition(FXNewsProvider.class,true); 
	AbstractBeanDefinition newsListener = new RootBeanDefinition(DowJonesNewsListener.class,true); 
	AbstractBeanDefinition newsPersister = new RootBeanDefinition(DowJonesNewsPersister.class,true);
	// 将bean定义注册到容器中 registry.registerBeanDefinition("djNewsProvider", newsProvider); 
	registry.registerBeanDefinition("djListener", newsListener); 	registry.registerBeanDefinition("djPersister", newsPersister);
	// 指定依赖关系
	// 1. 可以通过构造方法注入方式
	ConstructorArgumentValues argValues = new ConstructorArgumentValues(); 
	argValues.addIndexedArgumentValue(0, newsListener); 
	argValues.addIndexedArgumentValue(1, newsPersister); 
	newsProvider.setConstructorArgumentValues(argValues);
	// 2. 或者通过setter方法注入方式
	MutablePropertyValues propertyValues = new MutablePropertyValues(); 
	propertyValues.addPropertyValue(new ropertyValue("newsListener",newsListener)); 
	propertyValues.addPropertyValue(new PropertyValue("newPersistener",newsPersister)); 
	newsProvider.setPropertyValues(propertyValues);
	// 绑定完成
	return (BeanFactory)registry;
}
```
> - 注解方式
> 	- 这个应该是最常用的也是推荐的方式：@Autowired `在Spring2.5引入了@Autowired注解` 
> 	- `@Autowire`是Spring引入的，其实Java自己也设计了一个叫`@Resource`
> 	- **@Autowired原理**：其实在启动Spring IoC时，容器自动装载了一个`AutowiredAnnotationBeanPostProcessor`后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性
> ![](http://xbqn.nbshk.cn/20190222104505_b8cQez_Screenshot.jpeg)

### 2.ApplicationContext
> ApplicationContext在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等，这些会在后面详述。ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来说，ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext类型的容器是比较合适的选择。
> 
> *由于ApplicationContext是`拥有BeanFactory所有功能`，所以在BeanFactory里介绍过的这边就不再重复赘述。*
> 
> ApplicationContext通过Xml配置，获取对象的示例代码：
> ![](http://xbqn.nbshk.cn/20190222131945_hExDVX_Screenshot.jpeg)
> 
> #### 高级特性
> - **ApplicationContext支持统一资源加载**：是因为它是继承了ResourceLoader类
> 
```
ByteArrayResource。将字节(byte)数组提供的数据作为一种资源进行封装，如果通过InputStream形式访问该类型的资源，该实现会根据字节数组的数据，构造相应的ByteArray-
InputStream并返回。
>
􏰀ClassPathResource。该实现从Java应用程序的ClassPath中加载具体资源并进行封装，可以使
用指定的类加载器(ClassLoader)或者给定的类进行资源加载。
􏰀 
FileSystemResource。对java.io.File类型的封装，所以，我们可以以文件或者URL的形式对该类型资源进行访问，只要能跟File打的交道，基本上跟FileSystemResource也可以。
􏰀
UrlResource。通过java.net.URL进行的具体资源查找定位的实现类，内部委派URL进行具
体的资源操作。
􏰀	
InputStreamResource。将给定的InputStream视为一种资源的Resource实现类，较为少用。如果以上这些资源实现还不能满足要求，那么我们还可以根据相应场景给出自己的实现，只需实 12
可能的情况下，以ByteArrayResource以及其他形式资源实现代之。
```
> - **国际化信息支持**：对于Java中的国际化信息处理，主要涉及两个类，即java.util.Locale和java.util.ResourceBundle
> 	- **Locale**：不同的Locale代表不同的国家和地区，每个国家和地区在Locale这里都有相应的简写代码表示，包括语言代码以及国家代码，这些代码是ISO标准代码。
> 	- **ResourceBundle**：ResourceBundle用来保存特定于某个Locale的信息(可以是String类型信息，也可以是任何类型 的对象)。
> 	- **MessageSource**：在JavaSE的国际化基础上进一步抽象了MessageSource接口
> 
```
public interface MessageSource {
	String getMessage(String code, Object[] args, String defaultMessage, Locale locale);
	String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException;
	String getMessage(MessageSourceResolvable resolvable, Locale locale) throws ➥ NoSuchMessage zException;
}
```
> - **容器内部事件发布**
> 	- **ApplicationEvent**：Spring容器自定义事件类型，继承自 java.util.EventObject，ContextClosedEvent,ContextRefreshedEvent、RequestHandlerEvent
> 	- **ApplicationListener**：Spring容器内使用的自定义事件监听器接口，继承自java.util.EventListener ，ApplicationContext容器在启动时，会自动识别并加载 EventListener类型bean定义，一旦容器内有事件发布，将通知这些注册到容器的EventListener
> 	- **ApplicationContext**：ApplicationContext 继承了ApplicationEventPublisher 接口，所以ApplicationContext就是担当的就是事件发布者的角色
> 	- **ApplicationEventMuticaster**：实现了监听器的管理功能


### 3.小结BeanFactory与ApplicationContext有什么不同？
- ApplicationContext包含BeanFactory所有功能，并拥有其他高级特性
- BeanFactory是默认lazy-load，ApplicationContext默认是全部初始化并绑定

## 总结
以上就是Spring IoC容器的大致介绍，最后，感谢书籍《Spring揭秘》，通过此书对Spring有一定的了解。