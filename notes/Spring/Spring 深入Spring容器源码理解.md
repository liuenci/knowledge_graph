#### 来源
[深入Spring源码系列（二）——深入Spring容器，通过源码阅读和时序图来彻底弄懂Spring容器（上）](https://github.com/coderbruis/JavaSourceCodeLearning/blob/master/note/Spring/%E6%B7%B1%E5%85%A5Spring%E6%BA%90%E7%A0%81%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94%E6%B7%B1%E5%85%A5Spring%E5%AE%B9%E5%99%A8%EF%BC%8C%E9%80%9A%E8%BF%87%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E5%92%8C%E6%97%B6%E5%BA%8F%E5%9B%BE%E6%9D%A5%E5%BD%BB%E5%BA%95%E5%BC%84%E6%87%82Spring%E5%AE%B9%E5%99%A8%EF%BC%88%E4%B8%8A%EF%BC%89.md)
​

#### Spring 容器
Spring 容器就相当于一个大的水桶，里面装着很多水-bean 对象。bean 对象就是一个普通的 pojo 对象。这里有一个很重要的概念，就是 IOC-Invertion of Control，即控制翻转。通俗点就是将创建并且绑定数据 bean 的权利赋予了 Spring 容器（或者 Spring IOC 容器），在 bean 生成或者初始化的时候，Spring 就会将数据注入到 bean 中，或者是将对象的引用注入对象数据域中的方式来注入对方法调用的依赖。
​

#### 如何通俗理解 Spring IOC 控制翻转
参考文章：[控制反转和依赖注入的理解(通俗易懂)](https://segmentfault.com/a/1190000022439622)
IOC 意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。
​

接下来明确“谁控制谁，控制什么，为何是反转，正转是什么”
谁控制谁，控制什么：Spring 容器控制事先设计好的 Bean 对象，控制 Bean 对象的创建以及内部属性数据的获取。
为何是反转，正转是什么：正转是由程序主动去控制并且获取依赖对象的方式叫做正转，反转则是由 Spring 容器帮忙创建并且初始化注入依赖对象，反转的其实是依赖对象的获取叫做反转。
​

#### BeanFactory And ApplicationContext
在 Spring 容器的设计中，有两个主要的容器系列，一个是实现 BeanFactory 接口的简单容器系列，这个接口实现了容器最基本的功能。另一个是 ApplicationContext 应用上下文，作为容器的高级形态而存在，他用于扩展 BeanFactory 中现有的功能。ApplicationContext 和 BeanFactory 两者都是用来加载 Bean 的，但是相比之下，ApplicationContext 提供了更多的扩展功能，简单一点说：ApplicationContext 包含 BeanFactory 的所有功能。
​

两者读取 XML 配置文件的方式类似：
```java
BeanFactory bf = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
ApplicationContext bf = new ClassPathXmlApplicationContext("applicationContext.xml");
```
#### DefaultListableBeanFactory
DefaultListableBeanFactory 是整个 bean 加载的核心部分，是 Spring 注册及加载 bean 的默认实现。


#### XmlBeanDefinitionReader
XML 配置文件的读取是 Spring 中最重要的功能，因为 Spring 的大部分功能都是以配置作为切入点的，XmlBeanDefinitionReader 实现了对资源文件的读取、解析以及注册。
​

#### 通过 xml 配置文件初始化容器
applicationContext.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.bruis.learnspring.context.Person">
        <property name="age" value="23"/>
        <property name="name" value="Bruis"/>
    </bean>
    
</beans>
```
测试类
```java
public class SpringMain {
    public static void main(String[] args) {
        // 使用spring容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = (Person)context.getBean("person");
        System.out.println(person);
    }
}

运行结果：
Person{name='Bruis', age=23}
```
断点调试

1. SpringMain.class
```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
```

2. ClassPathXmlApplicationContext.class
```java
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
    this(new String[]{configLocation}, true, (ApplicationContext)null);
}
/*
* 使用给定的父类容器创建新的ClassPathXmlApplicationContext，然后从给定的XML文件加载定义，加载所有bean定义并且创建所有的单例，在进一步配置上下文后调用refresh。换句话说xml文件的读取，bean的创建和实例化都是在refresh()方法中进行的，refresh()函数中包含了几乎所有的ApplicationContext中提供的全部功能。
*/
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
    super(parent);
    // 设置配置路径
    this.setConfigLocations(configLocations);
    if (refresh) {
        //refresh Spring容器
        this.refresh();
    }
}
```

3. AbstractRefreshableConfigApplicationContext.class
```java
// 给configLocations字符串数组设置值，支持多个配置文件已数组方式同时传入。
public void setConfigLocations(String... locations) {
    if (locations != null) {
        Assert.noNullElements(locations, "Config locations must not benull");
        this.configLocations = new String[locations.length];
        for(int i = 0; i < locations.length; ++i) {
            this.configLocations[i] = this.resolvePath(locations[i]).trim();
        }
    } else {
        this.configLocations = null;
    }
}
```
#### 一些问题
Spring 容器的生命周期分为几个阶段？
TODO
​

Spring 容器的初始化发生在什么时候？发生了什么？

- 容器的初始化发生在 refresh 时候。
- 发生了以下几个过程
   - Resource 定位过程。
   - BeanDefinition 载入。
   - IOC 容器注入 BeanDefinition 的过程

​

Spring 容器的销毁过程又发生了什么？
​

Spring 容器什么时候读取 xml 配置文件？并且把什么配置文件读取？

- 在刷新上下文环境之后就会读取 xml 配置文件
- 读取配置文件主要是定位资源，加载 bean ，最后注册 bean

​

Spring 容器用什么数据结构存储用于创建 bean 的 K/V 信息？

- ConcurrentHashMap 

​

Spring 容器获取了用于创建 bean 的 K/V 信息后，在什么时候去创建并且初始化 bean？

-  初始化 BeanFactory 之后，并且刷新了上下文，填充 bean 的所有属性，并且注册到 IOC 容器之后就会去初始化 bean。
- 对 bean 赋值填充是在 AbstractAutowireCapableBeanFactory.class 类里面 applyPropertyValues 方法里 的，并且是通过对原属性值进行了一次深拷贝，然后将生拷贝的属性填充在 bean 里面的。
#### refresh 过程
![68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f6170692f706572736f6e616c2f66696c652f37364145384645444146463534423638383143333336423035364143354230413f6d6574686f643d646f776e6c6f61642673686172654b65793d343330663532363331.jpg](https://cdn.nlark.com/yuque/0/2022/jpeg/493161/1645944017562-fc3e2397-9da6-44c0-89bb-f6c7631d5b2b.jpeg#clientId=u4846624a-48f1-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u1085d6ae&margin=%5Bobject%20Object%5D&name=68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f6170692f706572736f6e616c2f66696c652f37364145384645444146463534423638383143333336423035364143354230413f6d6574686f643d646f776e6c6f61642673686172654b65793d343330663532363331.jpg&originHeight=387&originWidth=1119&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38826&status=done&style=none&taskId=ubb0eee6b-8347-4468-875b-ba782d3213f&title=)

1. AbstractApplicationContext.class
```java
/*
    简单来说，Spring容器的初始化时右refresh()方法来启动的，这个方法标志着IOC容器的正式启动。具体来说，这里的启动包括了BeanDefinition和Resource的定位、载入和注册三个基本过程。
*/
@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		//准备刷新容器（上下文环境）
		prepareRefresh();
		//通知子类刷新内部bean工厂，初始化BeanFactory并进行XML文件的解析、读取
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
		//准备bean工厂在此上下文中使用，并对BeanFactory进行各种功能填充
		prepareBeanFactory(beanFactory);
		try {
			// 允许在上下文子类中对bean工厂进行后处理
			postProcessBeanFactory(beanFactory);
			// 在上下文中调用注册并激活BeanFactory的处理器，就是这个方法中的处理器注册了bean
			invokeBeanFactoryPostProcessors(beanFactory);
			// 注册拦截bean创建的bean处理器，这里只是注册，真正的调用Bean是在getBean方法的时候
			registerBeanPostProcessors(beanFactory);
			// 初始化此上下文的消息源，及不同语言的消息体，例如国际化处理
			initMessageSource();
			// 初始化此上下文的事件多播器
			initApplicationEventMulticaster();
			// 在特定上下文子类中初始化其他特殊bean.
			onRefresh();
			// 检查监听器bean并注册它们
			registerListeners();
			// 实例化所有剩余（非惰性的）单例
			finishBeanFactoryInitialization(beanFactory);
			// 完成刷新过程，通知生命周期处理器lifecycleProcessor刷新过程，同时发出ContextRefreshEvent通知别人
			finishRefresh();
		}
		catch (BeansException ex) {
		    ...
			// 摧毁已经创建的单例以避免资源浪费
			destroyBeans();
			// 重置'有效'标志
			cancelRefresh(ex);
			// Propagate exception to caller.
			throw ex;
		}
		finally {
			resetCommonCaches();
		}
	}
}
```

2. AbstractRefreshableApplicationContext.class
```java
/*
    通知子类刷新内部bean工厂，初始化BeanFactory并进行XML文件的解析、读取。obtain就是指获得的含义，这个方法obtaiinFreshBeanFactory正是实现BeanFactory的地方，也就是经过这个方法，ApplicationContext就已经拥有了BeanFactory的全部功能（也就是BeanFactory包含在了Spring容器里了）。
*/
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    this.refreshBeanFactory();
    ...
    return beanFactory;
}
/*
此实现执行此上下文的基础bean工厂的实际刷新，关闭先前的bean工厂（如果有）并初始化上一个生命周期的下一阶段的新bean工厂。

经过debug，最后从refreshBeanFactory()方法返回后，this也就是AbstractRefreshableApplicationContext实例对象

*/
@Override
protected final void refreshBeanFactory() throws BeansException {
    //如果有bean工厂，则关闭该工厂
	if (hasBeanFactory()) {
		destroyBeans();
		closeBeanFactory();
	}
	try {
	    //创建一个新bean工厂，这里的DefaultListableBeanFactory就是前面笔者将的Spring核心类，这个类真的很重要！
		DefaultListableBeanFactory beanFactory = createBeanFactory();
		//为了序列化指定ID，如果需要的话，让这个BeanFactory从ID反序列化掉BeanFactory对象
		beanFactory.setSerializationId(getId());
		//定制beanFactory，设置相关属性，包括是否允许覆盖同名称的不同定义的对象以及循环依赖以及设置@Autowired和@Qualifier注解解析器QualifierAnnotationAutowireCandidateResolver
		customizeBeanFactory(beanFactory);
		//加载bean定义信息，这一步实际上就从XML配置文件里的bean信息给读取到了Factory里了。
		loadBeanDefinitions(beanFactory);
		synchronized (this.beanFactoryMonitor) {
			this.beanFactory = beanFactory;
		}
	}
	catch (IOException ex) {
	    ...
	}	
}
```
这里先看看上面代码的 loadBeanDefinitions() 方法运行完后的结果
![68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f6170692f706572736f6e616c2f66696c652f35394642434433434331423534313336413035333039454136423838464542333f6d6574686f643d646f776e6c6f61642673686172654b65793d3830.png](https://cdn.nlark.com/yuque/0/2022/png/493161/1645945461370-63dfd614-86c7-4ea2-86f0-940ff5f1290d.png#clientId=u4846624a-48f1-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u34ea2fa6&margin=%5Bobject%20Object%5D&name=68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f6170692f706572736f6e616c2f66696c652f35394642434433434331423534313336413035333039454136423838464542333f6d6574686f643d646f776e6c6f61642673686172654b65793d3830.png&originHeight=693&originWidth=1146&originalType=binary&ratio=1&rotation=0&showTitle=false&size=100080&status=done&style=none&taskId=u3dfc214b-f0d6-4932-8876-64dff8a11d0&title=)
![68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f6170692f706572736f6e616c2f66696c652f45323538393037383532323834413646393341324333303533313945424236343f6d6574686f643d646f776e6c6f61642673686172654b65793d376531646261393664.png](https://cdn.nlark.com/yuque/0/2022/png/493161/1645945506613-52538744-7109-4a30-ade8-a8bcdf6b6d61.png#clientId=u4846624a-48f1-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u61ac5608&margin=%5Bobject%20Object%5D&name=68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f6170692f706572736f6e616c2f66696c652f45323538393037383532323834413646393341324333303533313945424236343f6d6574686f643d646f776e6c6f61642673686172654b65793d376531646261393664.png&originHeight=696&originWidth=1099&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81237&status=done&style=none&taskId=udbb99ae8-d34c-4450-acc8-bdbd17a9022&title=)
从图中可以看到，loadBeanDefintions() 方法运行完之后，在 beanFactory 变量中存放这一个 ConcurrentHashMap 变量，用于存放着 person 这个键值对，key 为 person， Value 为一个 ArrayList 的变量，里面存放着 person 的两个属性：age 和 name
