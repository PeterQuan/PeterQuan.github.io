---
layout:       post
title:        "Selenium元素定位By.cssSelector()"
subtitle:     "CSS Selector最常用方法汇总"
date:         2020-06-30
author:       "权芹乐"
catalog:      true
tags:
    - CSS Selector
    - Automated Testing
    - Selenium
    - 自动化测试
---

[toc]

[toc]

CSS Selector 使用CSS定位元素

# `CSS Selector`是什么？

> [MDN web docs]
>
> CSS selectors define the elements to which a set of CSS rules apply.

> [w3schools]
>
> In CSS, selectors are patterns used to select the element(s) you want to style.


# CSS Selector语法

## 4个基础选择器
个人认为，`CSS Selector`最重要的选择器只有下面4个，分别是 tagname标签名称，标签id属性，class属性，以及其他属性：

* tag
* #id
* .class
* [attribute=value]

这4个的各种组合，可以出现丰富、强大的效果。

举几个例子

```css
tag#id
tag.class
tag.class1.class2   // 多个class
tag[attributeName='attributeValue']
tag[attributeName='attributeValue'][attributeName2='attributeValue2'] // 多个属性
tag#id.class[attributeName='attributeValue'] // 4个选择器
```

## 更多选择器

下面再列几个我常用的选择器。

```css
/*-- 属性的模糊匹配 --*/
// ^ - Starts with
[attribute^='attributeValue']
// $ - Ends with
[attribute$='attributeValue']
// * - Contains
[attribute*='attributeValue']

/*-- 父子节点 --*/
// 直接子节点，多个（parentLocator>childLocator）。XPath: //div/a
div#buttonDiv>button
// 相对子节点，多个（locator1 locator2）。XPath: //div//a
div#buttonDiv button
// 同一父节点下的第n个子元素，从1开始（:nth-child(n)）。下例，若li不是其父的第2位节点，则失败；数位子时，所有类型一起数
#testingTypes li:nth-child(2)

/*-- 兄弟节点 --*/
// 紧邻的下一个节点，1个（locator1+locator2）
li#automation + li
// 其后的兄弟节点，可以被其他兄弟隔断（element1~element2）
p~ul
```

CSS Selector还有其他复杂的语法，但上面我们讲到的已经是精华，实用性最强。

更多的见资料 [^1]

# `Selenium`中的`CSS Selector`

```
By by = By.cssSelector(String selector);
//Find elements via the driver's underlying W3C Selector engine.
```

示例：
```java
By.cssSelector("button[id='stop']")  
By.cssSelector("button#stop")  
By.cssSelector("#stop")  

By.cssSelector("button.x-right-button")   
By.cssSelector(".x-right-button") 
// <button class="x-btn-text module_picker_icon" /> 
By.cssSelector("button.x-btn-text.module_picker_icon")  // 多个class

//^=start with $=ends with *=contains
By.cssSelector("input[id^='Em']")
By.cssSelector("input[id$='wd']")
By.cssSelector("input[id*='ni']") // 使用a[href*='.jpg']成功！
By.cssSelector("a:contains('Sign in')") 
//contains试用失败！原因见下文invalid selector: An invalid or illegal selector was specified


By.cssSelector("a[title=\"Go to Facebook home\"]")
By.cssSelector("input[id='signIn'][value='Sign in']") // 多个属性
By.cssSelector("div#west-panel>div:nth-child(1)>div:nth-child(1)>div:nth-child(2)>div") 
By.cssSelector("td.col5>div>input:nth-child(4)") // nth-child(4) 表示查找第四个Pseudo-selements元素

// XPATH: //input[@id='username']/following-sibling::input[1]
By.cssSelector(".username + input") 

```


# 不能通过部分文本`contains`定位元素

有一点需要特别注意，不能使用 `a:contains('Sign in')`。

原因：
1. **Inner texts**   
are the actual string patterns that the HTML label shows on the page.  
2. (Not supported by WebDriver)  
CSS: The CSS2 contains() function is not in CSS3; however, Selenium supports the superset of CSS1, 2, and 3.  
contains() is not part of the current CSS3 specification so it will not work on all browsers, only ones that implemented it before it was pulled.

遇到此种场景，可以使用`By by = By.ByXPath(String xpathExpression)`。  
xpath见资料 [^2]

# 资料：

[^1]: CSS Selector详解：https://www.w3schools.com/cssref/css_selectors.asp

[^2]: CSS Selector与xpath语法比较：https://saucelabs.com/resources/articles/selenium-tips-css-selectors
