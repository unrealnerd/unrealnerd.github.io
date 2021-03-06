---
layout: post
title: Getting new instance everytime in Java Spring Boot 
description: Getting new instance everytime in Java Sping Boot
author: bitsmonkey
tags: java, spring, spring-boot
---

![java-springboot-newinstance](/img/java-newinstance-springboot.jpg){:.coverimage}

This is one of the ways to get a new instance of a type everytime we request for it from a singleton scoped instance.

[`@Lookup`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Lookup.html) Informs spring container to getBean from method decorated with this annotation.

Creating a method in configuration class so that it can be used anywhere configuration is injected.

```java
//ApplicationConfiguration.java
@Lookup
public UserDTO getUserDTOProtoTypeBean() {
    return null;
}
```

Make sure the `UserDTO` class is decorated with `@Scope("prototype")`. Now in your consuming class call this method when you need a new instance.


```java
@Component
public class UserMapper {

    @Autowired
    ApplicationConfiguration appConfig;

    public UserDTO ToDto(User userEntity) {
        var UserDTO = appConfig.getUserDTOProtoTypeBean();
        ...
    }
}
```

Code On!