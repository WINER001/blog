---
title: Spring是如何通过反射注入bean的
date: 2023/11/12 23:59
tags: Spring
categories: Spring
top_img: ./img/41.jpg
cover: ./img/34.jpg
---

# Spring是如何通过反射注入bean的

Spring 框架的底层使用了反射来实现依赖注入和对象管理。下面是一个简单的示例，演示了 Spring 底层是如何通过反射进行依赖注入的。

首先，我们定义一个简单的类 `UserService` 和 `UserRepository`：

```
JAVA
public class UserRepository {
    public void saveUser(User user) {
        System.out.println("Saving user: " + user.getUsername());
    }
}

public class UserService {
    private UserRepository userRepository;

    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public void createUser(String username, String password) {
        User user = new User(username, password);
        userRepository.saveUser(user);
    }
}
```

然后，我们定义一个简单的容器，该容器在初始化时使用反射创建对象并进行依赖注入：

```
JAVA
import java.lang.reflect.Field;

public class SimpleContainer {
    public static void main(String[] args) {
        // 创建 UserRepository 对象
        UserRepository userRepository = createObject(UserRepository.class);

        // 创建 UserService 对象，并注入 UserRepository 依赖
        UserService userService = createObject(UserService.class);
        userService.setUserRepository(userRepository);

        // 调用方法，触发依赖注入
        userService.createUser("john_doe", "password123");
    }

    private static <T> T createObject(Class<T> clazz) {
        try {
            // 使用反射创建对象
            T obj = clazz.getDeclaredConstructor().newInstance();

            // 模拟 Spring 的依赖注入，通过反射设置字段的值
            for (Field field : clazz.getDeclaredFields()) {
                if (field.isAnnotationPresent(Autowired.class)) {
                    // 获取字段的类型
                    Class<?> fieldType = field.getType();

                    // 递归创建依赖对象
                    Object fieldObject = createObject(fieldType);

                    // 设置字段的值
                    field.setAccessible(true);
                    field.set(obj, fieldObject);
                }
            }

            return obj;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

在这个例子中，`SimpleContainer` 类模拟了一个简化版的容器，通过反射创建对象并进行依赖注入。它使用 `createObject` 方法来递归创建对象，并通过反射设置带有 `@Autowired` 注解的字段的值。

请注意，这只是一个简单的演示，实际的 Spring 框架比这个复杂得多。Spring 使用了更复杂的机制和策略来处理不同类型的依赖注入，支持构造方法注入、setter 方法注入等多种方式。