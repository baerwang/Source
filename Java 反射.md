# Java 反射



Java 反射机制是在运行状态中，对于任意一个实体类，都能够知道这个类的所有属性和方法。对于任意一个对象，能够调用它的任意方法和属性。这种动态获取信息以及调用对象方法的功能称为Java语言的反射机制。

反射：框架设计的灵魂
	反射：半成品软件。可以在框架的基础上进行软件开发，简化编码。
	反射：将类的各个组成部分封装为其他对象，这就是反射机制。
		好处：
			1.可以在程序运行过程中，操作这些对象。
			2.可以解耦，复用性，提高程序的可扩展性。
	获取Class对象的方式:
		1.Class.forName("类名"):将字节码文件加载进内存，返回Class对象。
			多用于配置文件，将类名定义在配置文件中。读取文件，加载类。
		2.类名.class:通过类名的属性class获取。
			多用于参数的传递。
		3.对象.getClass():getClass()方法在Object类中定义着。
			多用于对象的获取字节码的方式。
		结论：
			同一个字节码文件(*.class)再一次程序运行过程中，只会被加载一次，不论通过哪一种。方法获取的Class对象都是同一个。

# Java 在计算机中经历的三个阶段

![](D:\images\2020-2-1\程序在计算机中的三个阶段.png)

# Class

**Class 代表类的实体**，在运行的Java应用程序中表示类和接口。在这个类中提供了很多有用的方法，这里对他们简单的分类介绍。

- 获得类的相关的方法

| 方法                       | 用途                                                   |
| -------------------------- | ------------------------------------------------------ |
| asSubclass(Class<U> clazz) | 把传递的类的对象转换成代表其子类的对象                 |
| getClassLoader()           | 获得类的加载器                                         |
| getClasses()               | 返回一个数组，数组中包含该类中所有公共类和接口类的对象 |
| getDeclaredClasses()       | 返回一个数组，数组中包含该类中所有类和接口类的对象     |
| forName(String className)  | 根据类名返回类的对象                                   |
| getName()                  | 获得类的完整路径名字                                   |
| newInstance()              | 创建类的实例                                           |
| getPackage()               | 获得类的包                                             |
| getSimpleName()            | 获得类的名字                                           |
| getSuperclass()            | 获得当前类继承的父类的名字                             |
| getInterfaces()            | 获得当前类实现类或是接口                               |
| .class                     | 获取当前对象的类                                       |

- 获得类中字段相关的方法

| 方法                          | 用途                   |
| ----------------------------- | ---------------------- |
| getField(String name)         | 获得某个共有的字段对象 |
| getDeclaredField(String name) | 获得某个字段对象       |
| getDeclaredFields()           | 获得所有字段对象       |

-  获得类中注解相关的方法

| 方法                                    | 用途                                   |
| --------------------------------------- | -------------------------------------- |
| getAnnotation(Class<A> annotationClass) | 返回该类中与参数类型匹配的共有注解对象 |

- 获得类中构造器相关的方法

| 方法                                               | 用途                                   |
| -------------------------------------------------- | -------------------------------------- |
| getConstructor(Class...<?> parameterTypes)         | 获得该类中与参数类型匹配的共有构造方法 |
| getConstructors()                                  | 获得该类的所有共有构造方法             |
| getDeclaredConstructor(Class...<?> parameterTypes) | 获得该类中与参数类型匹配的构造方法     |
| getDeclaredConstructors()                          | 获得该类所有构造方法                   |

- 获得类中方法相关的方法

| 方法                                                      | 用途                   |
| --------------------------------------------------------- | ---------------------- |
| getMethod(String name,Class...<?> parameterTypes)         | 获得该类某个共有的方法 |
| getMethods()                                              | 获得该类所有共有的方法 |
| getDeclaredMethod(String name,Class...<?> parameterTypes) | 获得该类某个方法       |
| getDeclaredMethods()                                      | 获得该类所有方法       |

- 类中其他重要的方法

| 方法                                                         | 用途                            |
| ------------------------------------------------------------ | ------------------------------- |
| isAnnotation()                                               | 如果是注解类型则返回true        |
| inAnnotationPressent(Class<? extends Annotation> annotationClass) | 判断这个Class是否拥有指定的注解 |
| isArray()                                                    | 如果是一个数组类则返回true      |
| isEnum()                                                     | 如果是枚举类则返回true          |
| isInstance(Object obj)                                       | 如果是obj是该类的实例则返回true |
| isInterface()                                                | 如果是接口类则返回true          |

# Field

​		Field 代表类的成员变量。成员变量(字段)和成员属性是两个概念。比如：当一个User类中有一个name变量，name这个时候我们就说他有name的这个字段。如果没有setName()和getName()这两个方法，那么这个类就没有name属性。反之，如果这个类拥有setAge()和getAge()这两个方法，不管有没有age字段，我们都认为它有age这个属性。

| 方法                         | 用途                         |
| ---------------------------- | ---------------------------- |
| get(Object obj)              | 获得obj中对应的属性值        |
| set(Object obj,Object value) | 设置obj中对应属性值          |
| SetAccessible                | 暴力反射，忽略访问权限修饰符 |

- Method

  Method 代表类的方法(不包括构造方法)

| 用途                               | 方法                                     |
| ---------------------------------- | ---------------------------------------- |
| invoke(Object obj,Object ... args) | 传递object对象及参数调用该对象对应的方法 |
| getName()                          | 获取方法名                               |
| SetAccessible                      | 暴力反射，忽略访问权限修饰符             |

​    invoke(Object obj,Object ... args)方法用处

​	SpringAOP在切面方法执行的前后进行某些操作，就是使用invoke方法

- Constructor

  Constructor代表类的构造方法

| 方法                              | 用途                       |
| --------------------------------- | -------------------------- |
| new Instance(Object ... initargs) | 根据传递的参数创建类的对象 |

​	Constructor类在实际开发中使用极少，几乎不会使用Constructor。因为Constructor违背了Java的一些思想，比如私有构造不让用户去new 对象。单例模式保证全局只有一个该类的实例。而Constructor则可以破坏这些规则。

#  工具类

- 执行当前指定对象的指定方法

```java
package cn.baerwang.utils;

import java.lang.reflect.Method;
import java.util.Arrays;

/**
 * @author baerwang
 * @date 2020/2/5 22:33
 */
public final class MethodUtils {

    public MethodUtils() {
    }

    /**
     * 执行对象方法
     *
     * @param object     要执行方法的对象
     * @param methodName 要执行的方法名称
     * @param params     方法参数
     * @return
     */
    @SuppressWarnings("all")
    public static Object invokeMethod(Object object, String methodName, Object... params) throws Exception {
        Class<?> aClass = object.getClass();
        System.out.println("类：" + aClass.getClass().getName());
        System.out.println("方法：" + methodName);
        Object invoke = null;
        if (params == null || params.length == 0) {
            Method declaredMethod = aClass.getDeclaredMethod(methodName);
            declaredMethod.setAccessible(true);
            invoke = declaredMethod.invoke(object);
            System.out.println("返回值：" + invoke);
        } else {
            int length = params.length;
            Class<?>[] classes = new Class[length];
            Object[] paramsValue = new Object[length];
            for (int i = 0; i < length; i++) {
                classes[i] = params[i].getClass();
                paramsValue[i] = params[i];
            }
            System.out.println("参数：" + Arrays.asList(paramsValue));
            Method declaredMethod = aClass.getDeclaredMethod(methodName, classes);
            declaredMethod.setAccessible(true);
            invoke = declaredMethod.invoke(object, paramsValue);
            System.out.println("返回值：" + invoke);
        }
        return invoke;
    }
}

```

- 调用工具类

```java
   	@Test
    public void test() throws Exception {
        User user = new User();
        user.setName("zhangsan");
        Object name = MethodUtils.invokeMethod(user, "setName","李四");
        System.out.println(user);
    }
```

# 注解

注解：用文字描述程序，给程序员看的。

定义：用来说明程序的一个标识，给计算机看的，注解也叫元数据，是一种代码级别的说明，它是JDK1.5之后引入的一个特性，是一个特殊的接口，可以使用在字段，类，方法，包，参数等上面。
注意：注解本身并没有任何功能，仅仅起到一个标示性的作用。我们是通过反射去获取到注解，再根据是否有这个注解，注解中的一些属性去判断，执行哪种业务逻辑。

作用分类：

 - 编写文档

   通过代码里的注解去标识去生成API文档(Swagger)。

 - 代码分析

   通过注解去代码进行逻辑上的分析(通过反射去操作业务)。

 - 编译检查

   通过注解让编译器实现基本的编译检查(Override，SuppressWarning)。

JDK中预定义的一些注解：

- Override：

  检查该注解标识的方法是否继承自父类(接口)

- Deprecated：

  标识该内容已过时，后续的版本可能会进行移除。

- SuppressWarnings：

  压制警告，指定警告级别，这个级别下的警告全部不显示。

## 自定义注解

在实际开发中，可能存在一些代码极其复杂或者复用性很低的业务逻辑，比如导出Excel，缓存，将返回值转JSON，事务等等，这个时候就可以使用自定义注解。

## 格式

```java
元注解
public @interface 注解名称{
    属性列表;
}
```

## 属性

本质：接口中的抽象方法

属性只能是以下几种数据类型：

	- 基本数据类型
	- String
	- 枚举
	- 注解
	- 以上类型的数组

如果定义了属性，在使用的时候需要给属性赋值。
如果只有一个属性需要赋值，并且这个属性叫value，那么就可以省略属性名数组赋值时，值用{}包裹，如果数组中只有一个值，那么大括号可以省略。

## 元注解

用于描述注解的注解：
@Target：描述该注解作用的范围
	ElementTyoe：取值：
		Type：作用于类。
		METHOD：作用于方法。
		FIELD：作用于字段。
@Retention：描述注解被保留的阶段
	RetentionPolicy.RUNTIME：当前描述的注解，会保留到class字节码文件中，并被jvm读取到。
@Documented：描述注解是否被抽取到API文档中。
@Inherited：描述注解是否可用被继承。

```java
package cn.baerwang.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @author baerwang
 * @date 2020/2/6 12:47
 */
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String content();
    int num() default 0;
}

```

```java
@MyAnnotation(content = "用户实体类")
public class User  {
}
```

获取类当前注解的名称

```java
    @Test
    public void test() {
        Class<User> userClass = User.class;
        MyAnnotation annotation = userClass.getAnnotation(MyAnnotation.class);
        System.out.println(annotation.content());
    }
```

通过注解把名称赋值到对象中

```java
    @MyAnnotation(content = "张三")
    private String name;

    @MyAnnotation(content = "", num = 1)
    protected Integer age;

    @MyAnnotation(content = "1xx")
    public String idNumber;
```

```java
    @Test
    public void test2() throws Exception {
        User user = new User();
        Class<User> userClass = User.class;
        Field[] declaredFields = userClass.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            MyAnnotation annotation = declaredField.getAnnotation(MyAnnotation.class);
            if (annotation != null) {
                declaredField.setAccessible(true);
                if (annotation.num() != 0) {
                    declaredField.set(user, annotation.num());
                } else {
                    declaredField.set(user, annotation.content());
                }
            }
        }
        System.out.println(user);
    }
```

## 手写缓存注解

```java
package cn.baerwang.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.concurrent.TimeUnit;

/**
 * @author baerwang
 * @date 2020/2/6 13:09
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Cache {

    String key();

    /**
     * 过期时间
     *
     * @return
     */
    int timeOut() default 10;

    /**
     * 默认过期时间单位
     *
     * @return
     */
    TimeUnit timeUnit() default TimeUnit.MINUTES;
}
```

```java
package cn.baerwang.controller;

import cn.baerwang.annotation.Cache;
import cn.baerwang.pojo.User;

/**
 * @author baerwang
 * @date 2020/2/6 13:21
 */
public class UserController {

    private User[] users = {
            new User("张三"),
            new User("李四"),
            new User("王五"),
            new User("赵六")
    };

    @Cache(key = "user")
    public User getUserById(Integer id) {
        System.out.println("方法执行。。。");
        return users[id];
    }
}
```

```java
package cn.baerwang.utils;

import cn.baerwang.annotation.Cache;

import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author baerwang
 * @date 2020/2/6 13:21
 */
public final class CachedUtils {

    private static Map<String, Object> map = new ConcurrentHashMap<>();

    private CachedUtils() {
    }

    /**
     * 执行对象方法
     *
     * @param object     要执行方法的对象
     * @param methodName 要执行的方法名称
     * @param params     方法参数
     * @return
     */
    public static Object invokeMethod(Object object, String methodName, Object... params) throws Exception {
        Object invoke = null;
        try {
            Class<?> aClass = object.getClass();
            if (params == null || params.length == 0) {
                /*无参数，以key为缓存的key*/
                Method declaredMethod = aClass.getDeclaredMethod(methodName);
                declaredMethod.setAccessible(true);
                Cache annotation = declaredMethod.getAnnotation(Cache.class);
                /*方法上有cache注解才能缓存*/
                if (annotation != null) {
                    /*获取key*/
                    String key = annotation.key();
                    Object o = map.get(key);
                    if (o != null) {
                        return o;
                    }
                }
                invoke = declaredMethod.invoke(object);
                if (annotation != null) {
                    String key = annotation.key();
                    /*将返回结果放入缓存中*/
                    map.put(key, invoke);
                }

            } else {
                int length = params.length;
                Class<?>[] classes = new Class[length];
                Object[] paramsValue = new Object[length];
                for (int i = 0; i < length; i++) {
                    classes[i] = params[i].getClass();
                    paramsValue[i] = params[i];
                }
                Method declaredMethod = aClass.getDeclaredMethod(methodName, classes);
                declaredMethod.setAccessible(true);

                Cache annotation = declaredMethod.getAnnotation(Cache.class);
                if (annotation != null) {
                    String key = annotation.key();
                    Object o = paramsValue[0];
                    /*从缓存中查找*/
                    String cacheKey = key + "::" + o;
                    Object o1 = map.get(cacheKey);
                    if (o1 != null) {
                        return o1;
                    }
                }
                invoke = declaredMethod.invoke(object, paramsValue);
                if (annotation != null) {
                    String key = annotation.key();
                    Object o = paramsValue[0];
                    /*从缓存中查找*/
                    String cacheKey = key + "::" + o;
                    /*将返回结果放入缓存中*/
                    map.put(cacheKey, invoke);
                }
            }
        } catch (Exception e) {
            return null;
        }
        return invoke;
    }
}
```

```java
    @Test
    public void test() throws Exception {
        UserController userController = new UserController();
        Object getUserById = CachedUtils.invokeMethod(userController, "getUserById", 1);
        Object getUserById2 = CachedUtils.invokeMethod(userController, "getUserById", 2);
        Object getUserById3 = CachedUtils.invokeMethod(userController, "getUserById", 1);
        System.out.println(getUserById);
        System.out.println();
        System.out.println(getUserById2);
        System.out.println();
        System.out.println(getUserById3);
    }
```

# 总结

反射：就是在程序运行中对Class，对象等进行一些列的操作

注解：注解就是个标识，相当于是给程序看的注释。注解本身不存在功能，需要通过反射去进行某些判断，根据判断结果去执行对应的逻辑，这个过程就是给注解赋予功能的过程。

