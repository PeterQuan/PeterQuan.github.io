---
layout:      post
title:       "Selenium | 自动登录脚本如何绕过二维码、校验码等安全校验"
subtitle:    "我们很难通过登陆过程中的防自动化校验，但是换种思路还是可以解决这个问题的！"
date:        2020-06-06 08:08:08
updated:     2020-06-09 09:09:09
author:      "Quan Qinle"
header-img:  "img/post-bg.webp"
multilingual: false
catalog:      true
tags:
    - Selenium
    - Automated Testing
    - Interface Testing
    - API Testing
---

做Web UI自动化测试的朋友，基本上都会遇到用户登录操作，然而基于安全性考虑，登录过程除了要求输入用户名和密码，往往还会存在为了防止自动化登陆而设置的更加复杂的校验。比如，
![imag](/img/in-post/selenium-login-cookie/01.webp)

既然这些校验的存在就是为了对抗robot、防止自动化登陆，那么，它们自然就成了自动化登陆所面临的难题了。如果知道了这一点，你仍去试着用Selenium或其他技术进行破解，那么，在这样做之前，建议你先掂量掂量自己的技术实力吧。

# 我们自己项目中的安全校验

如果这种校验出现在我们自己的测试项目中，正如我在视频《Web UI自动化测试：Selenium入门》中所说，建议你和研发人员沟通，**在test版本上去除它们，只保留用户和密码即可**。

解决问题的方法有很多，换一种思路，一切豁然开朗。把“登录过程中二维码/短信验证”**从自动化测试用例集中移出，放到手工测试用例集**，技术难题就这么轻松愉快的解决了，码农再也不用为了苦思冥想、挠头掉发担心了。

各位看客，山不过来我就过去~~

# 第三方网站中的安全校验

可是……如果你是在对第三方网站编写网页爬虫，或者编写一些自动化脚本，遇到防自动化登录校验，又该怎么办呢？现在可没有网站程序员帮你临时去掉这个功能模块了。

这种情况，正是本文需要讲解的内容。

# 半自动化方案

我的解决方案是这样的：
1. 人工登录所需网站。
2. 提取页面请求头中的Cookie。
3. 将Cookie添加到WebDriver。

前两步很简单，应该是从业人员的基本功，不过多赘述，但下文也会演示。重点在第3步，我不确定Selenium是否提供了这样的API，幸运的是，最终在官方docs中找到所需的方法。

也许你已经发现了，经过这个方案的处理，我们的自动化变得不纯粹了，它变成了半自动化，因为获取请求Cookie的过程是人工的。Anyway，我的自动化脚本至少又可以运行起来了，加入一点点人工介入也是值得的。

另外一点不方便的是，Cookie中的登录信息都是有有效期的，无论这个网站登录时效使用的是Cookie还是Session，我们不可避免的都需要每隔一段时间重新手工登录网站，获取最新的Cookie，并在我们的代码中进行替换。

# 演示

下面我们以头条公众号后台为例演示整个过程。

## 1、人工登录所需网站

这一步就不演示了，可别泄露了我的密码。
![imag](/img/in-post/selenium-login-cookie/02.webp)

## 2、提取页面请求头中的Cookie

操作步骤：
1. 点击<kbd>F12</kbd>，打开“开发者工具”
2. 刷新页面，捕获请求
3. 在左侧请求列表中切换几次，找到`Request Headers`中`Cookie`有值的请求
4. Cookie字段后面的一长串字符串，就是我们需要的
![imag](/img/in-post/selenium-login-cookie/03.webp)

## 3、将Cookie添加到WebDriver

上一步中得到的Cookie字符串还需要经过处理，因为它是很多Cookie以分号为分隔符拼接之后的内容，我们需要逆向操作：
1. 先以“分号`;`”为分隔符获取Cookie列表，
2. 列表中的元素都含有=号，=左侧是Cookie的name，=右侧是Cookie的value，用name和value构造Cookie，
3. 最后传给WebDriver。

为了验证我们分析的 **“Cookie字符串是以分号分隔、Cookie的name和value在=两侧”**，我们仍回到“开发者工具”，切换到Cookies卡片下，查看Cookie列表完整信息：

![imag](/img/in-post/selenium-login-cookie/04.webp)

# 编码

闲话少叙，我们开始编码吧。

先解析我们得到的原始Cookie字符串，
```java
import org.openqa.selenium.Cookie;

/**
 * 将cookie字符串拆解到list中
 * @param rawCookie 原始Cookie字符串
 * @return Cookie列表
 */
public List<Cookie> parseRawCookie(String rawCookie) {
  List<Cookie> cookies = new ArrayList<>();

  String[] rawCookieParams = rawCookie.split(";");

  for (String cookieParam : rawCookieParams) {
    String[] rawCookieNameAndValue = cookieParam.split("=");
    if (2 != rawCookieNameAndValue.length) {
      log.error("Invalid cookie: missing name and value.");
      continue;
    }
    String cookieName = rawCookieNameAndValue[0].trim();
    String cookieValue = rawCookieNameAndValue[1].trim();
    Cookie cookie = new Cookie(cookieName, cookieValue);
    cookies.add(cookie);
  }
  return cookies;
}
```

将上一步得到的Cookie列表添加给WebDriver，这一步需要放在所有需要权限验证的步骤之前。  
我们可以先添加Cookie，然后刷新页面，这样我们打开的网页就是登陆状态了。
![imag](/img/in-post/selenium-login-cookie/06.webp)

编码演示到这里就够用了，大家不妨试试。

# API补充说明

在这个解决方案中，我们主要用到了两个接口，它们还有一些相关接口，如有需要，请自行查阅。

## 1、添加Cookie

`void addCookie(Cookie cookie) // Add a specific cookie.`
+ 所属接口：
  `org.openqa.selenium.WebDriver.Options`
+ 相关方法：
  `java.util.Set<Cookie> getCookies()`、`deleteCookie(Cookie cookie)` 等等。

## 2、构造Cookie

`Cookie(java.lang.String name, java.lang.String value)`
+ 所属接口：
  `org.openqa.selenium.Cookie`
+ 相关方法：
  `Cookie(java.lang.String name, java.lang.String value, java.lang.String domain, java.lang.String path, java.util.Date expiry, boolean isSecure, boolean isHttpOnly)` 等等

官方docs地址，如下：
[https://seleniumhq.github.io/selenium/docs/api/java/](https://seleniumhq.github.io/selenium/docs/api/java/)
