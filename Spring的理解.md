> 
>
> Spring IOC 和 AOP 的理解

### IOC

> 简洁版

IOC(Inverse of Control:控制反转) 是Spring中一个非常重要得概念，它不是什么技术，而是一种解耦的设计思想。它的主要目的时借助于"第三方"(Spring中的IOC容器)实现具有依赖关系的对象之间的解耦(IOC容易管理对象，你只管使用即可)，从而降低代码之间的耦合度。**IOC是一个原则，而不是一个模式。**

**SpringIOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。**IOC容器负责创建对象，将对象连接在一起，配置这些对象，并从创建中处理这些对象的整个生命周期，直到它们被完全销毁。

> 复杂版

IOC(Inverse of Control:控制反转) 是一种设计思想，就是将原本在程序中手动创建对象的控制权，交由Spring框架来管理。IOC在其他语言也有应用，并非Spring持有。IOC容器时Spring用来实现IOC的载体，IOC容器实际上就是Map(Key , Value)，Map中存放的时各种对象。

将对象之间的相互依赖关系交给IOC容器来管理，并由IOC容器完成对象的注入。这样可以很大程序上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好文件/注解即可，完全不用考虑对象是如何被创建出来的。在实际项目中一个Service类可能有几百个甚至上千个类作为它的底层，加入我们需要实例化这个service，你可能要每次都要搞清这个Service所有底层类的构造函数，会把人逼疯，如果利用IOC你只需要配置好，然后再需要的地方引用就行了，大大增加了项目的可维护性降低了开发难度。

阅读：https://www.zhihu.com/question/23277575/answer/169698662

### AOP

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任(例如事务处理，日志管理，权限管理)封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。

Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用JDK Proxy去进行代理，这时候Spring AOP会使用CGLib，这时候Spring AOP会使用CGLib生成一个被代理对象的子类来作为代理。

![](D:\Software\data\md\images\spring\1584023156.jpg)

你也可以使用AspectJ，Spring AOP已经继承了AspectJ，AspectJ应该算的上是Java生态系统中最完整的AOP框架了。

使用AOP之后我们可以把一些通用的功能抽象出来，再需要用到的地方直接使用即可，这样大大简化了代码量。需要增加新功能也比较方便，也提高了系统扩展性。日志功能，事务管理等等场景都用到了AOP。

> Spring AOP 和 AspectJ AOP有什么区别

Spring AOP属于运行时增强，而AspectJ时编译时增强。Spring AOP基于代理(Proxying)，而AspectJ基于字节码操作(ByteCode Manipulation)。

Spring AOP已经集成了AspectJ，AspectJ应该算的上时Java生态中最完整的AOP框架了。AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单，如果我们切面比较少，那么两者性能差异不大，但是切面太多，最好选择AspectJ，它比Spring AOP快很多。

> DI(依赖注入)

**DI(Dependecy Inject，依赖注入)**是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去。

> Spring的理解，AOP和IOC在项目里是怎么用的

​		Spring是一个开源框架，处于MVC模式中的控制层。它能应对需求快速的变化，其主要原因它又一种面向切面编程(AOP)的优势，其次它提升了系统性能，因为通过依赖倒置机制(IOC)，系统中用到的对象不是在系统加载时就全部实例化，而是在调用这个类才会实例化该类的对象，从而提升了系统性能。

### Spring的优点

- 降低了组件之间的耦合性，实现了软件各层之间的解耦。
- 可以提供容易提供的众多服务，如事务管理，消息服务，日志记录等。
- 容器提供了AOP技术，利用它很容易实现权限拦截，运行期监控等功能。

Spring中AOP技术是设计模式中的动态代理模式，只需实现jdk提供的动态代理接口InvocationHandler，所有被代理对象的方法都由InvocationHandler接管实际的处理任务。

面向切面编程中还要理解切入点，切面，通知，织入等概念。

> AOP的两种实现方式，并且说一下哪一个效率更高一些，为什么。

两种方式：一种是JDK动态代理，另一种是CGLib的方式。

### JDK动态代理具体实现原理：

- 通过InvocationHandlet接口创建自己的调用处理器。
- 通过Proxy类指定ClassLoader对象和一组interface来创建动态代理。
- 通过反射机制获取动态代理类的构造函数，其唯一参数类型就是调用处理器接口类型。
- 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数参入。

### CGLib动态代理：

CGLib时一个强大，高性能的Code生产类库，可以实现运行期动态扩展Java类，Spring在运行期间通过CGLib继承要被动态代理的类，重写父类的方法，实现AOP面向切面编程。

### 两者对比：

JDK动态代理时面向接口的。

CGLIb动态代理时通过字节码底层继承要代理类来实现的(如果被代理类被final关键字所修饰，那么就会失败)。

### 性能：

关于两者之间的性能的话，JDK动态代理所创建的代理对象，在以前JDK版本中，性能并不是很高，虽然在高版本中JDK动态代理对象的性能得到了很大的提升，但是它也并不是适用于所有的场景。

主要体现如下的两个指标中：

- CGLIb所创建的动态代理对象在实际运行时候的性能要比JDK动态代理要高不少，由研究表明，大概要高十倍。
- 但是CGLib在创建对象的时候所花费的时间却比JDK动态代理要多很多，有研究表明，大概有八倍的差距。
- 因此，对于singleton的代理对象或者具有实例池的代理，因为无需频繁的创建代理对象，所以比较合适采用CGLib动态代理，反正比较合用于JDK动态代理。

> Spring中的bean作用域有那些

- singleton：唯一bean实例，Spring中的bean默认都是单例的。
- prototype：每次请求都会创建一个新的bean实例。
- request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP session内有效。
- global-session：全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet时能够生成语义代码(例如：HTML)片段的小型Java Web插件。他们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与servlet不同，每个portlet都有不同的会话。

> Spring中的单例bean的线程安全问题了解吗

大部分时候我们并没有在系统中使用多线程，所以很少有人去回关注这个问题。单例bean存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

- 在bean对象中尽量避免定义可变的成员变量
- 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中。

> Spring中的bean声明周期

这部分网上有很多文章都讲到了，下面的内容整理自：https://yemengying.com/2016/07/14/spring-bean-life-cycle/ ，除了这篇文章，再推荐一篇很不错的文章 ：https://www.cnblogs.com/zrtqsk/p/3735273.html 。

- Bean 容器找到配置文件中的Spring Bean 的定义

- B容器利用Java Reflection API创建一个Bean的实例

- 如果涉及到一些属性值利用set()方法设置一些属性值

- 如果Bean实现了BeanNameAware接口，调用setBeanName()，传入Bean的名字

- 如果Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例

- 如果Bean实现了BeanFactoryAware接口，调用了setBeanClassLoader()方法，传入ClassLoader对象的实例

- 于上面的类似，如果实现了其他的 *.Aware接口，就调用相应的方法

- 如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessBeforeInitialization()方法

- 如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法

- 如果Bean在配置文件中的定义包含init-method属性，执行指定的方法

- 如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessAfterInitialization()方法

- 当要销毁Bean的时候，如果Bean实现了DisposableBean接口，执行destroy()方法

- 当要销毁Bean的时候，如果Bean在配置文件中定义包含destroy-method属性，执行指定的方法

  ![Spring Bean 生命周期](D:\Software\data\md\images\2020313\202003141211.png)

> SpringMVC 了解

- **Model1 时代** : 很多学 Java 后端比较晚的朋友可能并没有接触过 Model1 模式下的 JavaWeb 应用开发。在 Model1 模式下，整个 Web 应用几乎全部用 JSP 页面组成，只用少量的 JavaBean 来处理数据库连接、访问等操作。这个模式下 JSP 即是控制层又是表现层。显而易见，这种模式存在很多问题。比如①将控制逻辑和表现逻辑混杂在一起，导致代码重用率极低；②前端和后端相互依赖，难以进行测试并且开发效率极低；
- 学过 Servlet 并做过相关 Demo 的朋友应该了解“Java Bean(Model)+ JSP（View,）+Servlet（Controller） ”这种开发模式,这就是早期的 JavaWeb MVC 开发模式。Model:系统涉及的数据，也就是 dao 和 bean。View：展示模型中的数据，只是用来展示。Controller：处理用户请求都发送给 ，返回数据给 JSP 并展示给用户。

Model2模式下还存在很多问题，Model2的抽象和封装程序远远不够，使用Model2进行开发时候不可避免重复造轮子，这就大大降低了程序的可维护性和复用性。于是很多Java Web开发相关的MVC框架应该运而生比如Struts2，但是Struts2比较笨重，随着Spring轻量级开发框架的流行，Spring生态出现了Spring MVC框架，SpringMVC是当前最优秀的MVC框架，相当于Struts2，Spring MVC使用更加简单和方便，开发效率更高，并且Spring MVC运行效率比较快。

MVC是一种设置模式，Spring MVC是一款很优秀的MVC框架。Spring MVC可以帮助我们进行更简洁的Web层的开发，并且它天生于Spring框架集成。Spring MVC下我们一般把后端项目分为Service层(处理业务)，Dao层(数据库操作)，Entiry层(实体类)，Controller层(控制层，返回数据给前台页面)。

### Spring MVC 简单原理图：

![](D:\Software\data\md\images\2020313\20200314130054.png)

> Spring MVC 工作原理了解吗

### 原理如下图所示：

![](D:\Software\data\md\images\2020313\20200314130225.png)

### 流程过程

1. 用户发送请求给前端控制器DispatcherServlet。
2. DispatcherServlet接受到用户请求后，通过调用HandlerMapping处理器映射器录找合适的处理器进行处理。
3. 处理器映射器找到具体的处理器后，生成处理器对象以及处理拦截器，并返回给前端控制器。
4. 前端控制器通过调用处理器适配器，将请求交给处理器进行处理。
5. 处理器适配调用具体的处理器对请求进行处理。
6. 处理器处理完成后返回ModelAndView。
7. 处理器适配器将处理器返回ModelAdnView返回给前端控制器。
8. 前端控制器将ModelAndView交给视图解析器进行解析。
9. 视图解析器解析之后返回给具体的View。
10. 前端控制器根据具体的View进行渲染。
11. 前端控制器响应用户请求。

> Spring 框架中用到了那些设计模式

- 工厂设计模式：Spring使用工厂模式通过 **BeanFactory**，**ApplicationContext**创建bean对象。
- 代理设计模式：SpringAOP功能的实现。
- 单例设计模式：Spring中的Bean默认都是单例的。
- 模板方法模式：Spring中的 **jdbcTemplate**，**hibernateTemplate**等以Template结尾的对数据库操作的类，它们就是用到了模板模式。
- 包装器设计模式：我们的项目需要连接多个数据库，而且不同的客户再每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- 观察者模式：Spring事件驱动模式就是观察者模式很经典的一个应用。
- 适配器模式：Spring AOP的增强或通知(Advice)使用到了适配器模式，Spring MVC中也是用到了适配器模式适配Controller。
- ……

> 单例设计模式

在我们的系统中，有一些对象其实我们只需要一个，比如：线程池，缓存，对话框，注册表，日志对象，充当打印机，显卡等设备驱动程序的对象。事实上，这一类对象只能有一个实例，如果制造出多个实例就可能会导致一些问题的产生，比如：程序的行为异常，资源使用过量，或者不一致性的结果。

### 使用单例模式的好处：

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销。
- 由于new操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻GC压力，缩短GC停顿时间。

> 观察者模式

观察者模式是一种对象行为性模式。它表示的是一种对象于对象之间具有依赖关系，当一个对象发生改变的时候，这个对象所依赖的对象也会做出反应。Spring事件驱动模式就是观察者模式很经典的一个应用。Spring事件驱动模式非常有用，在很多场景都可以解耦我们的代码。比如我们每次添加商品的时候都需要重新更新商品索引，这个时候就可以利用观察者模式来解决这个问题。

### Spring事件驱动模式中的三种角色

#### 事件角色

**ApplicationEvent**(org.springframework.context包下)充当事件的角色，这是一个抽象类，它继承了**java.util.EventObject**并实现了**java.ioSerializable**接口。

Spring种默认存在以下事件，它们都是对 **ApplicationContextEvent**的实现(继承自**ApplicationContextEvent**)

- **ContextStartedEvent**：**ApplicationContext**启动后触发的事件；
- **ContextStoppedEvent**：**ApplicationContext**停止后触发的事件；
- **ContextRefreshedEvent**：**ApplicationContext**初始化或刷新完成后触发的事件；
- **ContextClosedEvent**：**ApplicationContext**关闭后触发的事件。

![](D:\Software\data\md\images\2020313\20200314210912.png)

### 事件监听者角色

**ApplicationListener**充当了事件监听者角色，它是一个接口，里面只定义了一个 **onApplicationEvent()**方法来处理**ApplicationEvent**。**ApplicationListener**接口类源码如下，可以看出接口定义看出接口种的事件只要实现了 **ApplicationEvent**就可以了，所以在Spring中我们只要下实现 **ApplicationListener**接口实现 **onApplicationEvent()**方法即可完成监听事件。

```java
package org.springframework.context;

import java.util.EventListener;

@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E var1);
}
```

### 事件发布者角色

**ApplicationEventPublisher**充当了事件的发布者，它也是一个接口。

```java
package org.springframework.context;

@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        this.publishEvent((Object)event);
    }

    void publishEvent(Object var1);
}
```

**ApplicationEventPublisher**接口中的**publishEvent()**这个方法在 **AbstractApplicationContext**类中被实现，阅读这个方法的实现，你会发现实际上事件真正是通过 **ApplicationEventMulticaster**来广播出去的。

> 将一个类声明为Spring的bean的注解有那些

我们一般使用的 **@Autowired**注解自动装配bean，要想把类标识成可用于 **@Autowired**注解自动装配的bean类，采用一下注解可实现：

- @Component：通用的注解，可标注任意类为Spring组件。如果一个Bean不知道属于那个层，可以使用@Component注解标注。
- @Repository：对应持久层即Dao层，主要用于数据库相关操作。
- @Service：对应服务层，主要涉及一些复杂的逻辑，需要用到Dao层。
- @Controller：对应Spring MVC控制层，主要用户接受用户请求并调用Service层返回数据给前端页面。

> 工厂设计模式

Spring使用工厂模式可以通过 **BeanFactory**或 **ApplicationContext**创建bean对象。

### 两者对比：

- **BeanFactory**：延迟注入(使用到某个bean的时候才会注入)，相比于 **BeanFactory**来说会占用更少的内存，程序驱动速度更快。
- **ApplicationContext**：容器启动的时候，不管你用没用到，一次性创建所有bean。**BeanFactory**仅提供了最基本的依赖注入支持，**ApplicationContext**扩展了 **BeanFactory**，除了有 **BeanFactory**的功能还有额外更多功能，所以一般开发人员使用 **ApplicationContext**会更多。

ApplicationContext的三个实现类：

1. **ClassPathXmlApplication**：把上下文文件当成类路径资源。
2. **FileSystemXmlApplication**：从文件系统中的XML文件载入上下文定义信息。
3. **XmlWebApplicationContext**：从Web系统中的XML文件载入上下文定义信息。

### Spring事件的流程总结

1. 定义一个事件：实现一个继承自ApplicationEvent，并且写相应的构造函数。
2. 定义一个事件监听者：实现ApplicationListener接口，重写onApplicationEvent()方法。
3. 使用事件发布者发布消息：可以通过ApplicationEventPublisher的publishEvent()发布消息。

Example：

```java
import org.springframework.context.ApplicationEvent;

/**
 * 定义一个事件，继承ApplicationEvent并且写相应的构造函数
 *
 * @author baerwang
 * @since 2020/3/14 21:00
 */
public class DemoEvent extends ApplicationEvent {

    private static final long serialVersionUID = 1L;

    private String message;

    public DemoEvent(Object source, String message) {
        super(source);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}
```

```java
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

/**
 * 定义一个事件监听者，实现ApplicationListener接口，重写onApplicationEvent
 * @author baerwang
 * @since 2020/3/14 21:18
 */
@Component
public class DemoListener implements ApplicationListener<DemoEvent> {
    /**
     * 接受消息
     * @param demoEvent
     */
    @Override
    public void onApplicationEvent(DemoEvent demoEvent) {
        System.out.println(demoEvent.getMessage());
    }
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Service;

/**
 * 发布事件，可以通过ApplicationEventPublisher的publishEvent()方法发布消息
 * @author baerwang
 * @since 2020/3/14 21:20
 */
@Service
public class DemoPublisher {

    @Autowired
    ApplicationContext applicationContext;

    public void publish(String message) {
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 定义一个控制层来模拟发送消息，我这里使用的是springboot
 *
 * @author baerwang
 * @since 2020/3/14 22:10
 */
@RestController
public class DemoController {

    @Autowired
    private DemoPublisher demoPublisher;

    @GetMapping("/hello")
    public String hello(String msg) {
        demoPublisher.publish(msg);
        return msg;
    }
}
```

当调用控制层的hello，调用demoPublisher.publish()方法的时候，比如demoPublisher.publish("张三")，控制台就会打印出 ：接收到信息是：张三。

> 适配器模式

适配器模式(Adpater Pattern)将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。

### Spring AOP中的适配器模式

我们知道Spring AOP的实现是基于代理模式，但是Spring AOP的增强或通知(Advice)使用到了适配器模式，与之相关的接口是**AdvisorAdpater**。Advice常用的类型有：BeforeAdvice(目标方法调用前，前置通知)，**AfterAdvice**(目标方法调用后，后置通知)，**AfterReturningAdvice**(目标方法执行结束后，return之前)等等。每个类型Advice(通知)，都有对应的拦截器：**MethodBeforeAdviceInterceptor**，**AfterReturningAdviceAdapter**，**AfterReturningAdviceInterceptor**。Spring预定义的通知要通过对应的适配器，适配成 **MethodInterceptor**接口(方法拦截器)类型的对象(如：**MethodBeforeAdviceInterceptor**负责适配 **MethodBeforeAdvice**)。

### Spring MVC中的适配器

在Spring MVC中，**DispatcherServlet**根据请求信息调用 **HandlerMapping**，解析请求对应的 **Handler**。解析对应的 **Handler**(也就是我们常说的 **Controller**控制器)后，开始由 **HandlerAdapter**适配器处理。**HandlerAdapter**作为期望接口，具体的适配器实现类用于目标类进行适配，**Controller**作为需要适配的类。

**为什么要在Spring MVC中使用适配器模式** Spring MVC中的 **Controller种类众多**，不同类型的 **Controller**通过不同的方法来对请求进行处理。如果不利用适配器模式的话，**DispatcherServlet**直接获取对应类型的 **Controller**，需要的自行来判断，像下main这段代码一样：

```java
if(mappedHandler.getHandler() instanceof MultiActionController){  
   ((MultiActionController)mappedHandler.getHandler()).xxx  
}else if(mappedHandler.getHandler() instanceof XXX){  
    ...  
}else if(...){  
   ...  
}  
```

推荐阅读适配器模式：https://www.cnblogs.com/mingmingcome/p/9810731.html

> 装饰者模式

装饰者模式可以动态地给对象添加一些额外得属性或行为。相比于使用继承，装饰者模式更加灵活。简单说就是当我们需要修改原由得功能，但我们不愿意直接去修改原有得代码时，设计一个Decorator套在原有代码外面。其实在JDK中由很多地方用到了装饰者模式，比如InputStream家族，InputStream类下由FileInputStream(读取文件)，BufferedInputStream(增加缓存，使读取文件速度大大提升)等子类都在不修改InputStream代码的情况下扩展了它的功能。

Spring中的配置DataSource的时候，DataSource可能是不同的数据库和数据源。我们能否根据客户的需求在修改原有类的动态切换不同的数据源？这个时候就要用到装饰者模式。Spring中用到的包装器模式在类名上含有 **Wrapper** 或者 **Decorator**。这些类基本上都是动态地给一个对象添加一些额外的职责。

> Spring 管理事务的方式有几种

1. 编程式事务吗，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置。(推荐使用)

### 声明式事务又分为两种：

1. 基于XML的声明式事务
2. 基于注解的声明式事务

> Spring事务中的隔离级别有哪几种

### TransactionDefinition接口中定义了五个表示隔离级别的常量

- TransactionDefinition.ISOLATION_DEFAULT：使用后端数据库默认的隔离级别，MySQL默认采用的REPEATABLE_READ隔离级别，Oracle默认采用的READ_COMMITED隔离级别。
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读，幻读或不可重复读。
- TransactionDefinition.ISOLATION_READ_COMMITTED：允许读取并发事务以提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
- TransactionDefinition.ISOLATION_REPEATABLE_READ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务所自己修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- TransactionDefinition.ISOLATION_SERIALIZABLE：最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读，不可重复读以及幻读。但是这将严重影响程序的性能，通常情况下也不会用到该级别。

> Spring的事务传播机制

### 传播事务有如下几种：

```java
public interface TransactionDefinition {
     int PROPAGATION_REQUIRED = 0;
     int PROPAGATION_SUPPORTS = 1;
     int PROPAGATION_MANDATORY = 2;
     int PROPAGATION_REQUIRES_NEW = 3;
     int PROPAGATION_NOT_SUPPORTED = 4;
     int PROPAGATION_NEVER = 5;
     int PROPAGATION_NESTED = 6;
}
```

### 支持当前事务的情况：

- REQUIRED：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
- SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式运行。
- MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常。

### 不支持当前事务的情况：

- REQUIRES_NEW：新建事务，如果当前存在事务，就把当前事务挂起。
- NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

### 其他情况：

- NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与REQUIRED类似的操作。

### 常用的主要有Required，RequiresNew，Nested三种

- Required：简单理解就是事务方法会判断是否存在事务，有事务就用已有的，没有就重新开启一个。
- RequiresNew：简单理解就是开启新事务，若当前已有事务，挂期当前事务。新开启的事务和之前的事务无关，拥有自己的锁和隔离级别，可以独立提交和回滚，内层事务执行期间，外层事务挂起，内层事务执行完成后，外层事务恢复执行。
- Nested：简单理解就是嵌套事务，如果外部事务回滚，则嵌套事务也会回滚。外部事务提交的时候，嵌套它才会被提交。嵌套事务回滚不会影响外部事务。子事务是上层事务的嵌套事务，在子事务执行之前会简历savepoint，嵌套事务的回滚会回到这个savepoint，不会造成父事务的回滚。

如果想事务一起执行可以用Required满足大部分场景，如果不想让执行的子事务的结果影响到父事务的提交可以将子事务设置为RequiresNew。

> @Transactional(rollbackFor = Exception.class)

我们知道：Exception分为运行时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。

当**@Transactional**注解作用于类上，该类的所有public方法将都具有该类型的事务属性，同时，我们也可以方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在 **@Transactional** 注解中如果不配置 **rollbackFor** 属性，那么事务只会在遇到RuntimeException的时候才会回滚，加上rollbackFor = Exception.class，可以让事务在遇到非运行时异常也回滚。

解读摘自其他博客：

https://mp.weixin.qq.com/s/wcK2qsZxKDJTLIGqEIyaNg