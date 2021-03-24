---
layout:       post
title:        "Junit | 在Spring Boot2低版本中使用Junit 5"
subtitle:     "Spring Boot 2.2.0开始支持Junit 5，之前的版本需要手动引入依赖"
date:         2020-06-12
updated:      2020-06-12
author:       "Quan Qinle"
header-img:   "img/post-bg-beach1.webp"
multilingual: false
catalog:      true
tags:
    - Junit
    - Spring Boot
    - Unit Testing
---

先说结论：如果你用的`Spring Boot`版本是`2.2.0`或更新，那么，`spring-boot-starter-test`已自带`Junit 5`，本文对你无用，你可以直接在你的工程中编码了。

下文是`Spring Boot 2`版本低于`2.2.0.RELEASE`时的配置方法。

我的环境：
+ Java 8+
+ Maven 3
+ Spring Boot 2.0.7.RELEASE
+ 想使用Junit 5，且没有Junit 4的历史用例

配置步骤不复杂，简单来说，从`spring-boot-starter-test`中去掉`Junit 4`，再引入`Junit 5`。  
pom.xml需要修改的部分如下：
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
  <exclusions>
    <exclusion>
      <!-- Junit updates to version 5 since springboot 2.2-->
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.6.2</version>
  <scope>test</scope>
</dependency>


<plugin>
  <!-- unit test: Need at least 2.22.0 to support JUnit 5 -->
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.22.2</version>
</plugin>
```

Tip：`junit-jupiter`已包含`junit-jupiter-engine`、`junit-jupiter-api`、`junit-jupiter-params`，所以，只引入它就够了。
