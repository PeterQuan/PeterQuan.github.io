---
layout:       post
title:        "JUnit | 参数化测试用例"
subtitle:     "使用`@ParameterizedTest`接收不同参数，循环多次执行测试方法"
date:         2020-08-14
updated:      2020-08-14
author:       "权芹乐"
catalog:      true
tags:
    - junit
---

[toc]

在JUnit5中，可以将`@Test`替换成`@ParameterizedTest`，这样就可以使用不同的参数、多次循环运行测试方法。

# `@ValueSource`

```Java
@DisplayName("this is demo 1")
@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
public void test_Demo1(int arg) {
    int expected = 2;
    assertEquals(expected, arg, "You are not such 2");
}

@DisplayName("this is demo 2")
@ParameterizedTest(name = "{index} ==> the testcase is running")
@ValueSource(strings = {"1", "2", "3,4"})
public void test_Demo2(String arg) {
    assertNull(arg, "不应空虚");

    String expected = "2";
    assertEquals(expected, arg, "You are not such 2");
}
```

在`@ValueSource`中，可以使用strings、ints等等。但是，我个人更**倾向于使用strings**，因为String可以隐式转换为很多格式，这样传参更灵活一些。

```Java
@ParameterizedTest
@ValueSource(strings = {"true", "false"})
public void test_Demo3(boolean arg) {
    assertTrue(arg);
}

@ParameterizedTest
@ValueSource(strings = {"0", "1.1", "2.2"})
public void test_Demo4(BigDecimal arg) {
    assertEquals(BigDecimal.ZERO, arg);
}
```

[String可以自动转换成哪些格式？](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit)


# `@CsvSource`

```Java
@ParameterizedTest
@CsvSource({
    "  85,  体育,         true",
    "99.5,  '语,数,外',   false",
    })
public void test_Demo5(BigDecimal score, String subject, boolean isWin) {

}
```

## ArgumentsAccessor 参数聚合器

通过`ArgumentsAccessor`一次接收多个参数。

```java
@ParameterizedTest
@CsvSource({
    "  85,  体育,         true",
    "99.5,  '语,数,外',   false",
    })
public void test_Demo6(ArgumentsAccessor args) {
    args.get(0 , BigDecimal.class);
    args.get(1, String.class); // OR args.getString(1);
    args.get(2, Boolean.class);
}
```

[ArgumentsAccessor doc api](https://junit.org/junit5/docs/current/api/org.junit.jupiter.params/org/junit/jupiter/params/aggregator/ArgumentsAccessor.html)


# `@CsvFileSource`

和`@CsvSource`的区别，从csv文件读取测试数据，传参等用法相同。

```java
@ParameterizedTest
@CsvFileSource(resources = "/login-data.csv", numLinesToSkip = 1)
void test_Demo7(ArgumentsAccessor args) {
}
```
