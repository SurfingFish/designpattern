#        [ApplicationContext 相关接口架构分析]

在使用Spring的时候，我们经常需要先得到一个ApplicationContext对象，然后从该对象中获取我们配置的Bean对象。ApplicationContext隶属于org.springframework.context，是SpringFramework中Bean的管理者，为SpringFramework的诸多功能提供支撑作用。

下图是Spring-4.3.2.RELEASE版本中ApplicationContext相关的UML类视图（浅绿色的为接口，浅黄色的为类）：

[![SpringApplicationContext](https://images2015.cnblogs.com/blog/947629/201608/947629-20160827175220538-489562952.png)](http://images2015.cnblogs.com/blog/947629/201608/947629-20160827175218382-1804880151.png)

## BeanFactory系列接口：

```
public interface BeanFactory
```

BeanFactory 是 Spring 管理 Bean 的最顶层接口，是一个 Bean 容器, 管理一系列的 bean，每一个 bean 使用一个String 类型的 name(或称之为id) 来唯一确定，这些 Bean 可以是 prototype 的或者 singleton的 。Spring 提倡使用依赖注入(Dependency Injection) 的方式装配 Bean。BeanFactory从“configuration source”加载Bean的定义，configuration source 可以是xml文件或者properties文件甚至是数据库。

```
public interface HierarchicalBeanFactory extends BeanFactory
```

BeanFactory的子接口HierarchicalBeanFactory是一个具有层级关系的Bean 工厂，拥有属性parentBeanFactory。当获取 Bean对象时，如果当前BeanFactory中不存在对应的bean，则会访问其直接 parentBeanFactory 以尝试获取bean 对象。此外，还可以在当前的 BeanFactory 中 *override* 父级BeanFactory的同名bean。

```
public interface ListableBeanFactory extends BeanFactory
```

ListableBeanFactory 继承了BeanFactory，实现了枚举方法可以列举出当前BeanFactory中所有的bean对象而不必根据name一个一个的获取。 如果 ListableBeanFactory 对象还是一个HierarchicalBeanFactory则getBeanDefinitionNames()方法只会返回当前BeanFactory中的Bean对象而不会去父级BeanFactory中查询。

## 其他接口：

```
public interface PropertyResolver
```

配置文件解析器的最顶级接口，解析配置文件获取属性值等作用

```
public interface Environment extends PropertyResolver
```

提供当前Application运行的所需环境

```
public interface EnvironmentCapable
```

这是一个Environment Holder，只有一个方法：Environment getEnvironment() 用来获取Environment对象

```
public interface ApplicationEventPublisher
```

该接口的功能是publish Event，向事件监听器（Listener）发送事件消息

```
public interface MessageSource
```

解析message的策略接口，方法形如: String getMessage(String, Object[], String, Locale)，用于支撑国际化等功能

```
public interface ResourcePatternResolver extends ResourceLoader
```

其中ResourceLoader用于从一个源（如InputStream等）加载资源文件，ResourcePatternResolver 是ResourceLoader的子类，根据 path-pattern 加载资源。

## ApplicationContext接口：

```
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```

重点来了，ApplicationContext接口继承众多接口，集众多接口功能与一身，为Spring的运行提供基本的功能支撑。**根据程序设计的“单一职责原则”，其实每个较顶层接口都是“单一职责的”，只提供某一方面的功能，而ApplicationContext接口继承了众多接口，相当于拥有了众多接口的功能**，下面看看它的主要功能：

- 首先，它是个BeanFactory，可以管理、装配bean，可以有父级BeanFactory实现Bean的层级管理（具体到这里来说它可以有父级的ApplicationContext，因为ApplicationContext本身就是一个BeanFactory。这在web项目中很有用，可以使每个Servlet具有其独立的context, 所有Servlet共享一个父级的context），它还是Listable的，可以枚举出所管理的bean对象。
- 其次，它是一个ResourceLoader，可以加载资源文件；
- 再次，它可以管理一些Message实现国际化等功能；
- 还有，它可以发布事件给注册的Listener，实现监听机制。

## ApplicationContext 的子接口：

ApplicationContext 接口具有两个直接子接口，分别是：

```
org.springframework.context.ConfigurableApplicationContext 
org.springframework.web.context.WebApplicationContext
```

分别看这两个子接口:

```
public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable
```

根据接口名可以判决，该接口是可配置的！ApplicationContext 接口本身是 read-only 的，所以子接口 ConfigurableApplicationContext 就提供了如setID()、setParent()、setEnvironment()等方法，用来配置ApplicationContext。

再看其继承的另外两个接口：

- Lifecycle：Lifecycle接口中具有start()、stop()等方法，用于对context生命周期的管理；
- Closeable：Closeable是标准JDK所提供的一个接口，用于最后关闭组件释放资源等；

```
public interface WebApplicationContext extends ApplicationContext
```

该接口仅仅在原接口基础上提供了getServletContext()，用于给servlet提供上下文信息。

```
public interface ConfigurableWebApplicationContext extends WebApplicationContext, ConfigurableApplicationContext
```

这里 ConfigurableWebApplicationContext 又将上述两个接口结合起来，提供了一个可配置、可管理、可关闭的WebApplicationContext，同时该接口还增加了setServletContext()，setServletConfig()等set方法，用于装配WebApplicationContext。

**到这里ApplicationContext相关接口基本上已经讲完了，总结起来就两大接口：**

```
org.springframework.context.ConfigurableApplicationContext 
org.springframework.web.context.ConfigurableWebApplicationContext
```

对于普通应用，使用ConfigurableApplicationContext 接口的实现类作为bean的管理者，对于web应用，使用ConfigurableWebApplicationContext 接口的实现类作为bean的管理者。**这两个接口，从结构上讲他们是继承关系，从应用上讲他们是平级关系**，在不同的领域为Spring提供强大的支撑。  

## ApplicationContext相关实现类设计：

Spring是一个优秀的框架，具有良好的结构设计和接口抽象，它的每一个接口都是其功能具体到各个模块中的高度抽象，实际使用过程中相当于把接口的各个实现类按照接口所提供的组织架构装配起来以提供完整的服务，可以说掌握了Spring的接口就相当于掌握了Spring的大部分功能。

ApplicationContext 的实现类众多，然而

上文中分析了 ApplicationContext 接口的各个功能，下面将分析 ApplicationContext 的实现类对上述接口的各个功能都是怎样实现的（PS. 限于篇幅，这里仅仅指出上述各个功能在实现类中什么位置通过什么方法实现，至于其具体实现过程，每一个功能拿出来都可以单独写一篇文章了，这里不进行详述）。至于实现类又扩展了其他接口或者继承了其他父类，这些只是实现类为了扩展功能或者为了对实现上述接口提供便利而做的事情，对ApplicationContext接口抽象出来的功能没有影响或者没有太大帮助，因此略去。

以ClassPathXmlApplicationContext为例，其主要继承关系如下：

```
org.springframework.context.support.AbstractApplicationContext 
      org.springframework.context.support.AbstractRefreshableApplicationContext 
            org.springframework.context.support.AbstractRefreshableConfigApplicationContext 
                  org.springframework.context.support.AbstractXmlApplicationContext 
                        org.springframework.context.support.ClassPathXmlApplicationContext
```

而最顶层抽象类 AbstractApplicationContext 又实现了 ConfigurableApplicationContext 接口。

```
AbstractApplicationContext extends DefaultResourceLoader    implements ConfigurableApplicationContext, DisposableBean
```

根据上文所述，这里略去其父类 DefaultResourceLoader 和接口 DisposableBean ，只关注接口 ConfigurableApplicationContext，回忆一下该的主要功能：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Configable, //可配置（该接口本身扩展的功能）
Lifecycle, //生命周期可管理 
Closeable，//可关闭（释放资源） 
EnvironmentCapable，//可配置Environment 
MessageSource, //可管理message实现国际化等功能
ApplicationEventPublisher, //可publish事件，调用Listener 
ResourcePatternResolver，//加载pattern指定的资源 
ListableBeanFactory, HierarchicalBeanFactory,//管理Bean的生命周期，这个最重要，放最后说
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

然后回到最顶层抽象类 AbstractApplicationContext ，该抽象类可以说是 ClassPathXmlApplicationContext 继承结构中代码量最多的类，上述的大部分功能也都在该类中实现。该类采用**模板方法模式**，实现了上述功能的大部分逻辑，然后又抽出许多 protected方法（或 abstract 方法）供子类override 。

**各功能的实现列举如下：**

**Configable：**如setParent/setEnvironment等方法，由AbstractApplicationContext实现 。代码示例：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
    public void setParent(ApplicationContext parent) {
        this.parent = parent;
        if (parent != null) {
            Environment parentEnvironment = parent.getEnvironment();
            if (parentEnvironment instanceof ConfigurableEnvironment) {
                getEnvironment().merge((ConfigurableEnvironment) parentEnvironment);
            }
        }
    }
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**Lifecycle：**AbstractApplicationContext拥有一个LifecycleProcessor实例，用于管理自身的生命周期，AbstractApplicationContext的Lifecycle方法如start、stop等由会代理给LifecycleProcessor进行处理，代码示例：

```
@Override
    public void start() {
        getLifecycleProcessor().start();
        publishEvent(new ContextStartedEvent(this));
    }
```

**Closeable：**由AbstractApplicationContext实现，用于关闭ApplicationContext销毁所有beans，此外如果注册有JVM shutdown hook，同样要将其移除 。代码示例：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
    public void close() {
        synchronized (this.startupShutdownMonitor) {
            doClose();            
            if (this.shutdownHook != null) {
                try {
                    Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
                }catch (IllegalStateException ex) {
                    // ignore - VM is already shutting down
                }
            }
        }
    }
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**EnvironmentCapable：**由AbstractApplicationContext实现，其持有一个ConfigurableEnvironment实例，用来实现EnvironmentCapable接口的getEnvironment方法 。代码示例：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
    public ConfigurableEnvironment getEnvironment() {
        if (this.environment == null) {
            this.environment = createEnvironment();
        }
        return this.environment;
    }
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**MessageSource：**AbstractApplicationContext持有一个MessageSource实例，将MessageSource接口的方法代理给该实例来完成 。代码示例：

```
@Override
    public String getMessage(String code, Object args[], String defaultMessage, Locale locale) {
        return getMessageSource().getMessage(code, args, defaultMessage, locale);
    }
```

**ApplicationEventPublisher：**由AbstractApplicationContext实现，如果存在父级ApplicationContext，则同样要将event发布给父级ApplicationContext。代码示例：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Override
    public void publishEvent(ApplicationEvent event) {
        Assert.notNull(event, "Event must not be null");
        if (logger.isTraceEnabled()) {
            logger.trace("Publishing event in " + getDisplayName() + ": " + event);
        }
        getApplicationEventMulticaster().multicastEvent(event);
        if (this.parent != null) {
            this.parent.publishEvent(event);
        }
    }
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**ResourcePatternResolver：**AbstractApplicationContext持有一个ResourcePatternResolver实例，该接口的方法代理给该实例完成 。代码示例：

```
@Override
    public Resource[] getResources(String locationPattern) throws IOException {
        return this.resourcePatternResolver.getResources(locationPattern);
    }
```

**ListableBeanFactory, HierarchicalBeanFactory：** AbstractApplicationContext 间接实现了这两个接口，然而却并没有实现任何BeanFactory的任何功能。AbstractApplicationContext 拥有一个 ConfigurableListableBeanFactory实例，所有BeanFactory的功能都代理给该实例完成。代码示例：

```
@Override
    public Object getBean(String name) throws BeansException {
        assertBeanFactoryActive();
        return getBeanFactory().getBean(name);
    }
```

而AbstractApplicationContext 的 getBeanFactory() 方法是一个抽象方法，即由子类来提供这个BeanFactory。代码示例：

```
@Override
public abstract ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
```

在 ClassPathXmlApplicationContext 的继承体系中，类AbstractRefreshableApplicationContext实现了这个 getBeanFactory()方法。这里getBeanFactory()方法会创建一个DefaultListableBeanFactory实例作为返回值。

## 小结

本文以 Spring Framework 的 ApplicationContext 为中心，分析了 ApplicationContext 的机构体系和功能实现。接口 ApplicationContext 继承了众多接口，可以满足应用中“面向接口编程”的常用功能需求。抽象类 AbstractApplicationContext 实现了ApplicationContext 接口中简单不易变动的部分，然后通过“组合”将众多“容易变动”功能代理给它的一些成员变量来实现，最后再使用模板方法模式让子类为父类提供一些函数的支持或者设置替换父类的上述成员变量，从而实现了“对扩展开放，对修改封闭”的设计原则，为Spring Framework 提供了灵活性大可扩展性强的架构支撑。