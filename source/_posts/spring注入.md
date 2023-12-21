---
title: Spring的Bean注入方式
date: 2023/11/12
tags: Spring
categories: Spring
top_img: ./img/41.jpg
cover: ./img/13.jpg
---

# Spring的bean注入方式

## （一）IOC(控制反转) DI（依赖注入）常见有三种方式：构造器注入，setter注入，接口注入

### 1. 构造方法注入（Constructor Injection）:

```
JAVA
public class UserService {
    private final UserRepository userRepository;

    // 构造方法注入
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // 其他业务方法使用 userRepository
}
```

在这个例子中，`UserService` 类通过构造方法接受一个 `UserRepository` 的实例。当创建 `UserService` 对象时，将传入一个 `UserRepository` 实例，这就是构造方法注入。

### 2. Setter 方法注入（Setter Injection）:

```
JAVA
public class OrderService {
    private PaymentGateway paymentGateway;

    // Setter 方法注入
    public void setPaymentGateway(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    // 其他业务方法使用 paymentGateway
}
```

在这个例子中，`OrderService` 类提供了一个 setter 方法 `setPaymentGateway`，用于设置 `PaymentGateway` 的实例。在使用时，可以调用这个方法来注入依赖。

### 3. 接口注入（Interface Injection）:

```
JAVA
public interface MessagingService {
    void sendMessage(String message);
}

public class EmailService implements MessagingService {
    @Override
    public void sendMessage(String message) {
        // 实现发送邮件的逻辑
    }
}

public class NotificationService {
    private MessagingService messagingService;

    // 接口注入
    public void setMessagingService(MessagingService messagingService) {
        this.messagingService = messagingService;
    }

    public void sendNotification(String notification) {
        // 其他业务逻辑
        messagingService.sendMessage(notification);
    }
}
```

在这个例子中，`NotificationService` 类通过接口注入，它包含一个 `MessagingService` 接口的引用，并提供了一个方法 `setMessagingService` 来设置实际的消息服务实现（例如 `EmailService`）。这就是通过接口来进行依赖注入的方式。

## （二）另外还有几种不常见的注入方式

除了构造方法注入、setter 方法注入和接口注入，还有一些其他的依赖注入方式，具体取决于使用的框架或容器。以下是一些可能的方式：

1. **方法注入：** 除了构造方法和 setter 方法，有些容器支持使用普通方法进行注入。在这种情况下，容器会调用指定的方法，并将依赖传递给该方法。
2. **工厂方法注入：** 有时可以使用工厂方法来创建对象并注入依赖。这种方式通常需要配置工厂方法的相关信息。
3. **字段注入：** 在一些容器中，可以直接通过反射或其他机制来注入依赖到字段上，而不需要 setter 方法或构造方法。
4. **模块系统注入：** 一些框架或平台（如 Java 9+ 的模块系统）提供了模块级别的依赖注入机制，允许在模块之间注入依赖。
5. **属性注入：** 有些容器支持通过注入属性来实现依赖注入，类似于 setter 方法注入，但是是通过设置对象的属性值来完成的。
6. **Service Locator 模式：** 通过一个服务定位器，对象可以查询服务定位器以获取它所需的依赖。这种方式不太推荐，因为它引入了全局状态。

需要注意的是，不同的框架和容器可能对依赖注入的支持方式有所不同。在使用特定框架或容器时，建议查阅相关文档以了解该框架支持的依赖注入方式。

### 下面给出几个例子

我理解了，让我为你提供一些关于方法注入、工厂方法和字段注入的例子：

#### 1. 方法注入（Method Injection）:

```
JAVA
public class ReportGenerator {
    // 方法注入
    public void generateReport(DataSource dataSource) {
        // 生成报告的逻辑，使用传入的 dataSource
    }
}
```

在这个例子中，`ReportGenerator` 类包含了一个方法 `generateReport`，该方法接受一个 `DataSource` 参数。通过调用这个方法，可以将依赖传递给 `ReportGenerator`。

#### 2. 工厂方法注入（Factory Method Injection）:

```
JAVA
public class PaymentProcessor {
    private PaymentGatewayFactory paymentGatewayFactory;

    // 工厂方法注入
    public void processPayment(Order order) {
        PaymentGateway paymentGateway = paymentGatewayFactory.createPaymentGateway();
        // 使用 paymentGateway 处理支付
    }

    // Setter 方法用于注入 PaymentGatewayFactory
    public void setPaymentGatewayFactory(PaymentGatewayFactory paymentGatewayFactory) {
        this.paymentGatewayFactory = paymentGatewayFactory;
    }
}
```

在这个例子中，`PaymentProcessor` 类使用一个工厂方法 `createPaymentGateway` 来创建 `PaymentGateway` 的实例。`PaymentGatewayFactory` 被注入到 `PaymentProcessor` 中，以便在需要时调用工厂方法来获取 `PaymentGateway`。

#### 3. 字段注入（Field Injection）:

```
JAVA
public class LoggerService {
    // 字段注入
    @Inject
    private Logger logger;

    // 其他业务方法使用 logger
}
```

在这个例子中，`LoggerService` 类使用字段注入，通过 `@Inject` 注解将一个 `Logger` 实例注入到 `logger` 字段中。在类的其他方法中，可以直接使用这个注入的 `Logger` 实例。

请注意，尽管这些方法都是可能的依赖注入方式，但并不是所有的框架或容器都支持所有这些方式。具体的使用可能会取决于你所使用的依赖注入框架和其支持的特性。