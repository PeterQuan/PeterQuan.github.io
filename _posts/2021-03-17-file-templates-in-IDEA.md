---
layout:       post
title:        "Java | 使用IDEA的文件模板功能简化Spring Boot的类创建"
subtitle:     "在Spring Boot项目增加一个entity时，一般需要顺带创建多个类，使用IntelliJ的`File and Code Templates`一键创建所有类"
date:         2021-03-17
updated:      2021-03-17
author:       "权芹乐"
header-img:   "img/home-bg.webp"
catalog:      true
tags:
    - Java
    - IDEA
    - Spring Boot
---

[toc]

# 环境
+ IntelliJ IDEA Community 2020.3.3

# 介绍

在Spring Boot项目中，每当新增一个entity/module对象时，如`UserPO.java`，大多时候接下来还要创建对应的repository、service、service implement、controller等，并且最开始类的内容也是相似，一套模板类做下来，慢慢的重复劳动。

于是，想着是否存在根据类模板“一键”创建多个类文件的功能，在`Settings`中一番翻找，在`File and Code Templates`中找到了解决办法。本文就是介绍通过文件模板批量创建文件的步骤。

# 期望+最终效果演示

> 结果即需求

先演示一下最终实现的效果，也就是最初的需求
1. 在包根目录下，右键——>`New`——>选择新设置的模板，
   ![right click](/img/in-post/file-templates-in-IDEA/right-click-new-entity.webp)
2. 输入entity名，如`User`，首字母大写
   ![input entity](/img/in-post/file-templates-in-IDEA/input-entity.webp)
3. 生成的如下文件：
   + entity/po : `User.java`
   + dao : `UserRepository.java`
   + service : `UserService.java`
   + service/impl : `UserServiceImpl.java`
   + controller : `UserController.java`

对应的项目结构如下，下文的配置也是以这个结构为前提的。
```
src/main/java
└── com.github.quanqinle
        ├── Application.java
        ├── entity
        │   ├── po
        │   │   └── User.java
        │   └── vo
        ├── dao
        │   └── UserRepository.java
        ├── service
        │   ├── impl
        │   │   └── UserServiceImpl.java
        │   └── UserService.java
        └── controller
            └── UserController.java
```

# 设置

先放一张最终的配置，就像下图中的(1)所示：

![final settings](/img/in-post/file-templates-in-IDEA/final-settings.webp)
<center style="color:#C0C0C0;">图1</center> 

## 配置po模板
打开`Settings`窗口，找到`Editor`——>`File and Code templates`，在`Files`分类下，点击<kbd>Create Template</kbd>，即图1的按钮(2)

+ Name：右键创建时看到的名字，例`Create whole classes in package root`
+ Extension：默认的java
+ File Name：文件路径和文件名（不用加.java后缀），`./entity/po/${Subject}`
+ 输入模板内容，如下：

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.entity.po;#end

import lombok.Data;
import javax.persistence.*;
import java.time.LocalDateTime;

#parse("File Header.java")
@Data
@Entity
@Table(name = "${SubjectOfLowerFirst}")
public class ${Subject} {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    /**
     * name
     */
    private String name;

    @Column(name = "create_time")
    private LocalDateTime createTime;
    @Column(name = "update_time")
    private LocalDateTime updateTime;
}
```

## 配置dao模板
选中第一步创建的<kbd>Create Template</kbd>前提下，点击<kbd>Create Child Template File</kbd>，即图1的按钮(3)

+ File Name：文件路径和文件名（不用加.java后缀），`./dao/${Subject}Repository`
+ Extension：默认的java
+ 输入模板内容，如下：

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.dao;#end

import ${PACKAGE_NAME}.entity.po.${Subject};
import org.springframework.stereotype.Repository;
import org.springframework.data.jpa.repository.JpaRepository;

#parse("File Header.java")
@Repository
public interface ${Subject}Repository extends JpaRepository<${Subject}, Long> {
}
```

## 配置service模板
选中第一步创建的<kbd>Create Template</kbd>前提下，点击<kbd>Create Child Template File</kbd>，即图1的按钮(3)

+ File Name：文件路径和文件名（不用加.java后缀），`./service/${Subject}Service`
+ Extension：默认的java
+ 输入模板内容，如下：

```java
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.service;#end

import ${PACKAGE_NAME}.entity.po.${Subject};
import java.util.Optional;

#parse("File Header.java")
public interface ${Subject}Service {

    /**
     * insert
     * @param ${SubjectOfLowerFirst} a ${Subject} object
     * @return
     */
    ${Subject} insert(${Subject} ${SubjectOfLowerFirst});

    /**
     * update
     * @param ${SubjectOfLowerFirst} a ${Subject} object
     * @return
     */
    ${Subject} update(${Subject} ${SubjectOfLowerFirst});
    
    /**
     * query by id
     * @param id ${Subject} id
     * @return
     */
    Optional<${Subject}> queryById(Long id);
    
    /**
     * delete by id
     * @param id ${Subject} id
     * @return
     */
    boolean deleteById(Long id);
    
}
```

## 配置service implement模板
选中第一步创建的<kbd>Create Template</kbd>前提下，点击<kbd>Create Child Template File</kbd>，即图1的按钮(3)

+ File Name：文件路径和文件名（不用加.java后缀），`./service/impl/${Subject}ServiceImpl`
+ Extension：默认的java
+ 输入模板内容，如下：

```
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.service.impl;#end

import ${PACKAGE_NAME}.dao.${Subject}Repository;
import ${PACKAGE_NAME}.entity.po.${Subject};
import ${PACKAGE_NAME}.service.${Subject}Service;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.Optional;

#parse("File Header.java")
@Service
@Transactional(rollbackFor = Exception.class)
public class ${Subject}ServiceImpl implements ${Subject}Service {

    private Logger log = LoggerFactory.getLogger(${Subject}Service.class);

    @Autowired
    private ${Subject}Repository repository;

    public ${Subject}ServiceImpl() {
    }

    @Override
    public ${Subject} insert(${Subject} ${SubjectOfLowerFirst}) {
        return repository.save(${SubjectOfLowerFirst});
    }

    @Override
    public ${Subject} update(${Subject} ${SubjectOfLowerFirst}) {
        return repository.save(${SubjectOfLowerFirst});
    }

    @Override
    public boolean deleteById(Long id) {
        Boolean result = true;
        try {
            repository.deleteById(id);
        } catch (Exception e) {
            result = false;
        }
        return result;
    }

    @Override
    public Optional<${Subject}> queryById(Long id) {
        return repository.findById(id);
    }

}
```

## 配置controller模板
选中第一步创建的<kbd>Create Template</kbd>前提下，点击<kbd>Create Child Template File</kbd>，即图1的按钮(3)

+ File Name：文件路径和文件名（不用加.java后缀），`./controller/${Subject}Controller`
+ Extension：默认的java
+ 输入模板内容，如下：

```
#set($SubjectOfLowerFirst = ${Subject.substring(0,1).toLowerCase()} + $Subject.substring(1))
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}.controller;#end

import ${PACKAGE_NAME}.entity.Result;
import ${PACKAGE_NAME}.entity.po.${Subject};
import ${PACKAGE_NAME}.service.${Subject}Service;
import org.springframework.web.bind.annotation.*;
import javax.annotation.Resource;

#parse("File Header.java")
@RestController
@RequestMapping("api/${SubjectOfLowerFirst}")
public class ${Subject}Controller {

    @Resource
    private ${Subject}Service ${SubjectOfLowerFirst}Service;

    @PostMapping
    public Result<${Subject}> create(@RequestBody ${Subject} record) {
        ${Subject} ${SubjectOfLowerFirst} = ${SubjectOfLowerFirst}Service.insert(record);
        return Result.success(${SubjectOfLowerFirst});
    }

    @PutMapping
    public Result<${Subject}> update(@RequestBody ${Subject} record) {
        ${Subject} ${SubjectOfLowerFirst} = ${SubjectOfLowerFirst}Service.update(record);
        return Result.success(${SubjectOfLowerFirst});
    }

    @DeleteMapping("{id}")
    public Result<Void> deleteById(@PathVariable Long id) {
        boolean success = ${SubjectOfLowerFirst}Service.deleteById(id);
        return success ? Result.success() : Result.fail();
    }

    @GetMapping("{id}")
    public Result<${Subject}> queryById(@PathVariable Long id) {
        var optional = ${SubjectOfLowerFirst}Service.queryById(id);
        if (optional.isPresent()) {
            return Result.success(optional.get());
        } else {
            return Result.success();
        }
    }

}
```

保存配置。
