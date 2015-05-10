---
layout: post
category : architecture
tagline: 
tags : [Spring, architecture, design]
excerpt : 
title_cn: Spring框架的设计理念与设计模式分析
---
{% include JB/setup %}

<code>Spring</code> 作为现在最优秀的框架之一，已被广泛的使用，并且有很多对其分析的文章。本文将从另外一个视角试图剖析出<code>Spring</code>框架的作者设计 <code>Spring</code> 框架的骨骼架构的设计理念，有那几个核心组件？为什么需要这些组件？它们又是如何结合在一起构成<code>Spring</code>的骨骼架构？<code>Spring</code>的AOP特性又是如何利用这些基础的骨骼架构来工作的？<code>Spring</code>中又使用了那些设计模式来完成它的这种设计的？它的这种设计理念对对我们以后的软件设计有何启示？本文将详细解答这些问题。

## **一、Spring的骨骼架构**

<code>Spring</code>总共有十几个组件，但是真正核心的组件只有几个，下面是<code>Spring</code>框架的总体架构图：  

图 1 .<code>Spring</code> 框架的总体架构图  
<img src="/assets/images/article_imgs/architecture/2015/05/09/1.gif" alt="Spring框架的总体架构图" align="center"/>  

从上图中可以看出<code>Spring</code>框架中的核心组件只有三个：<code>Core</code>、<code>Context</code>和<code>Beans</code>。它们构建起了整个<code>Spring</code> 的骨骼架构。没有它们就不可能有<code>AOP</code>、<code>Web</code>等上层的特性功能。下面也将主要从这三个组件入手分析<code>Spring</code>。

## **二、Spring 的设计理念**

前面介绍了<code>Spring</code>的三个核心组件，如果再在它们三个中选出核心的话，那就非<code>Beans</code>组件莫属了，为何这样说，其实<code>Spring</code>就是面向<code>Bean</code>的编程（BOP,Bean Oriented Programming），Bean在<code>Spring</code>中才是真正的主角。  

<code>Bean</code>在<code>Spring</code>中作用就像<code>Object</code>对<code>OOP</code>的意义一样，没有对象的概念就像没有面向对象编程，<code>Spring</code>中没有<code>Bean</code>也就没有<code>Spring</code>存在的意义。就像一次演出舞台都准备好了但是却没有演员一样。为什么要<code>Bean</code>这种角色<code>Bean</code>或者为何在 <code>Spring</code> 如此重要，这由<code>Spring</code> 框架的设计目标决定，<code>Spring</code>为何如此流行，我们用<code>Spring</code> 的原因是什么，想想你会发现原来<code>Spring</code> 解决了一个非常关键的问题他可以让你把对象之间的依赖关系转而用配置文件来管理，也就是他的依赖注入机制。而这个注入关系在一个叫 <code>Ioc</code> 容器中管理，那 <code>Ioc</code> 容器中有又是什么就是被 <code>Bean</code> 包裹的对象。<code>Spring</code> 正是通过把对象包装在 Bean 中而达到对这些对象管理以及一些列额外操作的目的。
它这种设计策略完全类似于 <code>Java</code> 实现 <code>OOP</code> 的设计理念，当然了 <code>Java</code> 本身的设计要比 <code>Spring</code> 复杂太多太多，但是都是构建一个数据结构，然后根据这个数据结构设计他的生存环境，并让它在这个环境中按照一定的规律在不停的运动，在它们的不停运动中设计一系列与环境或者与其他个体完成信息交换。这样想来回过头想想我们用到的其他框架都是大慨类似的设计理念。

## **三、核心组件如何协同工作**

前面说 <code>Bean</code> 是 <code>Spring</code> 中关键因素，那 <code>Context</code> 和 <code>Core</code> 又有何作用呢？前面把 <code>Bean</code>比作一场演出中的演员的话，那 <code>Context</code> 就是这场演出的舞台背景，而 <code>Core</code> 应该就是演出的道具了。只有他们在一起才能具备能演出一场好戏的最基本的条件。当然有最基本的条件还不能使这场演出脱颖而出，还要他表演的节目足够的精彩，这些节目就是 <code>Spring</code> 能提供的特色功能了。

我们知道 <code>Bean</code> 包装的是 <code>Object</code>，而 <code>Object</code>必然有数据，如何给这些数据提供生存环境就是<code>Context</code>要解决的问题，对<code>Context</code>来说他就是要发现每个<code>Bean</code>之间的关系，为它们建立这种关系并且要维护好这种关系。所以<code>Context</code>就是一个 <code>Bean</code> 关系的集合，这个关系集合又叫 <code>Ioc</code>容器，一旦建立起这个 <code>Ioc</code> 容器后 <code>Spring</code> 就可以为你工作了。那 <code>Core</code> 组件又有什么用武之地呢？其实 <code>Core</code> 就是发现、建立和维护每个 <code>Bean</code> 之间的关系所需要的一些列的工具，从这个角度看来，<code>Core</code> 这个组件叫 <code>Util</code> 更能让你理解。  

它们之间可以用下图来表示：

图 2. 三个组件关系  
<img src="/assets/images/article_imgs/architecture/2015/05/09/2.gif" alt="三个组件关系" align="center"/>  

### **1、核心组件详解**

这里将详细介绍每个组件内部类的层次关系，以及它们在运行时的时序顺序。我们在使用 <code>Spring</code> 是应该注意的地方。

#### **1.1 Bean 组件**

前面已经说明了 <code>Bean</code> 组件对 <code>Spring</code> 的重要性，下面看看 <code>Bean</code> 这个组件式怎么设计的。<code>Bean</code> 组件在 <code>Spring</code> 的 <code>org.springframework.beans</code> 包下。这个包下的所有类主要解决了三件事：<code>Bean</code> 的定义、<code>Bean</code> 的创建以及对 <code>Bean</code> 的解析。对 <code>Spring</code> 的使用者来说唯一需要关心的就是 <code>Bean</code> 的创建，其他两个由 <code>Spring</code> 在内部帮你完成了，对你来说是透明的。
<code>Spring Bean</code> 的创建是典型的工厂模式，他的顶级接口是 <code>BeanFactory</code>，下图是这个工厂的继承层次关系：

图 3. <code>Bean</code> 工厂的继承关系

<img src="/assets/images/article_imgs/architecture/2015/05/09/3.png" alt="Bean 工厂的继承关系" align="center"/>  

<code>BeanFactory</code> 有三个子类：<code>ListableBeanFactory</code>、<code>HierarchicalBeanFactory</code> 和 <code>AutowireCapableBeanFactory</code>。但是从上图中我们可以发现最终的默认实现类是 <code>DefaultListableBeanFactory</code>，他实现了所有的接口。那为何要定义这么多层次的接口呢？查阅这些接口的源码和说明发现，每个接口都有他使用的场合，它主要是为了区分在 <code>Spring</code> 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制。例如 <code>ListableBeanFactory</code> 接口表示这些 <code>Bean</code> 是可列表的，而 <code>HierarchicalBeanFactory</code> 表示的是这些 <code>Bean</code> 是有继承关系的，也就是每个 <code>Bean</code> 有可能有父 <code>Bean</code>。<code>AutowireCapableBeanFactory</code> 接口定义 <code>Bean</code> 的自动装配规则。这四个接口共同定义了 <code>Bean</code> 的集合、<code>Bean</code> 之间的关系、以及 <code>Bean</code> 行为。

<code>Bean</code> 的定义主要有 <code>BeanDefinition</code> 描述，如下图说明了这些类的层次关系：

图 4. <code>Bean</code> 定义的类层次关系图

<img src="/assets/images/article_imgs/architecture/2015/05/09/4.png" alt="Bean 定义的类层次关系图" align="center"/>  

<code>Bean</code> 的定义就是完整的描述了在 <code>Spring</code> 的配置文件中你定义的 <code><bean/></code> 节点中所有的信息，包括各种子节点。当 <code>Spring</code> 成功解析你定义的一个 <code><bean/></code> 节点后，在 <code>Spring</code> 的内部他就被转化成 <code>BeanDefinition</code> 对象。以后所有的操作都是对这个对象完成的。

<code>Bean</code> 的解析过程非常复杂，功能被分的很细，因为这里需要被扩展的地方很多，必须保证有足够的灵活性，以应对可能的变化。<code>Bean</code> 的解析主要就是对 <code>Spring</code> 配置文件的解析。这个解析过程主要通过下图中的类完成：

图 5. <code>Bean</code> 的解析类

<img src="/assets/images/article_imgs/architecture/2015/05/09/5.png" alt="Bean的解析类" align="center"/>

当然还有具体对 <code>tag</code> 的解析这里并没有列出。

#### **1.2 Context 组件**

<code>Context</code> 在 <code>Spring</code> 的 <code>org.springframework.context</code> 包下，前面已经讲解了 <code>Context</code> 组件在 <code>Spring</code> 中的作用，他实际上就是给 <code>Spring</code> 提供一个运行时的环境，用以保存各个对象的状态。下面看一下这个环境是如何构建的。
<code>ApplicationContext</code> 是 <code>Context</code> 的顶级父类，他除了能标识一个应用环境的基本信息外，他还继承了五个接口，这五个接口主要是扩展了 <code>Context</code> 的功能。下面是 <code>Context</code> 的类结构图：

图 6. <code>Context</code> 相关的类结构图

<img src="/assets/images/article_imgs/architecture/2015/05/09/6.png" alt="Context 相关的类结构图" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/6.png" target="_blank">查看大图</a></small>

从上图中可以看出 <code>ApplicationContext</code> 继承了 <code>BeanFactory</code>，这也说明了 <code>Spring</code> 容器中运行的主体对象是 <code>Bean</code>，另外 <code>ApplicationContext</code> 继承了 <code>ResourceLoader</code> 接口，使得 <code>ApplicationContext</code> 可以访问到任何外部资源，这将在 <code>Core</code> 中详细说明。

<code>ApplicationContext</code> 的子类主要包含两个方面：

1.  <code>ConfigurableApplicationContext</code> 表示该 <code>Context</code> 是可修改的，也就是在构建 <code>Context</code> 中用户可以动态添加或修改已有的配置信息，它下面又有多个子类，其中最经常使用的是可更新的 <code>Context</code>，即 <code>AbstractRefreshableApplicationContext</code> 类。

2.  <code>WebApplicationContext</code> 顾名思义，就是为 <code>web</code> 准备的 <code>Context</code> 他可以直接访问到 <code>ServletContext</code>，通常情况下，这个接口使用的少。

再往下分就是按照构建 <code>Context</code> 的文件类型，接着就是访问 <code>Context</code> 的方式。这样一级一级构成了完整的 <code>Context</code> 等级层次。

总体来说 <code>ApplicationContext</code> 必须要完成以下几件事：

1.  标识一个应用环境
2.  利用 BeanFactory 创建 Bean 对象
3.  保存对象关系表
4.  能够捕获各种事件

<code>Context</code> 作为 <code>Spring</code> 的 <code>Ioc</code> 容器，基本上整合了 <code>Spring</code> 的大部分功能，或者说是大部分功能的基础。

#### **1.3 Core 组件**

<code>Core</code> 组件作为 <code>Spring</code> 的核心组件，他其中包含了很多的关键类，其中一个重要组成部分就是定义了资源的访问方式。这种把所有资源都抽象成一个接口的方式很值得在以后的设计中拿来学习。下面就重要看一下这个部分在 <code>Spring</code> 的作用。

下图是 Resource 相关的类结构图：

图 7. Resource 相关的类结构图

<img src="/assets/images/article_imgs/architecture/2015/05/09/7.png" alt="Resource 相关的类结构图" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/7.png" target="_blank">查看大图</a></small>

从上图可以看出 <code>Resource</code> 接口封装了各种可能的资源类型，也就是对使用者来说屏蔽了文件类型的不同。对资源的提供者来说，如何把资源包装起来交给其他人用这也是一个问题，我们看到 <code>Resource</code> 接口继承了 <code>InputStreamSource</code> 接口，这个接口中有个 <code>getInputStream</code> 方法，返回的是 <code>InputStream</code> 类。这样所有的资源都被可以通过 <code>InputStream</code> 这个类来获取，所以也屏蔽了资源的提供者。另外还有一个问题就是加载资源的问题，也就是资源的加载者要统一，从上图中可以看出这个任务是由 <code>ResourceLoader</code> 接口完成，他屏蔽了所有的资源加载者的差异，只需要实现这个接口就可以加载所有的资源，他的默认实现是 <code>DefaultResourceLoader</code>。

下面看一下 <code>Context</code> 和 <code>Resource</code> 是如何建立关系的？首先看一下他们的类关系图：

图 8. <code>Context</code> 和 <code>Resource</code> 的类关系图

<img src="/assets/images/article_imgs/architecture/2015/05/09/8.png" alt="Context和Resource的类关系图" align="center"/>

从上图可以看出，<code>Context</code> 是把资源的加载、解析和描述工作委托给了 <code>ResourcePatternResolver</code> 类来完成，他相当于一个接头人，他把资源的加载、解析和资源的定义整合在一起便于其他组件使用。<code>Core</code> 组件中还有很多类似的方式。

### **2、Ioc 容器如何工作**

前面介绍了 <code>Core</code> 组件、<code>Bean</code> 组件和 <code>Context</code> 组件的结构与相互关系，下面这里从使用者角度看一下他们是如何运行的，以及我们如何让 <code>Spring</code> 完成各种功能，<code>Spring</code> 到底能有那些功能，这些功能是如何得来的，下面介绍。

#### **2.1 如何创建 <code>BeanFactory</code> 工厂**

正如图 2 描述的那样，<code>Ioc</code> 容器实际上就是 <code>Context</code> 组件结合其他两个组件共同构建了一个 <code>Bean</code> 关系网，如何构建这个关系网？构建的入口就在 <code>AbstractApplicationContext</code> 类的 <code>refresh</code> 方法中。这个方法的代码如下：

清单 1. <code>AbstractApplicationContext.refresh</code>

{% highlight java %}
public void refresh() throws BeansException, IllegalStateException { 
    synchronized (this.startupShutdownMonitor) { 
        // Prepare this context for refreshing. 
        prepareRefresh(); 
        // Tell the subclass to refresh the internal bean factory. 
        ConfigurableListableBeanFactory beanFactory = btainFreshBeanFactory(); 
        // Prepare the bean factory for use in this context. 
        prepareBeanFactory(beanFactory); 
        try { 
            // Allows post-processing of the bean factory in context
            subclasses.postProcessBeanFactory(beanFactory);
            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);
            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory); 
            // Initialize message source for this context. 
            initMessageSource(); 
            // Initialize event multicaster for this context. 
            initApplicationEventMulticaster(); 
            // Initialize other special beans in specific context subclasses. 
            onRefresh(); 
            // Check for listener beans and register them. 
            registerListeners(); 
            // Instantiate all remaining (non-lazy-init) singletons. 
            finishBeanFactoryInitialization(beanFactory); 
            // Last step: publish corresponding event. 
            finishRefresh(); 
        } 
        catch (BeansException ex) { 
            // Destroy already created singletons to avoid dangling resources. 
            destroyBeans(); 
            // Reset 'active' flag. 
            cancelRefresh(ex); 
            // Propagate exception to caller. 
            throw ex; 
        } 
    } 
}
{% endhighlight %}

这个方法就是构建整个 <code>Ioc</code> 容器过程的完整的代码，了解了里面的每一行代码基本上就了解大部分 Spring 的原理和功能了。

这段代码主要包含这样几个步骤：

1.  构建 <code>BeanFactory</code>，以便于产生所需的“演员”
2.  注册可能感兴趣的事件
3.  创建 <code>Bean</code> 实例对象
4.  触发被监听的事件

下面就结合代码分析这几个过程。

第二三句就是在创建和配置 <code>BeanFactory</code>。这里是 <code>refresh</code> 也就是刷新配置，前面介绍了 <code>Context</code> 有可更新的子类，这里正是实现这个功能，当 <code>BeanFactory</code> 已存在是就更新，如果没有就新创建。下面是更新 <code>BeanFactory</code> 的方法代码：

清单 2. <code>AbstractRefreshableApplicationContext. refreshBeanFactory</code>

{% highlight java %}
protected final void refreshBeanFactory() throws BeansException { 
    if (hasBeanFactory()) { 
        destroyBeans(); 
        closeBeanFactory(); 
    } 
    try { 
        DefaultListableBeanFactory beanFactory = createBeanFactory(); 
        beanFactory.setSerializationId(getId()); 
        customizeBeanFactory(beanFactory); 
        loadBeanDefinitions(beanFactory); 
        synchronized (this.beanFactoryMonitor) { 
            this.beanFactory = beanFactory; 
        } 
    } 
    catch (IOException ex) { 
        throw new ApplicationContextException(
            "I/O error parsing bean definition source for " 
            + getDisplayName(), ex); 
    } 
}
{% endhighlight %}

这个方法实现了 <code>AbstractApplicationContext</code> 的抽象方法 <code>refreshBeanFactory</code>，这段代码清楚的说明了 <code>BeanFactory</code> 的创建过程。注意 <code>BeanFactory</code> 对象的类型的变化，前面介绍了他有很多子类，在什么情况下使用不同的子类这非常关键。<code>BeanFactory</code> 的原始对象是 <code>DefaultListableBeanFactory</code>，这个非常关键，因为他设计到后面对这个对象的多种操作，下面看一下这个类的继承层次类图：

图 9. <code>DefaultListableBeanFactory</code> 类继承关系图

<img src="/assets/images/article_imgs/architecture/2015/05/09/9.png" alt="DefaultListableBeanFactory 类继承关系图" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/9.png" target="_blank">查看大图</a></small>

从这个图中发现除了 <code>BeanFactory</code> 相关的类外，还发现了与 <code>Bean</code> 的 <code>register</code> 相关。这在 <code>refreshBeanFactory</code> 方法中有一行 <code>loadBeanDefinitions(beanFactory)</code> 将找到答案，这个方法将开始加载、解析 <code>Bean</code> 的定义，也就是把用户定义的数据结构转化为 <code>Ioc</code> 容器中的特定数据结构。

这个过程可以用下面时序图解释：

图 10. 创建 <code>BeanFactory</code> 时序图

<img src="/assets/images/article_imgs/architecture/2015/05/09/10.png" alt="创建BeanFactory 时序图" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/10.png" target="_blank">查看大图</a></small>

<code>Bean</code> 的解析和登记流程时序图如下：

图 11. 解析和登记 <code>Bean</code> 对象时序图

<img src="/assets/images/article_imgs/architecture/2015/05/09/11.png" alt="解析和登记Bean对象时序图" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/11.png" target="_blank">查看大图</a></small>

创建好 <code>BeanFactory</code> 后，接下去添加一些 <code>Spring</code> 本身需要的一些工具类，这个操作在 <code>AbstractApplicationContext</code> 的 <code>prepareBeanFactory</code> 方法完成。

<code>AbstractApplicationContext</code> 中接下来的三行代码对 <code>Spring</code> 的功能扩展性起了至关重要的作用。前两行主要是让你现在可以对已经构建的 <code>BeanFactory</code> 的配置做修改，后面一行就是让你可以对以后再创建 <code>Bean</code> 的实例对象时添加一些自定义的操作。所以他们都是扩展了 <code>Spring</code> 的功能，所以我们要学习使用 <code>Spring</code> 必须对这一部分搞清楚。

其中在 <code>invokeBeanFactoryPostProcessors</code> 方法中主要是获取实现 <code>BeanFactoryPostProcessor</code> 接口的子类。并执行它的 <code>postProcessBeanFactory</code> 方法，这个方法的声明如下：

清单 3. <code>BeanFactoryPostProcessor.postProcessBeanFactory</code>

{% highlight java %}
void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 
    throws BeansException;
{% endhighlight %}

它的参数是 <code>beanFactory</code>，说明可以对 <code>beanFactory</code> 做修改，这里注意这个 <code>beanFactory</code> 是 <code>ConfigurableListableBeanFactory</code> 类型的，这也印证了前面介绍的不同 <code>BeanFactory</code> 所使用的场合不同，这里只能是可配置的<code>BeanFactory</code>，防止一些数据被用户随意修改。

<code>registerBeanPostProcessors</code> 方法也是可以获取用户定义的实现了<code>BeanPostProcessor</code> 接口的子类，并执行把它们注册到 <code>BeanFactory</code> 对象中的 <code>beanPostProcessors</code> 变量中。<code>BeanPostProcessor</code> 中声明了两个方法：<code>postProcessBeforeInitialization</code>、<code>postProcessAfterInitialization</code> 分别用于在 <code>Bean</code> 对象初始化时执行。可以执行用户自定义的操作。

后面的几行代码是初始化监听事件和对系统的其他监听者的注册，监听者必须是 <code>ApplicationListener</code> 的子类。

#### **2.2 如何创建 Bean 实例并构建 Bean 的关系网？**

下面就是 <code>Bean</code> 的实例化代码，是从 <code>finishBeanFactoryInitialization</code> 方法开始的。

清单 4. <code>AbstractApplicationContext.finishBeanFactoryInitialization</code>

{% highlight java %}
protected void finishBeanFactoryInitialization(
    ConfigurableListableBeanFactory beanFactory) { 
    // Stop using the temporary ClassLoader for type matching. 
    beanFactory.setTempClassLoader(null); 
    // Allow for caching all bean definition metadata, not expecting further changes.
    beanFactory.freezeConfiguration(); 
    // Instantiate all remaining (non-lazy-init) singletons. 
    beanFactory.preInstantiateSingletons(); 
}
{% endhighlight %}

从上面代码中可以发现 <code>Bean</code> 的实例化是在 <code>BeanFactory</code> 中发生的。<code>preInstantiateSingletons</code> 方法的代码如下：

清单 5. <code>DefaultListableBeanFactory.preInstantiateSingletons</code>

{% highlight java %}
public void preInstantiateSingletons() throws BeansException { 
    if (this.logger.isInfoEnabled()) { 
        this.logger.info("Pre-instantiating singletons in " + this); 
    } 
    synchronized (this.beanDefinitionMap) { 
        for (String beanName : this.beanDefinitionNames) { 
            RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName); 
            if (!bd.isAbstract() && bd.isSingleton() 
                && !bd.isLazyInit()) {
                if (isFactoryBean(beanName)) { 
                    final FactoryBean factory = 
                        (FactoryBean) getBean(FACTORY_BEAN_PREFIX+ beanName); 
                    boolean isEagerInit; 
                    if (System.getSecurityManager() != null 
                        && factory instanceof SmartFactoryBean) { 
                        isEagerInit = AccessController.doPrivileged(
                            new PrivilegedAction<Boolean>() { 
                            public Boolean run() { 
                                return ((SmartFactoryBean) factory).isEagerInit(); 
                            } 
                        }, getAccessControlContext()); 
                    } 
                    else { 
                        isEagerInit = factory instanceof SmartFactoryBean 
                            && ((SmartFactoryBean) factory).isEagerInit(); 
                    } 
                    if (isEagerInit) { 
                        getBean(beanName); 
                    } 
                } 
                else { 
                    getBean(beanName); 
                } 
            } 
        } 
    } 
}
{% endhighlight %}

这里出现了一个非常重要的 <code>Bean</code> —— <code>FactoryBean</code>，可以说 <code>Spring</code> 一大半的扩展的功能都与这个 <code>Bean</code> 有关，这是个特殊的 <code>Bean</code> 他是个工厂 <code>Bean</code>，可以产生 <code>Bean</code> 的 <code>Bean</code>，这里的产生 <code>Bean</code> 是指 <code>Bean</code> 的实例，如果一个类继承 <code>FactoryBean</code> 用户可以自己定义产生实例对象的方法只要实现他的 <code>getObject</code> 方法。然而在 <code>Spring</code> 内部这个 <code>Bean</code> 的实例对象是 <code>FactoryBean</code>，通过调用这个对象的 <code>getObject</code> 方法就能获取用户自定义产生的对象，从而为 <code>Spring</code> 提供了很好的扩展性。<code>Spring</code> 获取 <code>FactoryBean</code> 本身的对象是在前面加上 <code>&</code> 来完成的。

如何创建 <code>Bean</code> 的实例对象以及如何构建 <code>Bean</code> 实例对象之间的关联关系式 <code>Spring</code> 中的一个核心关键，下面是这个过程的流程图。

图 12.<code>Bean</code> 实例创建流程图

<img src="/assets/images/article_imgs/architecture/2015/05/09/12.gif" alt="Bean实例创建流程图" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/12.gif" target="_blank">查看大图</a></small>

如果是普通的 <code>Bean</code> 就直接创建他的实例，是通过调用 <code>getBean</code> 方法。下面是创建 <code>Bean</code> 实例的时序图：

图 13.<code>Bean</code> 实例创建时序图

<img src="/assets/images/article_imgs/architecture/2015/05/09/13.png" alt="Bean实例创建流程图" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/13.png" target="_blank">查看大图</a></small>

还有一个非常重要的部分就是建立 <code>Bean</code> 对象实例之间的关系，这也是 <code>Spring</code> 框架的核心竞争力，何时、如何建立他们之间的关系请看下面的时序图：

图 14.<code>Bean</code> 对象关系建立

<img src="/assets/images/article_imgs/architecture/2015/05/09/14.png" alt="Bean对象关系建立" align="center"/>  
<small><a href="/assets/images/article_imgs/architecture/2015/05/09/14.png" target="_blank">查看大图</a></small>

#### **2.3 <code>Ioc</code> 容器的扩展点**

现在还有一个问题就是如何让这些 <code>Bean</code>对象有一定的扩展性，就是可以加入用户的一些操作。那么有哪些扩展点呢？ <code>Spring</code> 又是如何调用到这些扩展点的？

对 <code>Spring</code> 的 <code>Ioc</code> 容器来说，主要有这么几个。<code>BeanFactoryPostProcessor</code>， <code>BeanPostProcessor</code>。他们分别是在构建 <code>BeanFactory</code> 和构建 <code>Bean</code> 对象时调用。还有就是 <code>InitializingBean</code> 和 <code>DisposableBean</code> 他们分别是在 <code>Bean</code> 实例创建和销毁时被调用。用户可以实现这些接口中定义的方法，<code>Spring</code> 就会在适当的时候调用他们。还有一个是 <code>FactoryBean</code> 他是个特殊的 <code>Bean</code>，这个 <code>Bean</code> 可以被用户更多的控制。

这些扩展点通常也是我们使用 <code>Spring</code> 来完成我们特定任务的地方，如何精通 <code>Spring</code> 就看你有没有掌握好 <code>Spring</code> 有哪些扩展点，并且如何使用他们，要知道如何使用他们就必须了解他们内在的机理。可以用下面一个比喻来解释。

我们把 <code>Ioc</code> 容器比作一个箱子，这个箱子里有若干个球的模子，可以用这些模子来造很多种不同的球，还有一个造这些球模的机器，这个机器可以产生球模。那么他们的对应关系就是 <code>BeanFactory</code> 就是那个造球模的机器，球模就是 <code>Bean</code>，而球模造出来的球就是 <code>Bean</code> 的实例。那前面所说的几个扩展点又在什么地方呢？ <code>BeanFactoryPostProcessor</code> 对应到当造球模被造出来时，你将有机会可以对其做出适当的修正，也就是他可以帮你修改球模。而 <code>InitializingBean</code> 和 <code>DisposableBean</code> 是在球模造球的开始和结束阶段，你可以完成一些预备和扫尾工作。<code>BeanPostProcessor</code> 就可以让你对球模造出来的球做出适当的修正。最后还有一个 <code>FactoryBean</code>，它可是一个神奇的球模。这个球模不是预先就定型了，而是由你来给他确定它的形状，既然你可以确定这个球模型的形状，当然他造出来的球肯定就是你想要的球了，这样在这个箱子里尼可以发现所有你想要的球

#### **2.4 <code>Ioc</code>容器如何为我所用**

前面的介绍了 <code>Spring</code> 容器的构建过程，那 <code>Spring</code> 能为我们做什么，<code>Spring</code> 的 <code>Ioc</code> 容器又能做什么呢？我们使用 <code>Spring</code> 必须要首先构建 <code>Ioc</code> 容器，没有它 <code>Spring</code> 无法工作，<code>ApplicatonContext.xml</code> 就是 <code>Ioc</code> 容器的默认配置文件，<code>Spring</code> 的所有特性功能都是基于这个 <code>Ioc</code> 容器工作的，比如后面要介绍的 <code>AOP</code>。

<code>Ioc</code> 它实际上就是为你构建了一个魔方，<code>Spring</code> 为你搭好了骨骼架构，这个魔方到底能变出什么好的东西出来，这必须要有你的参与。那我们怎么参与？这就是前面说的要了解 <code>Spring</code> 中那有些扩展点，我们通过实现那些扩展点来改变 <code>Spring</code> 的通用行为。至于如何实现扩展点来得到我们想要的个性结果，<code>Spring</code> 中有很多例子，其中 <code>AOP</code> 的实现就是 <code>Spring</code> 本身实现了其扩展点来达到了它想要的特性功能，可以拿来参考。

### **3、Spring 中 AOP 特性详解**

#### **3.1 动态代理的实现原理**

要了解 <code>Spring</code> 的 <code>AOP</code> 就必须先了解的动态代理的原理，因为 <code>AOP</code> 就是基于动态代理实现的。动态代理还要从 <code>JDK</code> 本身说起。

在 <code>Jdk</code> 的 <code>java.lang.reflect</code> 包下有个 <code>Proxy</code> 类，它正是构造代理类的入口。这个类的结构入下：

图 15. <code>Proxy</code> 类结构

<img src="/assets/images/article_imgs/architecture/2015/05/09/15.png" alt="Proxy类结构" align="center"/>

从上图发现最后面四个是公有方法。而最后一个方法 <code>newProxyInstance</code> 就是创建代理对象的方法。这个方法的源码如下：

清单 6. <code>Proxy.newProxyInstance</code>

{% highlight java %}
public static Object newProxyInstance(ClassLoader loader, 
    Class<?>[] interfaces, 
    InvocationHandler h) 
    throws IllegalArgumentException { 

    if (h == null) { 
        throw new NullPointerException(); 
    } 
    Class cl = getProxyClass(loader, interfaces); 
    try { 
        Constructor cons = cl.getConstructor(constructorParams);
        return (Object) cons.newInstance(new Object[] { h }); 
    } catch (NoSuchMethodException e) { 
        throw new InternalError(e.toString()); 
    } catch (IllegalAccessException e) { 
        throw new InternalError(e.toString()); 
    } catch (InstantiationException e) { 
        throw new InternalError(e.toString()); 
    } catch (InvocationTargetException e) { 
        throw new InternalError(e.toString()); 
    } 
}
{% endhighlight %}

这个方法需要三个参数：<code>ClassLoader</code>，用于加载代理类的 <code>Loader</code> 类，通常这个 <code>Loader</code> 和被代理的类是同一个 <code>Loader</code> 类。<code>Interfaces</code>，是要被代理的那些那些接口。<code>InvocationHandler</code>，就是用于执行除了被代理接口中方法之外的用户自定义的操作，他也是用户需要代理的最终目的。用户调用目标方法都被代理到 <code>InvocationHandler</code> 类中定义的唯一方法 <code>invoke</code> 中。这在后面再详解。

下面还是看看<code>Proxy</code>
如何产生代理类的过程，他构造出来的代理类到底是什么样子？下面揭晓啦。

图 16. 创建代理对象时序图

<img src="/assets/images/article_imgs/architecture/2015/05/09/16.png" alt="创建代理对象时序图" align="center"/>

其实从上图中可以发现正在构造代理类的是在 <code>ProxyGenerator</code> 的 <code>generateProxyClass</code> 的方法中。<code>ProxyGenerator</code> 类在 <code>sun.misc</code> 包下，感兴趣的话可以看看他的源码。

假如有这样一个接口，如下：

清单 7. <code>SimpleProxy</code> 类

{% highlight java %}
public interface SimpleProxy { 
    public void simpleMethod1(); 
    public void simpleMethod2(); 
}
{% endhighlight %}

代理来生成的类结构如下：

清单 8. <code>$Proxy2</code>类

{% highlight java %}
public class $Proxy2 extends java.lang.reflect.Proxy implements SimpleProxy{ 
    java.lang.reflect.Method m0; 
    java.lang.reflect.Method m1; 
    java.lang.reflect.Method m2; 
    java.lang.reflect.Method m3; 
    java.lang.reflect.Method m4; 

    int hashCode(); 
    boolean equals(java.lang.Object); 
    java.lang.String toString(); 
    void simpleMethod1(); 
    void simpleMethod2(); 
}
{% endhighlight %}

这个类中的方法里面将会是调用 <code>InvocationHandler</code> 的 <code>invoke</code> 方法，而每个方法也将对应一个属性变量，这个属性变量 <code>m</code> 也将传给 <code>invoke</code> 方法中的 <code>Method</code> 参数。整个代理就是这样实现的。

#### **3.2 Spring AOP 如何实现**

从前面代理的原理我们知道，代理的目的是调用目标方法时我们可以转而执行 <code>InvocationHandler</code> 类的 <code>invoke</code> 方法，所以如何在 <code>InvocationHandler</code> 上做文章就是 <code>Spring</code> 实现 <code>Aop</code> 的关键所在。

<code>Spring</code> 的 <code>Aop</code> 实现是遵守 <code>Aop</code> 联盟的约定。同时 <code>Spring</code> 又扩展了它，增加了如 <code>Pointcut</code>、<code>Advisor</code> 等一些接口使得更加灵活。

下面是 Jdk 动态代理的类图：

图 17. Jdk 动态代理的类图
<img src="/assets/images/article_imgs/architecture/2015/05/09/17.png" alt="创建代理对象时序图" align="center"/>

上图清楚的显示了 <code>Spring</code> 引用了 <code>Aop Alliance</code> 定义的接口。姑且不讨论 <code>Spring</code> 如何扩展 <code>Aop Alliance</code>，先看看 <code>Spring</code> 如何实现代理类的，要实现代理类在 <code>Spring</code> 的配置文件中通常是这样定一个 <code>Bean</code> 的，如下：

清单 9. 配置代理类 <code>Bean</code>

{% highlight xml %}
<bean id="testBeanSingleton" 
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces">
        <value>
            org.springframework.aop.framework.PrototypeTargetTests$TestBean
        </value>
    </property>
    <property name="target"><ref local="testBeanTarget"></ref> </property>
    <property name="singleton"><value>true</value></property>
    <property name="interceptorNames">
        <list>
            <value>testInterceptor</value>
            <value>testInterceptor2</value>
        </list>
    </property>
</bean>
{% endhighlight %}

配置上看到要设置被代理的接口，和接口的实现类也就是目标类，以及拦截器也就在执行目标方法之前被调用，这里 <code>Spring</code> 中定义的各种各样的拦截器，可以选择使用。
下面看看 <code>Spring</code> 如何完成了代理以及是如何调用拦截器的。

前面提到 <code>Spring Aop</code> 也是实现其自身的扩展点来完成这个特性的，从这个代理类可以看出它正是继承了 <code>FactoryBean</code> 的 <code>ProxyFactoryBean</code>，<code>FactoryBean</code> 之所以特别就在它可以让你自定义对象的创建方法。当然代理对象要通过 <code>Proxy</code> 类来动态生成。

下面是 <code>Spring</code> 创建的代理对象的时序图：

图 18.<code>Spring</code> 代理对象的产生

<img src="/assets/images/article_imgs/architecture/2015/05/09/18.png" alt="Spring代理对象的产生" align="center"/>

<code>Spring</code> 创建了代理对象后，当你调用目标对象上的方法时，将都会被代理到 <code>InvocationHandler</code> 类的 <code>invoke</code> 方法中执行，这在前面已经解释。在这里 <code>JdkDynamicAopProxy</code> 类实现了 <code>InvocationHandler</code> 接口。

下面再看看 <code>Spring</code> 是如何调用拦截器的，下面是这个过程的时序图：

图 19.<code>Spring</code> 调用拦截器

<img src="/assets/images/article_imgs/architecture/2015/05/09/19.png" alt="Spring代理对象的产生" align="center"/>

以上所说的都是 <code>Jdk</code> 动态代理，<code>Spring</code> 还支持一种 <code>CGLIB</code> 类代理，感兴趣自己看吧。

## **四、Spring 中设计模式分析**

<code>Spring</code> 中使用的设计模式也很多，比如工厂模式、单例模式、模版模式等，在《Webx框架的系统架构与设计模式》、《Tomcat的系统架构与模式设计分析》已经有介绍，这里就不赘述了。这里主要介绍代理模式和策略模式。

### **1、代理模式**

#### **1.1 代理模式原理**

代理模式就是给某一个对象创建一个代理对象，而由这个代理对象控制对原对象的引用，而创建这个代理对象就是可以在调用原对象是可以增加一些额外的操作。下面是代理模式的结构：

图 20. 代理模式的结构

<img src="/assets/images/article_imgs/architecture/2015/05/09/20.png" alt="代理模式的结构" align="center"/>

*  <code>Subject</code>：抽象主题，它是代理对象的真实对象要实现的接口，当然这可以是多个接口组成。  
*  <code>ProxySubject</code>：代理类除了实现抽象主题定义的接口外，还必须持有所代理对象的引用。
*  <code>RealSubject</code>：被代理的类，是目标对象。

#### **1.2 <code>Spring</code> 中如何实现代理模式**

<code>Spring Aop</code> 中 <code>Jdk</code> 动态代理就是利用代理模式技术实现的。在 <code>Spring</code> 中除了实现被代理对象的接口外，还会有 <code>org.springframework.aop.SpringProxy</code> 和 <code>org.springframework.aop.framework.Advised</code> 两个接口。<code>Spring</code> 中使用代理模式的结构图如下：

图 21. <code>Spring</code> 中使用代理模式的结构图

<img src="/assets/images/article_imgs/architecture/2015/05/09/21.gif" alt="Spring中使用代理模式的结构图" align="center"/>

<code>$Proxy</code> 就是创建的代理对象，而 <code>Subject</code> 是抽象主题，代理对象是通过 <code>InvocationHandler</code> 来持有对目标对象的引用的。

<code>Spring</code> 中一个真实的代理对象结构如下：

清单 10 代理对象 <code>$Proxy4</code>

{% highlight java %}
public class $Proxy4 extends java.lang.reflect.Proxy implements 
    org.springframework.aop.framework.PrototypeTargetTests$TestBean 
    org.springframework.aop.SpringProxy 
    org.springframework.aop.framework.Advised
{
    java.lang.reflect.Method m16;
    java.lang.reflect.Method m9;
    java.lang.reflect.Method m25;
    java.lang.reflect.Method m5;
    java.lang.reflect.Method m2;
    java.lang.reflect.Method m23;
    java.lang.reflect.Method m18;
    java.lang.reflect.Method m26;
    java.lang.reflect.Method m6;
    java.lang.reflect.Method m28;
    java.lang.reflect.Method m14;
    java.lang.reflect.Method m12;
    java.lang.reflect.Method m27;
    java.lang.reflect.Method m11;
    java.lang.reflect.Method m22;
    java.lang.reflect.Method m3;
    java.lang.reflect.Method m8;
    java.lang.reflect.Method m4;
    java.lang.reflect.Method m19;
    java.lang.reflect.Method m7;
    java.lang.reflect.Method m15;
    java.lang.reflect.Method m20;
    java.lang.reflect.Method m10;
    java.lang.reflect.Method m1;
    java.lang.reflect.Method m17;
    java.lang.reflect.Method m21;
    java.lang.reflect.Method m0;
    java.lang.reflect.Method m13;
    java.lang.reflect.Method m24;

    int hashCode();
    int indexOf(org.springframework.aop.Advisor);
    int indexOf(org.aopalliance.aop.Advice);
    boolean equals(java.lang.Object);
    java.lang.String toString();
    void sayhello();
    void doSomething();
    void doSomething2();
    java.lang.Class getProxiedInterfaces();
    java.lang.Class getTargetClass();
    boolean isProxyTargetClass();
    org.springframework.aop.Advisor; getAdvisors();
    void addAdvisor(int, org.springframework.aop.Advisor) 
        throws org.springframework.aop.framework.AopConfigException;
    void addAdvisor(org.springframework.aop.Advisor)
        throws org.springframework.aop.framework.AopConfigException;
    void setTargetSource(org.springframework.aop.TargetSource);
    org.springframework.aop.TargetSource getTargetSource();
    void setPreFiltered(boolean);
    boolean isPreFiltered();
    boolean isInterfaceProxied(java.lang.Class);
    boolean removeAdvisor(org.springframework.aop.Advisor);
    void removeAdvisor(int)throws org.springframework.aop.framework.AopConfigException;
    boolean replaceAdvisor(org.springframework.aop.Advisor, 
        org.springframework.aop.Advisor)
        throws org.springframework.aop.framework.AopConfigException;
    void addAdvice(org.aopalliance.aop.Advice) 
        throws org.springframework.aop.framework.AopConfigException;
    void addAdvice(int, org.aopalliance.aop.Advice) 
        throws org.springframework.aop.framework.AopConfigException;
    boolean removeAdvice(org.aopalliance.aop.Advice);
    java.lang.String toProxyConfigString();
    boolean isFrozen();
    void setExposeProxy(boolean);
    boolean isExposeProxy();
}
{% endhighlight %}

### **2、策略模式**

#### ** 2.1 策略模式原理**

策略模式顾名思义就是做某事的策略，这在编程上通常是指完成某个操作可能有多种方法，这些方法各有千秋，可能有不同的适应的场合，然而这些操作方法都有可能用到。各一个操作方法都当作一个实现策略，使用者可能根据需要选择合适的策略。
下面是策略模式的结构：

图 22. 策略模式的结构

<img src="/assets/images/article_imgs/architecture/2015/05/09/22.png" alt="策略模式的结构" align="center"/>

*  <code>Context</code>：使用不同策略的环境，它可以根据自身的条件选择不同的策略实现类来完成所要的操作。它持有一个策略实例的引用。创建具体策略对象的方法也可以由他完成。
*  <code>Strategy</code>：抽象策略，定义每个策略都要实现的策略方法
*  <code>ConcreteStrategy</code>：具体策略实现类，实现抽象策略中定义的策略方法

#### **2.2 <code>Spring</code> 中策略模式的实现**

<code>Spring</code> 中策略模式使用有多个地方，如 <code>Bean</code> 定义对象的创建以及代理对象的创建等。这里主要看一下代理对象创建的策略模式的实现。

前面已经了解 <code>Spring</code> 的代理方式有两个 <code>Jdk</code> 动态代理和 <code>CGLIB</code> 代理。这两个代理方式的使用正是使用了策略模式。它的结构图如下所示：

图 23. <code>Spring</code> 中策略模式结构图

<img src="/assets/images/article_imgs/architecture/2015/05/09/23.png" alt="Spring中策略模式结构图" align="center"/>

在上面结构图中与标准的策略模式结构稍微有点不同，这里抽象策略是 <code>AopProxy</code> 接口，<code>Cglib2AopProxy</code> 和 <code>JdkDynamicAopProxy</code> 分别代表两种策略的实现方式，<code>ProxyFactoryBean</code> 就是代表 <code>Context</code> 角色，它根据条件选择使用 <code>Jdk</code> 代理方式还是 <code>CGLIB</code> 方式，而另外三个类主要是来负责创建具体策略对象，<code>ProxyFactoryBean</code> 是通过依赖的方法来关联具体策略对象的，它是通过调用策略对象的 <code>getProxy(ClassLoader classLoader)</code> 方法来完成操作。


## **总结**

本文通过从 <code>Spring</code> 的几个核心组件入手，试图找出构建 <code>Spring</code> 框架的骨骼架构，进而分析 <code>Spring</code> 在设计的一些设计理念，是否从中找出一些好的设计思想，对我们以后程序设计能提供一些思路。接着再详细分析了 <code>Spring</code> 中是如何实现这些理念的，以及在设计模式上是如何使用的。
通过分析 <code>Spring</code> 给我一个很大的启示就是其这套设计理念其实对我们有很强的借鉴意义，它通过抽象复杂多变的对象，进一步做规范，然后根据它定义的这套规范设计出一个容器，容器中构建它们的复杂关系，其实现在有很多情况都可以用这种类似的处理方法。

本文来源ibm开发者社区，并略作修改，原文地址：[http://www.ibm.com/developerworks/cn/java/j-lo-spring-principle/](http://www.ibm.com/developerworks/cn/java/j-lo-spring-principle/)
