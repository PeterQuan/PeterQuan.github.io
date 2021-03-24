---
layout:       post
title:        "Selenium | 借助JavaScript实现一些不易处理的操作"
subtitle:     "当无法按预期完成一些元素操作时，可以考虑执行JavaScript脚本"
date:         2020-09-08
updated:      2020-09-08
author:       "权芹乐"
catalog:      true
tags:
    - Selenium
    - JavaScript
---

[toc]

# JavascriptExecutor
Javascript脚本执行器。

`arguments[i]`是js脚本传参的占位符，i从0开始。

# 页面滚动

jse方式滚动页面的几种方式：
```java
JavascriptExecutor jse = (JavascriptExecutor)driver;

// 滚动到某元素（tips：适用于元素不可见或被遮挡）
jse.executeScript("arguments[0].scrollIntoView();", element);

// 向下滚动
jse.executeScript("window.scrollBy(0, 500)", "");
jse.executeScript("scroll(0, 500);");

// 滚动到底部
jse.executeScript("window.scrollTo(0, document.body.scrollHeight)");

// 动态元素的高度
double ImageHeight = eachtile.getSize().getHeight();
double f = 1.04*ImageHeight;
((JavascriptExecutor)driver).executeScript("window.scrollBy(0,arguments[0]);", -f);
```


# 元素内取值
```HTLM
<div class="item-price">700 Yuan</div>

<div class="item-price">
  <span>500 Yuan</span>
  400 Yuan
</div>
```

div可能有span也可能没有，但要不取出span中的文本，即，期望得到700 Yuan和400 Yuan
```java
WebElement element = driver.findElement(By.xpath("//div[contains(@class, 'item-price')]"));

JavascriptExecutor jse = (JavascriptExecutor)driver;
jse.executeScript("return arguments[0].lastChild.textContent;", element);
```


# list超长，点击其中“不可见”的选项
```java
public void selectListByJS() {
    JavascriptExecutor js = (JavascriptExecutor) driver;
    String css;
    css = "document.querySelectorAll('.select-menu')[0].querySelector('.select-option:nth-child(24)').click();";
    js.executeScript(css);
}


// 修改属性，使可见。
// tip：还没验证过
String strJs = "document.getElementsByClassName('arguments[0]').style.height='auto'; document.getElementsByClassName('arguments[0]').style.visibility='visible';";
```

# 点击元素
```java
/**
 * 通过JavaScript实现点击元素。
 * tips：适用于click()失效时
 *
 * @param element
 */
public void clickByJS(WebElement element) {
	JavascriptExecutor jse = (JavascriptExecutor)driver;
	jse.executeScript("arguments[0].click()", element);
}
```

# 资料
https://www.guru99.com/execute-javascript-selenium-webdriver.html
