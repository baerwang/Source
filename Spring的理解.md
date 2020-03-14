> Spring IOC 和 AOP 的理解

### IOC

IOC(Inverse of Control:控制反转) 是一种设计思想，就是将原本在程序中手动创建对象的控制权，交由Spring框架来管理。IOC在其他语言也有应用，并非Spring持有。IOC容器时Spring用来实现IOC的载体，IOC容器实际上就是Map(Key , Value)，Map中存放的时各种对象。

将对象之间的相互依赖关系交给IOC容器来管理，并由IOC容器完成对象的注入。这样可以很大程序上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好文件/注解即可，完全不用考虑对象是如何被创建出来的。在实际项目中一个Service类可能有几百个甚至上千个类作为它的底层，加入我们需要实例化这个service，你可能要每次都要搞清这个Service所有底层类的构造函数，会把人逼疯，如果利用IOC你只需要配置好，然后再需要的地方引用就行了，大大增加了项目的可维护性降低了开发难度。

阅读：https://www.zhihu.com/question/23277575/answer/169698662

### AOC

AOC(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任(例如事务处理，日志管理，权限管理)封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。

Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用JDK Proxy去进行代理，这时候Spring AOP会使用CGLib，这时候Spring AOP会使用CGLib生成一个被代理对象的子类来作为代理。

![](D:\Software\data\md\images\spring\1584023156.jpg)

你也可以使用AspectJ，Spring AOP已经继承了AspectJ，AspectJ应该算的上是Java生态系统中最完整的AOP框架了。

使用AOP之后我们可以把一些通用的功能抽象出来，再需要用到的地方直接使用即可，这样大大简化了代码量。需要增加新功能也比较方便，也提高了系统扩展性。日志功能，事务管理等等场景都用到了AOP。

> Spring AOP 和 AspectJ AOP有什么区别

Spring AOP属于运行时增强，而AspectJ时编译时增强。Spring AOP基于代理(Proxying)，而AspectJ基于字节码操作(ByteCode Manipulation)。

Spring AOP已经集成了AspectJ，AspectJ应该算的上时Java生态中最完整的AOP框架了。AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单，如果我们切面比较少，那么两者性能差异不大，但是切面太多，最好选择AspectJ，它比Spring AOP快很多。

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

- REQUIRED：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
- SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式运行。
- MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常。
- REQUIRES_NEW：新建事务，如果当前存在事务，就把当前事务挂起。
- NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
- NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与REQUIRED类似的操作。

### 常用的主要有Required，RequiresNew，Nested三种

- Required：简单理解就是事务方法会判断是否存在事务，有事务就用已有的，没有就重新开启一个。
- RequiresNew：简单理解就是开启新事务，若当前已有事务，挂期当前事务。新开启的事务和之前的事务无关，拥有自己的锁和隔离级别，可以独立提交和回滚，内层事务执行期间，外层事务挂起，内层事务执行完成后，外层事务恢复执行。
- Nested：简单理解就是嵌套事务，如果外部事务回滚，则嵌套事务也会回滚。外部事务提交的时候，嵌套它才会被提交。嵌套事务回滚不会影响外部事务。子事务是上层事务的嵌套事务，在子事务执行之前会简历savepoint，嵌套事务的回滚会回到这个savepoint，不会造成父事务的回滚。

如果想事务一起执行可以用Required满足大部分场景，如果不想让执行的子事务的结果影响到父事务的提交可以将子事务设置为RequiresNew。