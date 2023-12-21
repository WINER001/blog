---
title: Spring注解
date: 2023/11/13
tags: Spring
categories: Spring
top_img: ./img/41.jpg
cover: ./img/31.jpg
---

## 4.用于注入数据的注解

### 4.1@Autowired

4.1.1

```
JAVA
@Target({ElementType.CONSTRUCTOR,ElementType.METHOD,ElementType.PARAMETER,ElementType.FIELD,
        ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired{
    //Declares whether the annotated dependency is required.
    boolean required() default true;
}
```

4.1.2说明

作用：

自动按照类型注入。当ioc容器中有且只有一个类型匹配时可以直接注入成功。当有超过一个匹配时，则使用变量名称（写在方法上就是方法名称）作为bean的id，在符合类型的bean中再次匹配，能匹配上就可以注入成功。当匹配不上时，是否报错要看required属性的取值。

属性：

 required：

 是否必须注入成功。默认值是true，表示必须注入成功。当取值为true的时候，注入不成功会报错。

使用场景：

 此注解的使用场景非常之多，在实际开发中应用广泛。通常情况下我们自己写的类中注入依赖bean对象时，都可以采用此注解。

4.1.3示例

```
JAVA
//此处知识举例：使用jdbc座位持久层中的操作数据库对象
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao{
    @Autowired
    private JdbcTemplate jdbcTemplate;
}
```

### 4.2@Qualifier

4.2.1源码

```
JAVA
@Target({ELementType.FIELD,ElementType.METHOD,ELementType.PARAMETER,ElementType.TYPE,ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier{
    String value() default "";
}
```

4.2.2说明

作用：

 当使用自动按类型注入时，遇到有多个类型匹配的时候，就可以使用此注解来明确注入哪个bean对象。注意它通常情况下都必须配置@Autowired注解一起使用

属性：

 value：用于指定bean的唯一标识。

使用场景

 在我们的项目开发中，很多时候都会用到消息队列，我们以ActiveMQ为例。当和spring整合之后，spring框架提供了一个JmsTemplate对象，它既可以用于发送点对点模型消息也可以发送主题模型消息。如果项目中两种消息模型都用上了，那么针对不同的代码，将会注入不同的JmsTemplate，而容器中出现两个之后，就可以使用此注解注入。当然不用也可以，我们只需要把要注入的变量名称改为和要注入的bean的id一致即可。

### 4.3@Resource

4.3.1，源码

```
JAVA
@Target({TYPE,FIELD,METHOD})
@Rentention(RUNTIME)
public @interface Resource{
    String name() default "";
    String lookup() default "";
    Class<?> type() default java.lang.Object.class;
    enum AuthenticationType{
        CONTAINER,
        APPLICATION
    }
    AuthenticationType authenticationType() default AuthenticationType.CONTAINER;
    boolean shareable() default true;
    String mappedName() default "";
    String description() default "";
}
```

4.3.2说明

作用：

 此注解来源于JSR规范，起作用是找到依赖的组件注入到应用来，它利用了JNDI技术查找所需的资源。

 默认情况下，即所有属性都不指定，它默认按照byType的方式装配bean对象。如果指定了name，没有指定type，则采用byName。如果没有指定name，而是指定了type，则按照byType装配bean对象。当byName和byType都指定了，两个都会校验，有任何一个不符合条件就会报错。

属性：

 name：资源的JDNI名称。在spring的注入时，指定bean的唯一标识

 （JNDI ：Java Naming and Directory Interface,Java命名和目录接口是SUN公司提供的一种标准的Java命名系统接口，JNDI提供统一的客户端API，通过不同的访问提供者接口JNDI服务供应接口（SPI）的实现，由管理者将JNDI API映射为特定的命名服务和目录系统，使得Java应用程序可以和这些命名服务和目录服务之间进行交互。目录服务是命名服务的一种自然扩展。两者之间的关键差别是目录服务中对象不但可以有名称还可以有属性（例如，用户有email地址），而命名服务中对象没有属性。）

 JNDI是一个应用程序设计的API，为开发人员提供了查找和访问各种命名和目录服务的通用，统一的接口，类似JDBC都是构建在抽象层上。现在JNDI已经成为J2EE的标准之一，所有的J2EE容器都必须提供一个JNDI的服务。

 JNDI可访问的现有的目录及服务有：

 DNS，XName，Novell目录服务，LDAP，CORBA对象服务，文件系统，WindowsXP/2000/NT/Me/9X的注册表，RMI，DSML v1&v2，NIS。

## 5.注解的target的区分

| 注解                                                         | target                                                 | 依赖           |
| ------------------------------------------------------------ | ------------------------------------------------------ | -------------- |
| @Retention（保留多久，SOURCE，CLASS，RUNTIME）               |                                                        | 元             |
| @Target（注解作用目标） TYPE，允许在类，接口，枚举上 FIELD，允许在字段上 METHOD，允许在方法上 PARAMETER，允许在方法参数上 CONSTRUCTOR，允许在构造器上 LOCAL_VARIABLE，允许在本地局部变量上 ANNOTATION_TYPE，允许在注解上 PACKAGE，允许在包上 |                                                        | 元             |
| @Document（生成javadoc时会不会放进去）                       |                                                        | 元             |
| @Repeatable（是否可重复）                                    |                                                        | 元             |
| @Configuration（当前类是配置类）                             | TYPE                                                   | @Component     |
| @ComponentScan（指定要扫描的包）                             | TYPE                                                   | 元             |
| @Bean（方法上注入）                                          | ANNOTATION_TYPE，METHOD                                | 元             |
| @Import（和@Configuration配合，指定注入）                    | TYPE                                                   | 元             |
| @PropertySource（将properties配置文件的值存储到spring的Environment中） |                                                        | 元             |
| @DependsOn（注入时机和注入条件）                             | TYPE，METHOD                                           | 元             |
| @Lazy（先不加载，调用时才加载）                              | TYPE，METHOD，CONSTRUCTOR，PARAMETER，FIELD            | 元             |
| @Conditional（根据条件选择注入的bean对象）                   | TYPE，METHOD                                           | 元             |
| @Configuration（注入类）                                     | TYPE                                                   | 元             |
| @Controller                                                  | TYPE                                                   | @Component     |
| @Service                                                     | TYPE                                                   | @Component     |
| @Repository                                                  | TYPE                                                   | @Component     |
| @Autowired（默认byType自动装配，由@Qualifier实现byName装配） | CONSTRUCTOR，FIELD，METHOD，PARAMETER，ANNOTATION_TYPE | 元             |
| @Qualifier（必须配置@Autowired一起用）                       | TYPE，FIELD，METHOD，PARAMETER，ANNOTATION_TYPE        | @Inherited，元 |
| @Inherited(被这个标注的注解是可以被子类继承的)               | ANNOTATION_TYPE                                        | 元             |
| @Resource（来源于JSR-250规范，默认byType，指定name后，byName） | TYPE，FIELD，METHOD                                    | 元             |
| @Value（支持spring的EL表达式，${}获取properties，xml和yml文件） | FIELD，METHOD，PARAMETER，ANNOTATION_TYPE              | 元             |
| @Inject（JSR-330规范，可配合@Qualifier或@Primary使用，默认byType，指定@Named注解之后，byName） | METHOD,CONSTRUCTOR,FIELD                               | 元             |
| @Primary（指定bean的注入优先级，优先注入）                   | TYPE，METHOD                                           | 元             |
| @Scope（指定bean对象的作用范围）                             | TYPE，METHOD                                           | 元             |
| @PostConstruct（指定bean对象的初始化方法）                   | METHOD                                                 | 元             |
| @PreDestroy（指定bean对象的销毁方法）                        | METHOD                                                 | 元             |

### 6.@Inherited

```
JAVA
package com.test.inherited;
import java.lang.annotation.Annotation;

public class Test{
    public static void main(String[] args){
        //打印父类注解信息
        Annotation[] fatherAnnotations = Father.class.getAnnotations();
        System.out.println("--------父类 Father 信息---------");
        System.out.println("父类注解个数： "+fatherAnnotations.length);
        for(Annotation fa : fatherAnnotations){
            System.out.println(fa.annotationType().getSimpleName());
        }
        
        Annotation[] childAnnotations = Child.class.getAnnotations();
        System.out.println("----------子类 Child 信息-------------");
        System.out.println("子类注解个数： "+childAnnotations.length);
        for(Annotation ca : childAnnotations){
            System.out.println(ca.annotationType().getSimpleName());
        }
    }
}
```

### 7.@Scope

```
JAVA
@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Document
public @interface Scope{
    //Alias for{@link #scopeName}
    @AliasFor("scopeName")
    String value() default "";
    
    /**
    *Specifies the name of the scope to use for the annotated component/bean.
    *<p>Defaults to an empty strong({@code ""})which implies
    *{@link ConfigurableBeanFactory$SCOPE_SINGLETON SCOPE_SINGLETON}.
    *@since 4.2
    *@see ConfigurableBeanFactory#SCOPE_PROTOTYPE
    *@see ConfigurableBeanFactory#SCOPE_SINGLETON
    *@see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
    *@see rog.springframework.web.context.WebApplicationContext$SCOPE_SESSION
    *@see #value
    */
    @AliasFor("value")
    String scopeName() default "";
    /**
    *Specifies whether a component should be configured as a scoped proxy
    *and if so,whether the proxy should be interface-based or subclass-based.
    *<p>Defaults to{@link ScopedProxyMode#DEFAULT},which typically indicates
    *that no scoped proxy should be created unless a different default
    *has been configured at the component-scan instruction level.
    *<p>Analogous to {@code<aop:scoped-proxy/> }support in Spring XML.
    *@see ScopedProxyMode
    */
    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
    
}
```

5.1.2说明

作用：用于指定bean对象的作用范围。

属性：value：指定作用范围的取值。在注解中默认值是“”。

 但是在spring初始化容器时，会借助ConfigurableBeanFactory接口中的类成员：

 String SCOPE_SINGLETON = “singleton”;

 scopeName:它和value的作用是一样的。

 proxyMode：它是指定bean对象的代理方式的。指定的是ScopedProxyMode枚举的值

 DEFAULT：默认值（就是NO）

 NO：不适用代理

 INTERFACES:使用JDK官方的基于接口的代理。

 TARGET_CLASS：使用CGLIB基于目标类的子类创建代理对象。

使用场景：在实际开发中。我们的bean对象默认都是单例的。通常情况下，被spring管理的bean都是用单例模式来创建。但是也有例外，例如Struts2框架中的Action，由于有模型驱动和OGNL表达式的原因，就必须配置成多例的。

5.1.3示例

#### 5.2 @PostConstruct

5.2.1源码

```
JAVA
@Documented
@Retention(RUNTIME)
@Target(METHOD)
public @interface PostConstruct{}
```

5.2.2说明

作用：用于指定bean 对象的初始化方法。

属性：无

使用场景：在bean对象创建完成后，需要对bean中的成员进行一些初始化的操作是，就可以使用此注解配置一个初始化方法，完成一些初始化的操作。

### 5.3@PreDestroy

5.3.1源码

```
JAVA
@Documented
@Retention(RUNTIME)
@Target(METHOD)
public @interface PreDestroy{}
```

5.3.2说明

作用：用于指定bean对象的销毁方法。

属性：无

使用场景：在beandu9ixiang销毁之前，可以进行一些清理操作。

## 二。Spring高级-IOC的深入剖析

### 1.Spring中的BeanFacotry

1.1BeanFactory类视图

2.2.2说明

现实中的容器都是用来装物品的，Spring的容器也不例外，这里的物品就是bean。我通常对于bean的印象是一个个躺在配置文件中的标签，或者是被注解的类，但是这些都是bean的静态标识，是还没有放入容器的物料，最终（加载完配置，且在getBean之前）加载到容器中的是一个个BeanDefinition实例。BeanDefinition的继承关系如下图，RootBeanDefinition，ChildBeanDefinition，以及GenericBeanDefinition是是哪个主要的实现。有时候我们需要在配置时，通过parent属性指定bean的父子关系，这个时候父bean则用RootBeanDefinition表示，而子bean则用ChildBeanDefinition表示。GenericBeanDefinition自2.5版本引入，是对于一般的bean定义的一站式服务中心。

2.2Bean的定义信息详解

2.2.1源码分析

```
JAVA
/**在上一小节我们加少了RootBeanDefinition，ChildBeanDefinition，GenericBeanDefinition三个类
*他们都是由AbstractBeanDefinition派生而来，该抽象类中包含了bean的所有配置项和一些支持程序运行的属性。以**下是类中属性的说明。
*/
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor implements BeanDefinition,Cloneable{
    //常量定义略
    //bean对应的类实例
    private volatile Object beanClass;
    //bean对应的作用域，对应scope属性
    private String scope = SCOPE_DEFAULT;
    //是否是抽象类，对应abstract属性
    private boolean abstractFlag = false;
    //是否延迟加载，对应lazy-init属性
    private int autowireMode = AUTOWIRE_NO;
    //依赖检查，对应dependency-check属性
    private int dependencyCheck = DEPENDENCY_CHECK_NONE;
    //对应depends-on,表示一个bean实例化前置依赖另一个bean
    private String[] dependsOn;
    //对应autowire-candidate属性，设置为false时表示取消当前bean座位自动装配候选者的资格
    private boolean autowireCandidate = true;
    //对应primary属性，当自动装配存在多个候选者时，将其座位首选
    private boolean primary = false;
    //对应qualifier属性
    private final Map<String,AutowireCandidateQualifier> qualifiers = new LinkedHashMap<String,AutowireCandidateQualifier>(0);
    //非配置项：表示允许访问非公开的构造器和方法，由程序设置
    private booean nonPublicAccessAllowed = true;
    
    private boolean lenientConstructorResolution = true;
    
    //对应factory-bean属性
    private String factoryBeanName;
    //对应factory-method属性
    private String factoryMethodName;
    //记录构造函数注入属性，对应<construct-arg/>标签
    private ConstructorArgumentValues constructorArgumentValues;
    //记录<property/>属性集合
    private MutablePropertyValues propertyValues;
    //记录<lookup-method/>和<replaced-method/>标签配置
    private MethodOverrides methodOverrides = new MethodOverrides();
    //对应init-method属性
    private String initMethodName;
    //对应destroy-method属性
    private String destroyMethodName;
    //非配置项：是否执行init-method，由程序设置
    private boolean enforceInitMethod = true;
    //非配置项：是否执行destroy-method，由程序设置
    private boolean enforceDestroyMethod = true;
    //非配置项：标识是否是用户定义，而不是程序定义的，创建AOP时为true，由程序设置
    private boolean synthetic = false;
    /**非配置项：定义bean的应用场景，由程序设置，角色如下：
    *ROLE_APPLICATION：用户
    *ROLE_INFRASTRUCTURE:完全内部使用
    *ROLE_SUPPORT：某些复杂配置的一部分
    */
    private int role  = BeanDefinition.ROLE_APPLICATION;
    //bean的描述信息，对应description标签
    private String description;
    //bean定义的资源
    private Resource resource;
}
```

2.2.2总结

BeanDefinition是容器对于bean配置的内部表示，Spring将各个bean的BeanDefinition实例注册记录在BeanDefinitionRegistry中，该接口定义了对BeanDefinition的各种增删改查，类似于内存数据库，其实现类SimpleBeanDefinitionRegistry主要以Map座位存储标的。