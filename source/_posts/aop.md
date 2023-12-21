---
title: aop例子
date: 2023/12/8 15:12
tags: 
  - Spring
categories: Spring
top_img: ./img/41.jpg
cover: ./img/34.jpg
---

# aop面向切面编程

### 将程序重复的代码抽取出来，在需要执行的时候，使用动态代理的技术，在不修改源码的基础上，对自己的已有方法进行增强。

1.基于接口的动态代理

2.基于子类动态代理



### 例子分析

#### 1.接口

```java
public interface IExternCall{
	<T> String call(String message,Configure<T> config) throws Exception;
}
```

#### 2.切面增强

```java
@Component
@Aspect
public class DistributeAop{
    @Autowired
    IExternSystemConfig escc;
    
    @Around("execution(* com.xx.xx.ExternCall.call(..))")
    public Object distributeCall(ProceedingJoinPoint proceedingJoinPoint){
        Object resultValue = null;
        Object[] args = proceedingJoinPoint.getArgs();
        try{
            String message = String.valueOf(args[0]);
            Configure<?> configStr = (Configure<?>)args[1];
            resultValue = escc.getExternCall(configStr).call(message,configStr);
        }catch(Throwable e){
            ...
        }
        return resultValue;
    }
}
```

#### 3.Config类（其实这个可以优化掉，要不然过于复杂）

```java
public class Configure<T> {
    public static enum Type{
        A_RPC,B_RPC;
    }
    
    private T config;
    
    private Type type;
    
    public Config(Type type,T config){
        this.type = type;
        this.config = config;
    }
}
```

#### 4.动态代理管理类

```java
@Service
public class ExternSystemCallConfigure implements IExternSystemCallConfigure{
    @Autowired
    RegistrSpringBean registry;
    
    private final ConcurrentHashMap<Type,Class<? extends IExternCall>> config = new ConcurrentHashMap<>();
    
    public ExternSystemCallConfigure(){
        config.put(Type.A_RPC,ARpcCall.class);
        config.put(Type.B_RPC,BRpcCall.class);
    }
    
    @Override
    public IExternCall getExternCall(Configure<?> key){
        return registry.getBean(config.get(key.getType()));
    }
}
```

#### 5.动态管理注册类

```java
@Service
public class RegistrSpringBean implements ApplicationContextAware{
    
    protected ApplicationContext applicationContext;
    
    public void setApplicationContext(ApplicationContext arg0){
        this.applicationContext = arg0;
    }
    public <T> T getBean(Class<T> zlass){
        return this.getBean(zlass.getSimpleName(),zlass);
    }
    protected Object getBean(String id){
        return applicationContext.getBean(id);
    }
    
    protected <T> T getBean(String ckey,Class<T> zlass){
        try{
            Class<?> clazz = Thread.currentThread().getContextClassLoader().loadClass(zlass.getTypeName());
            BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(clazz);
            ConfigurableApplicationContext configurableApplicationContext = (ConfigurableApplicationContext) applicationContext.getBeanFactory();
            beanDefinitionRegistry.setAllowBeanDefinitionOverriding(true);
            System.out.println("ckey "+ckey);
            
            //主动注入bean
            beanDefinitionRegistry.registerBeanDefinition(ckey,beanDefinitionBuilder.getRawBeanDefinition());
            return (T)applicationContext.getBean(ckey);
        }catch(Exception e){
            System.out.println("切面注入失败")
        }
        return null;
    }
    
}
```



#### 6.service层表现

```java
@Component
public class AdapterService{
    @Autowired
    IExternCall extern;
    
    public BoRS xxFlow(BoRq rq){
        String response = extern.call(message,new Configure<>(Type.SEAT_RPC));
    } 
}


```



#### 