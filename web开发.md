## 1. Spring Boot

#### 1.1 说说你对Spring Boot的理解

**参考答案**

从本质上来说，Spring Boot就是Spring，它做了那些没有它你自己也会去做的Spring Bean配置。Spring Boot使用“习惯优于配置”的理念让你的项目快速地运行起来，使用Spring Boot很容易创建一个能独立运行、准生产级别、基于Spring框架的项目，使用Spring Boot你可以不用或者只需要很少的Spring配置。

简而言之，Spring Boot本身并不提供Spring的核心功能，而是作为Spring的脚手架框架，以达到快速构建项目、预置三方配置、开箱即用的目的。Spring Boot有如下的优点：

- 可以快速构建项目；
- 可以对主流开发框架的无配置集成；
- 项目可独立运行，无需外部依赖Servlet容器；
- 提供运行时的应用监控；
- 可以极大地提高开发、部署效率；
- 可以与云计算天然集成。

#### 1.2 Spring Boot Starter有什么用？

**参考答案**

Spring Boot通过提供众多起步依赖（Starter）降低项目依赖的复杂度。起步依赖本质上是一个Maven项目对象模型（Project Object Model, POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。很多起步依赖的命名都暗示了它们提供的某种或某类功能。

举例来说，你打算把这个阅读列表应用程序做成一个Web应用程序。与其向项目的构建文件里添加一堆单独的库依赖，还不如声明这是一个Web应用程序来得简单。你只要添加Spring Boot的Web起步依赖就好了。

#### 1.3 介绍Spring Boot的启动流程

**参考答案**

首先，Spring Boot项目创建完成会默认生成一个名为 `*Application` 的入口类，我们是通过该类的main方法启动Spring Boot项目的。在main方法中，通过SpringApplication的静态方法，即run方法进行SpringApplication类的实例化操作，然后再针对实例化对象调用另外一个run方法来完成整个项目的初始化和启动。

SpringApplication调用的run方法的大致流程，如下图：

![img](img/web开发/springboot-1.jpg)

其中，SpringApplication在run方法中重点做了以下操作：

- 获取监听器和参数配置；
- 打印Banner信息；
- 创建并初始化容器；
- 监听器发送通知。

当然，除了上述核心操作，run方法运行过程中还涉及启动时长统计、异常报告、启动日志、异常处理等辅助操作。比较完整的流程，可以参考如下源代码：

```java
public ConfigurableApplicationContext run(String... args) {  
    // 创建StopWatch对象，用于统计run方法启动时长。  
    StopWatch stopWatch = new StopWatch();  
    // 启动统计  
    stopWatch.start();  
    ConfigurableApplicationContext context = null; 
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();  
    // 配置Headless属性
    configureHeadlessProperty(); 
    // 获得SpringApplicationRunListener数组， 
    // 该数组封装于SpringApplicationRunListeners对象的listeners中。 
    SpringApplicationRunListeners listeners = getRunListeners(args); 
    // 启动监听，遍历SpringApplicationRunListener数组每个元素，并执行。  
    listeners.starting();  try {   
        // 创建ApplicationArguments对象   
        ApplicationArguments applicationArguments =    
            new DefaultApplicationArguments(args);  
        // 加载属性配置，包括所有的配置属性。  
        ConfigurableEnvironment environment =     
            prepareEnvironment(listeners, applicationArguments);
        configureIgnoreBeanInfo(environment);    
        // 打印Banner   
        Banner printedBanner = printBanner(environment);    
        // 创建容器   
        context = createApplicationContext();    
        // 异常报告器   
        exceptionReporters = getSpringFactoriesInstances(   
            SpringBootExceptionReporter.class,    
            new Class[] { ConfigurableApplicationContext.class }, context);  
        // 准备容器，组件对象之间进行关联。  
        prepareContext(context, environment,    
                       listeners, applicationArguments, printedBanner);   
        // 初始化容器    
        refreshContext(context);   
        // 初始化操作之后执行，默认实现为空。  
        afterRefresh(context, applicationArguments);    
        // 停止时长统计    
        stopWatch.stop();   
        // 打印启动日志    
        if (this.logStartupInfo) {     
            new StartupInfoLogger(this.mainApplicationClass)   
                .logStarted(getApplicationLog(), stopWatch);   
        }    
        // 通知监听器：容器完成启动。   
        listeners.started(context);    
        // 调用ApplicationRunner和CommandLineRunner的运行方法。
        callRunners(context, applicationArguments); 
    } catch (Throwable ex) {   
        // 异常处理    
        handleRunFailure(context, ex, exceptionReporters, listeners);  
        throw new IllegalStateException(ex); 
    }  
    
    try {    
        // 通知监听器：容器正在运行。
        listeners.running(context); 
    } catch (Throwable ex) {    
        // 异常处理    
        handleRunFailure(context, ex, exceptionReporters, null);  
        throw new IllegalStateException(ex); 
    } 
    return context;
}
```

#### 1.4 Spring Boot项目是如何导入包的？

**参考答案**

通过Spring Boot Starter导入包。

Spring Boot通过提供众多起步依赖（Starter）降低项目依赖的复杂度。起步依赖本质上是一个Maven项目对象模型（Project Object Model, POM），定义了对其他库的传递依赖，这些东西加在一起即支持某项功能。很多起步依赖的命名都暗示了它们提供的某种或某类功能。

举例来说，你打算把这个阅读列表应用程序做成一个Web应用程序。与其向项目的构建文件里添加一堆单独的库依赖，还不如声明这是一个Web应用程序来得简单。你只要添加Spring Boot的Web起步依赖就好了。

#### 1.5 请描述Spring Boot自动装配的过程

**参考答案**

使用Spring Boot时，我们只需引入对应的Starters，Spring Boot启动时便会自动加载相关依赖，配置相应的初始化参数，以最快捷、简单的形式对第三方软件进行集成，这便是Spring Boot的自动配置功能。Spring Boot实现该运作机制锁涉及的核心部分如下图所示：

![img](img/web开发/springboot-2.jpg)

整个自动装配的过程是：Spring Boot通过@EnableAutoConfiguration注解开启自动配置，加载spring.factories中注册的各种AutoConfiguration类，当某个AutoConfiguration类满足其注解@Conditional指定的生效条件（Starters提供的依赖、配置或Spring容器中是否存在某个Bean等）时，实例化该AutoConfiguration类中定义的Bean（组件等），并注入Spring容器，就可以完成依赖框架的自动配置。

#### 1.6 说说你对Spring Boot注解的了解

**参考答案**

@SpringBootApplication注解：

在Spring Boot入口类中，唯一的一个注解就是@SpringBootApplication。它是Spring Boot项目的核心注解，用于开启自动配置，准确说是通过该注解内组合的@EnableAutoConfiguration开启了自动配置。

@EnableAutoConfiguration注解：

@EnableAutoConfiguration的主要功能是启动Spring应用程序上下文时进行自动配置，它会尝试猜测并配置项目可能需要的Bean。自动配置通常是基于项目classpath中引入的类和已定义的Bean来实现的。在此过程中，被自动配置的组件来自项目自身和项目依赖的jar包中。

@Import注解：

@EnableAutoConfiguration的关键功能是通过@Import注解导入的ImportSelector来完成的。从源代码得知@Import(AutoConfigurationImportSelector.class)是@EnableAutoConfiguration注解的组成部分，也是自动配置功能的核心实现者。

@Conditional注解：

@Conditional注解是由Spring 4.0版本引入的新特性，可根据是否满足指定的条件来决定是否进行Bean的实例化及装配，比如，设定当类路径下包含某个jar包的时候才会对注解的类进行实例化操作。总之，就是根据一些特定条件来控制Bean实例化的行为。

@Conditional衍生注解：

在Spring Boot的autoconfigure项目中提供了各类基于@Conditional注解的衍生注解，它们适用不同的场景并提供了不同的功能。通过阅读这些注解的源码，你会发现它们其实都组合了@Conditional注解，不同之处是它们在注解中指定的条件（Condition）不同。

- @ConditionalOnBean：在容器中有指定Bean的条件下。
- @ConditionalOnClass：在classpath类路径下有指定类的条件下。
- @ConditionalOnCloudPlatform：当指定的云平台处于active状态时。
- @ConditionalOnExpression：基于SpEL表达式的条件判断。
- @ConditionalOnJava：基于JVM版本作为判断条件。
- @ConditionalOnJndi：在JNDI存在的条件下查找指定的位置。
- @ConditionalOnMissingBean：当容器里没有指定Bean的条件时。
- @ConditionalOnMissingClass：当类路径下没有指定类的条件时。
- @ConditionalOnNotWebApplication：在项目不是一个Web项目的条件下。
- @ConditionalOnProperty：在指定的属性有指定值的条件下。
- @ConditionalOnResource：类路径是否有指定的值。
- @ConditionalOnSingleCandidate：当指定的Bean在容器中只有一个或者有多个但是指定了首选的Bean时。
- @ConditionalOnWebApplication：在项目是一个Web项目的条件下。

## 2. Spring

#### 2.1 请你说说Spring的核心是什么

**参考答案**

Spring框架包含众多模块，如Core、Testing、Data Access、Web Servlet等，其中Core是整个Spring框架的核心模块。Core模块提供了IoC容器、AOP功能、数据绑定、类型转换等一系列的基础功能，而这些功能以及其他模块的功能都是建立在IoC和AOP之上的，所以IoC和AOP是Spring框架的核心。

IoC（Inversion of Control）是控制反转的意思，这是一种面向对象编程的设计思想。在不采用这种思想的情况下，我们需要自己维护对象与对象之间的依赖关系，很容易造成对象之间的耦合度过高，在一个大型的项目中这十分的不利于代码的维护。IoC则可以解决这种问题，它可以帮我们维护对象与对象之间的依赖关系，降低对象之间的耦合度。

说到IoC就不得不说DI（Dependency Injection），DI是依赖注入的意思，它是IoC实现的实现方式，就是说IoC是通过DI来实现的。由于IoC这个词汇比较抽象而DI却更直观，所以很多时候我们就用DI来代替它，在很多时候我们简单地将IoC和DI划等号，这是一种习惯。而实现依赖注入的关键是IoC容器，它的本质就是一个工厂。

AOP（Aspect Oriented Programing）是面向切面编程思想，这种思想是对OOP的补充，它可以在OOP的基础上进一步提高编程的效率。简单来说，它可以统一解决一批组件的共性需求（如权限检查、记录日志、事务管理等）。在AOP思想下，我们可以将解决共性需求的代码独立出来，然后通过配置的方式，声明这些代码在什么地方、什么时机调用。当满足调用条件时，AOP会将该业务代码织入到我们指定的位置，从而统一解决了问题，又不需要修改这一批组件的代码。

#### 2.2 说一说你对Spring容器的了解

**参考答案**

Spring主要提供了两种类型的容器：BeanFactory和ApplicationContext。

- BeanFactory：是基础类型的IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略。只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景，BeanFactory是比较合适的IoC容器选择。
- ApplicationContext：它是在BeanFactory的基础上构建的，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等。ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来说，ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext类型的容器是比较合适的选择。

#### 2.3 说一说你对BeanFactory的了解

**参考答案**

BeanFactory是一个类工厂，与传统类工厂不同的是，BeanFactory是类的通用工厂，它可以创建并管理各种类的对象。这些可被创建和管理的对象本身没有什么特别之处，仅是一个POJO，Spring称这些被创建和管理的Java对象为Bean。并且，Spring中所说的Bean比JavaBean更为宽泛一些，所有可以被Spring容器实例化并管理的Java类都可以成为Bean。

BeanFactory是Spring容器的顶层接口，Spring为BeanFactory提供了多种实现，最常用的是XmlBeanFactory。但它在Spring 3.2中已被废弃，建议使用XmlBeanDefinitionReader、DefaultListableBeanFactory替代。BeanFactory最主要的方法就是 `getBean(String beanName)`，该方法从容器中返回特定名称的Bean。

#### 2.4 说一说你对Spring IOC的理解

**参考答案**

IoC（Inversion of Control）是控制反转的意思，这是一种面向对象编程的设计思想。在不采用这种思想的情况下，我们需要自己维护对象与对象之间的依赖关系，很容易造成对象之间的耦合度过高，在一个大型的项目中这十分的不利于代码的维护。IoC则可以解决这种问题，它可以帮我们维护对象与对象之间的依赖关系，降低对象之间的耦合度。

说到IoC就不得不说DI（Dependency Injection），DI是依赖注入的意思，它是IoC实现的实现方式，就是说IoC是通过DI来实现的。由于IoC这个词汇比较抽象而DI却更直观，所以很多时候我们就用DI来代替它，在很多时候我们简单地将IoC和DI划等号，这是一种习惯。而实现依赖注入的关键是IoC容器，它的本质就是一个工厂。

在具体的实现中，主要由三种注入方式：

1. 构造方法注入

   就是被注入对象可以在它的构造方法中声明依赖对象的参数列表，让外部知道它需要哪些依赖对象。然后，IoC Service Provider会检查被注入的对象的构造方法，取得它所需要的依赖对象列表，进而为其注入相应的对象。构造方法注入方式比较直观，对象被构造完成后，即进入就绪状态，可以马上使用。

2. setter方法注入

   通过setter方法，可以更改相应的对象属性。所以，当前对象只要为其依赖对象所对应的属性添加setter方法，就可以通过setter方法将相应的依赖对象设置到被注入对象中。setter方法注入虽不像构造方法注入那样，让对象构造完成后即可使用，但相对来说更宽松一些，
   可以在对象构造完成后再注入。

3. 接口注入

   相对于前两种注入方式来说，接口注入没有那么简单明了。被注入对象如果想要IoC Service Provider为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。IoC Service Provider最终通过这些接口来了解应该为被注入对象注入什么依赖对象。相对于前两种依赖注入方式，接口注入比较死板和烦琐。

总体来说，构造方法注入和setter方法注入因为其侵入性较弱，且易于理解和使用，所以是现在使用最多的注入方式。而接口注入因为侵入性较强，近年来已经不流行了。

#### 2.5 Spring是如何管理Bean的？

**参考答案**

Spring通过IoC容器来管理Bean，我们可以通过XML配置或者注解配置，来指导IoC容器对Bean的管理。因为注解配置比XML配置方便很多，所以现在大多时候会使用注解配置的方式。

以下是管理Bean时常用的一些注解：

1. @ComponentScan用于声明扫描策略，通过它的声明，容器就知道要扫描哪些包下带有声明的类，也可以知道哪些特定的类是被排除在外的。
2. @Component、@Repository、@Service、@Controller用于声明Bean，它们的作用一样，但是语义不同。@Component用于声明通用的Bean，@Repository用于声明DAO层的Bean，@Service用于声明业务层的Bean，@Controller用于声明视图层的控制器Bean，被这些注解声明的类就可以被容器扫描并创建。
3. @Autowired、@Qualifier用于注入Bean，即告诉容器应该为当前属性注入哪个Bean。其中，@Autowired是按照Bean的类型进行匹配的，如果这个属性的类型具有多个Bean，就可以通过@Qualifier指定Bean的名称，以消除歧义。
4. @Scope用于声明Bean的作用域，默认情况下Bean是单例的，即在整个容器中这个类型只有一个实例。可以通过@Scope注解指定prototype值将其声明为多例的，也可以将Bean声明为session级作用域、request级作用域等等，但最常用的还是默认的单例模式。
5. @PostConstruct、@PreDestroy用于声明Bean的生命周期。其中，被@PostConstruct修饰的方法将在Bean实例化后被调用，@PreDestroy修饰的方法将在容器销毁前被调用。

#### 2.6 介绍Bean的作用域

**参考答案**

默认情况下，Bean在Spring容器中是单例的，我们可以通过@Scope注解修改Bean的作用域。该注解有如下5个取值，它们代表了Bean的5种不同类型的作用域：

|     类型      |                             说明                             |
| :-----------: | :----------------------------------------------------------: |
|   singleton   |    在Spring容器中仅存在一个实例，即Bean以单例的形式存在。    |
|   prototype   |   每次调用getBean()时，都会执行new操作，返回一个新的实例。   |
|    request    |              每次HTTP请求都会创建一个新的Bean。              |
|    session    | 同一个HTTP Session共享一个Bean，不同的HTTP Session使用不同的Bean。 |
| globalSession |    同一个全局的Session共享一个Bean，一般用于Portlet环境。    |

#### 2.7 说一说Bean的生命周期

**参考答案**

Spring容器管理Bean，涉及对Bean的创建、初始化、调用、销毁等一系列的流程，这个流程就是Bean的生命周期。整个流程参考下图：

![img](img/web开发/spring-1.jpg)

这个过程是由Spring容器自动管理的，其中有两个环节我们可以进行干预。

1. 我们可以自定义初始化方法，并在该方法前增加@PostConstruct注解，届时Spring容器将在调用SetBeanFactory方法之后调用该方法。
2. 我们可以自定义销毁方法，并在该方法前增加@PreDestroy注解，届时Spring容器将在自身销毁前，调用这个方法。

#### 2.8 Spring是怎么解决循环依赖的？

**参考答案**

首先，需要明确的是spring对循环依赖的处理有三种情况：

1. 构造器的循环依赖：这种依赖spring是处理不了的，直接抛出BeanCurrentlylnCreationException异常。
2. 单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖。
3. 非单例循环依赖：无法处理。

接下来，我们具体看看spring是如何处理第二种循环依赖的。

Spring单例对象的初始化大略分为三步：

1. createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象；
2. populateBean：填充属性，这一步主要是多bean的依赖属性进行填充；
3. initializeBean：调用spring xml中的init 方法。

从上面讲述的单例bean初始化步骤我们可以知道，循环依赖主要发生在第一步、第二步。也就是构造器循环依赖和field循环依赖。 Spring为了解决单例的循环依赖问题，使用了三级缓存。

```java
/** Cache of singleton objects: bean name –> bean instance */
private final Map singletonObjects = new ConcurrentHashMap(256);
/** Cache of singleton factories: bean name –> ObjectFactory */
private final Map> singletonFactories = new HashMap>(16);
/** Cache of early singleton objects: bean name –> bean instance */
private final Map earlySingletonObjects = new HashMap(16);
```

这三级缓存的作用分别是：

- singletonFactories ： 进入实例化阶段的单例对象工厂的cache （三级缓存）；
- earlySingletonObjects ：完成实例化但是尚未初始化的，提前暴光的单例对象的Cache （二级缓存）；
- singletonObjects：完成初始化的单例对象的cache（一级缓存）。

我们在创建bean的时候，会首先从cache中获取这个bean，这个缓存就是sigletonObjects。主要的调用方法是：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {  
    Object singletonObject = this.singletonObjects.get(beanName);  
    //isSingletonCurrentlyInCreation()判断当前单例bean是否正在创建中  
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {    
        synchronized (this.singletonObjects) {      
            singletonObject = this.earlySingletonObjects.get(beanName);      
            //allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象      
            if (singletonObject == null && allowEarlyReference) {        
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);        
                if (singletonFactory != null) {          
                    singletonObject = singletonFactory.getObject();          
                    //从singletonFactories中移除，并放入earlySingletonObjects中。          
                    //其实也就是从三级缓存移动到了二级缓存          
                    this.earlySingletonObjects.put(beanName, singletonObject);          
                    this.singletonFactories.remove(beanName);        
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

从上面三级缓存的分析，我们可以知道，Spring解决循环依赖的诀窍就在于singletonFactories这个三级cache。这个cache的类型是ObjectFactory，定义如下：

```java
public interface ObjectFactory<T> { 
    T getObject() throws BeansException;
}
```

这个接口在AbstractBeanFactory里实现，并在核心方法doCreateBean（）引用下面的方法：

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {  
    Assert.notNull(singletonFactory, "Singleton factory must not be null");  
    synchronized (this.singletonObjects) {    
        if (!this.singletonObjects.containsKey(beanName)) {  
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

这段代码发生在createBeanInstance之后，populateBean（）之前，也就是说单例对象此时已经被创建出来(调用了构造器)。这个对象已经被生产出来了，此时将这个对象提前曝光出来，让大家使用。

这样做有什么好处呢？让我们来分析一下“A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。

#### 2.9 @Autowired和@Resource注解有什么区别？

**参考答案**

1. @Autowired是Spring提供的注解，@Resource是JDK提供的注解。
2. @Autowired是只能按类型注入，@Resource默认按名称注入，也支持按类型注入。
3. @Autowired按类型装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false，如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。@Resource有两个中重要的属性：name和type。name属性指定byName，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。需要注意的是，@Resource如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。

#### 2.10 Spring中默认提供的单例是线程安全的吗？

**参考答案**

不是。

Spring容器本身并没有提供Bean的线程安全策略。如果单例的Bean是一个无状态的Bean，即线程中的操作不会对Bean的成员执行查询以外的操作，那么这个单例的Bean是线程安全的。比如，Controller、Service、DAO这样的组件，通常都是单例且线程安全的。如果单例的Bean是一个有状态的Bean，则可以采用ThreadLocal对状态数据做线程隔离，来保证线程安全。

#### 2.11 说一说你对Spring AOP的理解

**参考答案**

AOP（Aspect Oriented Programming）是面向切面编程，它是一种编程思想，是面向对象编程（OOP）的一种补充。面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。如下图，可以很形象地看出，所谓切面，相当于应用对象间的横切点，我们可以将其单独抽象为单独的模块。

![img](img/web开发/spring-2.png)

AOP的术语：

- 连接点（join point）：对应的是具体被拦截的对象，因为Spring只能支持方法，所以被拦截的对象往往就是指特定的方法，AOP将通过动态代理技术把它织入对应的流程中。
- 切点（point cut）：有时候，我们的切面不单单应用于单个方法，也可能是多个类的不同方法，这时，可以通过正则式和指示器的规则去定义，从而适配连接点。切点就是提供这样一个功能的概念。
- 通知（advice）：就是按照约定的流程下的方法，分为前置通知、后置通知、环绕通知、事后返回通知和异常通知，它会根据约定织入流程中。
- 目标对象（target）：即被代理对象。
- 引入（introduction）：是指引入新的类和其方法，增强现有Bean的功能。
- 织入（weaving）：它是一个通过动态代理技术，为原有服务对象生成代理对象，然后将与切点定义匹配的连接点拦截，并按约定将各类通知织入约定流程的过程。
- 切面（aspect）：是一个可以定义切点、各类通知和引入的内容，SpringAOP将通过它的信息来增强Bean的功能或者将对应的方法织入流程。

Spring AOP：

AOP可以有多种实现方式，而Spring AOP支持如下两种实现方式。

- JDK动态代理：这是Java提供的动态代理技术，可以在运行时创建接口的代理实例。Spring AOP默认采用这种方式，在接口的代理实例中织入代码。
- CGLib动态代理：采用底层的字节码技术，在运行时创建子类代理的实例。当目标对象不存在接口时，Spring AOP就会采用这种方式，在子类实例中织入代码。

#### 2.12 请你说说AOP的应用场景

**参考答案**

Spring AOP为IoC的使用提供了更多的便利，一方面，应用可以直接使用AOP的功能，设计应用的横切关注点，把跨越应用程序多个模块的功能抽象出来，并通过简单的AOP的使用，灵活地编制到模块中，比如可以通过AOP实现应用程序中的日志功能。另一方面，在Spring内部，一些支持模块也是通过Spring AOP来实现的，比如事务处理。从这两个角度就已经可以看到Spring AOP的核心地位了。

#### 2.13 Spring AOP不能对哪些类进行增强？

**参考答案**

1. Spring AOP只能对IoC容器中的Bean进行增强，对于不受容器管理的对象不能增强。
2. 由于CGLib采用动态创建子类的方式生成代理对象，所以不能对final修饰的类进行代理。

#### 2.14 JDK动态代理和CGLIB有什么区别？

**参考答案**

JDK动态代理

这是Java提供的动态代理技术，可以在运行时创建接口的代理实例。Spring AOP默认采用这种方式，在接口的代理实例中织入代码。

CGLib动态代理

采用底层的字节码技术，在运行时创建子类代理的实例。当目标对象不存在接口时，Spring AOP就会采用这种方式，在子类实例中织入代码。

#### 2.15 既然有没有接口都可以用CGLIB，为什么Spring还要使用JDK动态代理？

**参考答案**

在性能方面，CGLib创建的代理对象比JDK动态代理创建的代理对象高很多。但是，CGLib在创建代理对象时所花费的时间比JDK动态代理多很多。所以，对于单例的对象因为无需频繁创建代理对象，采用CGLib动态代理比较合适。反之，对于多例的对象因为需要频繁的创建代理对象，则JDK动态代理更合适。

#### 2.16 Spring如何管理事务？

**参考答案**

Spring为事务管理提供了一致的编程模板，在高层次上建立了统一的事务抽象。也就是说，不管是选择MyBatis、Hibernate、JPA还是Spring JDBC，Spring都可以让用户以统一的编程模型进行事务管理。

Spring支持两种事务编程模型：

1. 编程式事务

   Spring提供了TransactionTemplate模板，利用该模板我们可以通过编程的方式实现事务管理，而无需关注资源获取、复用、释放、事务同步及异常处理等操作。相对于声明式事务来说，这种方式相对麻烦一些，但是好在更为灵活，我们可以将事务管理的范围控制的更为精确。

2. 声明式事务

   Spring事务管理的亮点在于声明式事务管理，它允许我们通过声明的方式，在IoC配置中指定事务的边界和事务属性，Spring会自动在指定的事务边界上应用事务属性。相对于编程式事务来说，这种方式十分的方便，只需要在需要做事务管理的方法上，增加@Transactional注解，以声明事务特征即可。

#### 2.17 Spring的事务传播方式有哪些？

**参考答案**

当我们调用一个业务方法时，它的内部可能会调用其他的业务方法，以完成一个完整的业务操作。这种业务方法嵌套调用的时候，如果这两个方法都是要保证事务的，那么就要通过Spring的事务传播机制控制当前事务如何传播到被嵌套调用的业务方法中。

Spring在TransactionDefinition接口中规定了7种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时如何进行传播，如下表：

|       事务传播类型        |                             说明                             |
| :-----------------------: | :----------------------------------------------------------: |
|   PROPAGATION_REQUIRED    | 如果当前没有事务，则新建一个事务；如果已存在一个事务，则加入到这个事务中。这是最常见的选择。 |
|   PROPAGATION_SUPPORTS    |     支持当前事务，如果当前没有事务，则以非事务方式执行。     |
|   PROPAGATION_MANDATORY   |        使用当前的事务，如果当前没有事务，则抛出异常。        |
| PROPAGATION_REQUIRES_NEW  |        新建事务，如果当前存在事务，则把当前事务挂起。        |
| PROPAGATION_NOT_SUPPORTED |  以非事务方式执行操作，如果当前存在事务，则把当前事务挂起。  |
|     PROPAGATION_NEVER     |     以非事务方式执行操作，如果当前存在事务，则抛出异常。     |
|    PROPAGATION_NESTED     | 如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

#### 2.18 Spring的事务如何配置，常用注解有哪些？

**参考答案**

事务的打开、回滚和提交是由事务管理器来完成的，我们使用不同的数据库访问框架，就要使用与之对应的事务管理器。在Spring Boot中，当你添加了数据库访问框架的起步依赖时，它就会进行自动配置，即自动实例化正确的事务管理器。

对于声明式事务，是使用@Transactional进行标注的。这个注解可以标注在类或者方法上。

- 当它标注在类上时，代表这个类所有公共（public）非静态的方法都将启用事务功能。
- 当它标注在方法上时，代表这个方法将启用事务功能。

另外，在@Transactional注解上，我们可以使用isolation属性声明事务的隔离级别，使用propagation属性声明事务的传播机制。

#### 2.19 说一说你对声明式事务的理解

**参考答案**

Spring事务管理的亮点在于声明式事务管理，它允许我们通过声明的方式，在IoC配置中指定事务的边界和事务属性，Spring会自动在指定的事务边界上应用事务属性。相对于编程式事务来说，这种方式十分的方便，只需要在需要做事务管理的方法上，增加@Transactional注解，以声明事务特征即可。

## 3. Spring MVC

#### 3.1 什么是MVC？

**参考答案**

MVC是一种设计模式，在这种模式下软件被分为三层，即Model（模型）、View（视图）、Controller（控制器）。Model代表的是数据，View代表的是用户界面，Controller代表的是数据的处理逻辑，它是Model和View这两层的桥梁。将软件分层的好处是，可以将对象之间的耦合度降低，便于代码的维护。

#### 3.2 DAO层是做什么的？

**参考答案**

DAO是Data Access Object的缩写，即数据访问对象，在项目中它通常作为独立的一层，专门用于访问数据库。这一层的具体实现技术有很多，常用的有Spring JDBC、Hibernate、JPA、MyBatis等，在Spring框架下无论采用哪一种技术访问数据库，它的编程模式都是统一的。

#### 3.3 介绍一下Spring MVC的执行流程

**参考答案**

1. 整个过程开始于客户端发出的一个HTTP请求，Web应用服务器接收到这个请求。如果匹配DispatcherServlet的请求映射路径，则Web容器将该请求转交给DispatcherServlet处理。
2. DispatcherServlet接收到这个请求后，将根据请求的信息（包括URL、HTTP方法、请求报文头、请求参数、Cookie等）及HandlerMapping的配置找到处理请求的处理器（Handler）。可将HandlerMapping看做路由控制器，将Handler看做目标主机。值得注意的是，在Spring MVC中并没有定义一个Handler接口，实际上任何一个Object都可以成为请求处理器。
3. 当DispatcherServlet根据HandlerMapping得到对应当前请求的Handler后，通过HandlerAdapter对Handler进行封装，再以统一的适配器接口调用Handler。HandlerAdapter是Spring MVC框架级接口，顾名思义，HandlerAdapter是一个适配器，它用统一的接口对各种Handler方法进行调用。
4. 处理器完成业务逻 辑的处理后，将返回一个ModelAndView给DispatcherServlet，ModelAndView包含了视图逻辑名和模型数据信息。
5. ModelAndView中包含的是“逻辑视图名”而非真正的视图对象，DispatcherServlet借由ViewResolver完成逻辑视图名到真实视图对象的解析工作。
6. 当得到真实的视图对象View后，DispatcherServlet就使用这个View对象对ModelAndView中的模型数据进行视图渲染。
7. 最终客户端得到的响应消息可能是一个普通的HTML页面，也可能是一个XML或JSON串，甚至是一张图片或一个PDF文档等不同的媒体形式。

#### 3.4 说一说你知道的Spring MVC注解

**参考答案**

@RequestMapping：

作用：该注解的作用就是用来处理请求地址映射的，也就是说将其中的处理器方法映射到url路径上。

属性：

- method：是让你指定请求的method的类型，比如常用的有get和post。
- value：是指请求的实际地址，如果是多个地址就用{}来指定就可以啦。
- produces：指定返回的内容类型，当request请求头中的Accept类型中包含指定的类型才可以返回的。
- consumes：指定处理请求的提交内容类型，比如一些json、html、text等的类型。
- headers：指定request中必须包含那些的headed值时，它才会用该方法处理请求的。
- params：指定request中一定要有的参数值，它才会使用该方法处理请求。

@RequestParam：

作用：是将请求参数绑定到你的控制器的方法参数上，是Spring MVC中的接收普通参数的注解。

属性：

- value是请求参数中的名称。
- required是请求参数是否必须提供参数，它的默认是true，意思是表示必须提供。

@RequestBody：

作用：如果作用在方法上，就表示该方法的返回结果是直接按写入的Http responsebody中（一般在异步获取数据时使用的注解）。

属性：required，是否必须有请求体。它的默认值是true，在使用该注解时，值得注意的当为true时get的请求方式是报错的，如果你取值为false的话，get的请求是null。

@PathVaribale：

作用：该注解是用于绑定url中的占位符，但是注意，spring3.0以后，url才开始支持占位符的，它是Spring MVC支持的rest风格url的一个重要的标志。

#### 3.5 介绍一下Spring MVC的拦截器

**参考答案**

拦截器会对处理器进行拦截，这样通过拦截器就可以增强处理器的功能。Spring MVC中，所有的拦截器都需要实现HandlerInterceptor接口，该接口包含如下三个方法：preHandle()、postHandle()、afterCompletion()。

这些方法的执行流程如下图：

![img](img/web开发/springmvc-2.png)

通过上图可以看出，Spring MVC拦截器的执行流程如下：

- 执行preHandle方法，它会返回一个布尔值。如果为false，则结束所有流程，如果为true，则执行下一步。
- 执行处理器逻辑，它包含控制器的功能。
- 执行postHandle方法。
- 执行视图解析和视图渲染。
- 执行afterCompletion方法。

Spring MVC拦截器的开发步骤如下：

1. 开发拦截器：

   实现handlerInterceptor接口，从三个方法中选择合适的方法，实现拦截时要执行的具体业务逻辑。

2. 注册拦截器：

   定义配置类，并让它实现WebMvcConfigurer接口，在接口的addInterceptors方法中，注册拦截器，并定义该拦截器匹配哪些请求路径。

#### 3.6 怎么去做请求拦截？

**参考答案**

如果是对Controller记性拦截，则可以使用Spring MVC的拦截器。

如果是对所有的请求（如访问静态资源的请求）进行拦截，则可以使用Filter。

如果是对除了Controller之外的其他Bean的请求进行拦截，则可以使用Spring AOP。

## 4. MyBatis

#### 4.1 谈谈MyBatis和JPA的区别

**参考答案**

ORM映射不同：

MyBatis是半自动的ORM框架，提供数据库与结果集的映射；

JPA（默认采用Hibernate实现）是全自动的ORM框架，提供对象与数据库的映射。

可移植性不同：

JPA通过它强大的映射结构和HQL语言，大大降低了对象与数据库的耦合性；

MyBatis由于需要写SQL，因此与数据库的耦合性直接取决于SQL的写法，如果SQL不具备通用性而用了很多数据库的特性SQL的话，移植性就会降低很多，移植时成本很高。

日志系统的完整性不同：

JPA日志系统非常健全、涉及广泛，包括：SQL记录、关系异常、优化警告、缓存提示、脏数据警告等；

MyBatis除了基本的记录功能外，日志功能薄弱很多。

SQL优化上的区别：

由于Mybatis的SQL都是写在XML里，因此优化SQL比Hibernate方便很多。

而Hibernate的SQL很多都是自动生成的，无法直接维护SQL。虽有HQL，但功能还是不及SQL强大，见到报表等复杂需求时HQL就无能为力，也就是说HQL是有局限的Hhibernate虽然也支持原生SQL，但开发模式上却与ORM不同，需要转换思维，因此使用上不是非常方便。总之写SQL的灵活度上Hibernate不及Mybatis。

#### 4.2 MyBatis输入输出支持的类型有哪些？

**参考答案**

parameterType：

MyBatis支持多种输入输出类型，包括：

1. 简单的类型，如整数、小数、字符串等；
2. 集合类型，如Map等；
3. 自定义的JavaBean。

其中，简单的类型，其数值直接映射到参数上。对于Map或JavaBean则将其属性按照名称映射到参数上。

#### 4.3 MyBatis里如何实现一对多关联查询？

**参考答案**

一对多映射有两种配置方式，都是使用collection标签实现的。在此之前，为了能够存储一对多的数据，需要在主表对应的实体类中增加集合属性，用于封装子表对应的实体类。

嵌套查询：

1. 通过select标签定义查询主表的SQL，返回结果通过reusltMap进行映射。
2. 在resultMap中，除了映射主表属性，还要通过collection标签映射子表属性，该标签需包含如下内容：
   - 通过property属性指定子表属性名；
   - 通过javaType属性指定封装子表属性的集合类型；
   - 通过ofType属性指定子表的实体类型；
   - 通过select属性指定查询子表所依赖的SQL，这个SQL需单独定义，内部包含查询子表的语句。

嵌套结果：

1. 通过select标签定义关联查询主表和子表的SQL，返回结果通过resultMap进行映射。
2. 在resultMap中，除了映射主表属性，还要通过collection标签映射子表属性，该标签需包含如下内容：
   - 通过property属性指定子表属性名；
   - 通过ofType属性指定子表的实体类型；
   - 通过result子标签定义子表字段和属性的映射关系。

#### 4.4 MyBatis中的$和#有什么区别？

**参考答案**

使用#设置参数时，MyBatis会创建预编译的SQL语句，然后在执行SQL时MyBatis会为预编译SQL中的占位符（?）赋值。预编译的SQL语句执行效率高，并且可以防止注入攻击。

使用$设置参数时，MyBatis只是创建普通的SQL语句，然后在执行SQL语句时MyBatis将参数直接拼入到SQL里。这种方式在效率、安全性上均不如前者，但是可以解决一些特殊情况下的问题。例如，在一些动态表格（根据不同的条件产生不同的动态列）中，我们要传递SQL的列名，根据某些列进行排序，或者传递列名给SQL都是比较常见的场景，这就无法使用预编译的方式了。

#### 4.5 既然![img](img/web开发/equation)，什么时候会用到它？

**参考答案**

它可以解决一些特殊情况下的问题。例如，在一些动态表格（根据不同的条件产生不同的动态列）中，我们要传递SQL的列名，根据某些列进行排序，或者传递列名给SQL都是比较常见的场景，这就无法使用预编译的方式了。

#### 4.6 MyBatis的xml文件和Mapper接口是怎么绑定的？

**参考答案**

是通过xml文件中，`<mapper>` 根标签的namespace属性进行绑定的，即namespace属性的值需要配置成接口的全限定名称，MyBatis内部就会通过这个值将这个接口与这个xml关联起来。

#### 4.7 MyBatis分页和自己写的分页哪个效率高？

**参考答案**

自己写的分页效率高。

在MyBatis中，我们可以通过分页插件实现分页，也可以通过分页SQL自己实现分页。其中，分页插件的原理是，拦截查询SQL，在这个SQL基础上自动为其添加limit分页条件。它会大大的提高开发的效率，但是无法对分页语句做出有针对性的优化，比如分页偏移量很大的情况，而这些在自己写的分页SQL里却是可以灵活实现的。

#### 4.8 了解MyBatis缓存机制吗？

**参考答案**

MyBatis的缓存分为一级缓存和二级缓存。

一级缓存：

一级缓存也叫本地缓存，它默认会启用，并且不能关闭。一级缓存存在于SqlSession的生命周期中，即它是SqlSession级别的缓存。在同一个 SqlSession 中查询时，MyBatis 会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个SqlSession 中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当Map 缓存对象中己经存在该键值时，则会返回缓存中的对象。

二级缓存：

二级缓存存在于SqlSessionFactory 的生命周期中，即它是SqlSessionFactory级别的缓存。若想使用二级缓存，需要在如下两处进行配置。

在MyBatis 的全局配置settings 中有一个参数cacheEnabled，这个参数是二级缓存的全局开关，默认值是true ，初始状态为启用状态。

MyBatis 的二级缓存是和命名空间绑定的，即二级缓存需要配置在Mapper.xml 映射文件中。在保证二级缓存的全局配置开启的情况下，给Mapper.xml 开启二级缓存只需要在Mapper. xml 中添加如下代码：

```xml
<cache />
```

二级缓存具有如下效果：

- 映射语句文件中的所有SELECT 语句将会被缓存。
- 映射语句文件中的所有时INSERT 、UPDATE 、DELETE 语句会刷新缓存。
- 缓存会使用Least Rece ntly U sed ( LRU ，最近最少使用的）算法来收回。
- 根据时间表（如no Flush Int erv al ，没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储集合或对象（无论查询方法返回什么类型的值）的1024 个引用。
- 缓存会被视为read/write（可读／可写）的，意味着对象检索不是共享的，而且可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

## 5. 其他

#### 5.1 cookie和session的区别是什么？

**参考答案**

1. 存储位置不同：cookie存放于客户端；session存放于服务端。
2. 存储容量不同：单个cookie保存的数据<=4KB，一个站点最多保存20个cookie；而session并没有上限。
3. 存储方式不同：cookie只能保存ASCII字符串，并需要通过编码当时存储为Unicode字符或者二进制数据；session中能够存储任何类型的数据，例如字符串、整数、集合等。
4. 隐私策略不同：cookie对客户端是可见的，别有用心的人可以分析存放在本地的cookie并进行cookie欺骗，所以它是不安全的；session存储在服务器上，对客户端是透明的，不存在敏感信息泄露的风险。
5. 生命周期不同：可以通过设置cookie的属性，达到cookie长期有效的效果；session依赖于名为JSESSIONID的cookie，而该cookie的默认过期时间为-1，只需关闭窗口该session就会失效，因此session不能长期有效。
6. 服务器压力不同：cookie保存在客户端，不占用服务器资源；session保管在服务器上，每个用户都会产生一个session，如果并发量大的话，则会消耗大量的服务器内存。
7. 浏览器支持不同：cookie是需要浏览器支持的，如果客户端禁用了cookie，则会话跟踪就会失效；运用session就需要使用URL重写的方式，所有用到session的URL都要进行重写，否则session会话跟踪也会失效。
8. 跨域支持不同：cookie支持跨域访问，session不支持跨域访问。

#### 5.2 cookie和session各自适合的场景是什么？

**参考答案**

对于敏感数据，应存放在session里，因为cookie不安全。

对于普通数据，优先考虑存放在cookie里，这样会减少对服务器资源的占用。

#### 5.3 请介绍session的工作原理

**参考答案**

session依赖于cookie。

当客户端首次访问服务器时，服务器会为其创建一个session对象，该对象具有一个唯一标识SESSIONID。并且在响应阶段，服务器会创建一个cookie，并将SESSIONID存入其中。

客户端通过响应的cookie而持有SESSIONID，所以当它再次访问服务器时，会通过cookie携带这个SESSIONID。服务器获取到SESSIONID后，就可以找到与之对应的session对象，进而从这个session中获取该客户端的状态。

#### 5.4 get请求与post请求有什么区别？

**参考答案**

- GET在浏览器回退时是无害的，而POST会再次提交请求。
- GET产生的URL地址可以被Bookmark，而POST不可以。
- GET请求会被浏览器主动cache，而POST不会，除非手动设置。
- GET请求只能进行url编码，而POST支持多种编码方式。
- GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
- GET请求在URL中传送的参数是有长度限制的，而POST没有。
- 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
- GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
- GET参数通过URL传递，POST放在Request body中。

#### 5.5 get请求的参数能放到body里面吗？

**参考答案**

GET请求是可以将参数放到BODY里面的，官方并没有明确禁止，但给出的建议是这样不符合规范，无法保证所有的实现都支持。这就意味着，如果你试图这样做，可能出现各种未知的问题，所以应该当避免。

#### 5.6 post不幂等是为什么？

**参考答案**

HTTP方法的幂等性是指一次和多次请求某一个资源应该具有同样的副作用。幂等性属于语义范畴，正如编译器只能帮助检查语法错误一样，HTTP规范也没有办法通过消息格式等语法手段来定义它。

POST所对应的URI并非创建的资源本身，而是资源的接收者。比如：POST `http://www.forum.com/articles`的语义是在`http://www.forum.com/articles`下创建一篇帖子，HTTP响应中应包含帖子的创建状态以及帖子的URI。两次相同的POST请求会在服务器端创建两份资源，它们具有不同的URI。所以，POST方法不具备幂等性。

#### 5.7 页面报400错误是什么意思？

**参考答案**

400状态码标识请求的语义有误，当前请求无法被服务器理解。除非进行修改，否则客户端不应该重复提交这个请求。通常情况下，是本次请求中包含有错误的参数，此时应该排查前端传递的参数。

#### 5.8 请求数据出现乱码该怎么处理？

**参考答案**

服务端出现请求乱码的原因是，客户端编码与服务器解码方案不一致，可以有如下几种解决办法：

1. 将获得的数据按照客户端编码转成BYTE，再将BYTE按服务端编码转成字符串，这种方案对各种请求方式均有效，但是十分的麻烦。
2. 在接受请求数据之前，显示声明实体内容的编码与服务器一致，这种方式只对POST请求有效。
3. 修改服务器的配置文件，显示声明请求路径的编码与服务器一致，这种方式只对GET请求有效。

#### 5.9 如何在SpringBoot框架下实现一个定时任务？

**参考答案**

Spring给我们提供了可执行定时任务的线程池ThreadPoolTaskScheduler，该线程池提供了多个可以与执行定时任务的方法，如下图。在Spring Boot中，只需要在配置类中启用线程池注解，就可以直接使用这个线程池了。

![img](img/web开发/schedule.png)

#### 5.10 调用接口时要记录日志，该怎么设计？

**参考答案**

可以定义一个记录日志的组件，并通过AOP将其织入到这个接口的调用中。这种方式对接口无需做任何改造，业务代码中也无需增加任何调用的逻辑，完美地消除了记录日志和业务代码的耦合度。

#### 5.11 了解Spring Boot JPA吗？

**参考答案**

JPA即Java Persistence API，它是一个基于O/R映射的标准规范。也就是说它指定以了标准规则，不提供实现，软件提供商可以按照标准规范来实现，而使用者只需按照规范中定义的方式来使用，不用和软件提供商打交道。JPA主要实现有Hibernate、EclipseLink、OpenJPA等，我们使用JPA来开发，无论是采用哪一种实现方式都一样。