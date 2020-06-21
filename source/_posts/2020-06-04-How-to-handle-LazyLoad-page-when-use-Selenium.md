---
layout:      post
title:       "Selenium | 编写爬虫抓取网页图片遭遇懒加载"
subtitle:    "记录一次抓取网站漫画图片的历程"
date:        2020-06-04 08:00:09
updated:     2020-06-04 08:00:09
author:      "Quan Qinle"
header-img:  "img/in-post/selenium-lazyload/selenium-lazyload-01.webp"
multilingual: false
catalog:      true
tags:
    - Selenium
    - 自动化测试
    - 网页抓取
    - 爬虫
---

> 缘起：
> 
> 最近想重看一遍漫画《海贼王》，腾讯动漫上有完整的免费资源，先是在腾讯上看了一阵，后来又找了一个海贼小站的专门论坛，在其上的阅读体验更简洁一些。但是，无论网页怎样简化，浏览器的框架和网站的广告总是躲不掉，占据着阅读的视野。  
> 于是，就想着把漫画所有的图片抓取到本地磁盘，然后通过图片查看器开全屏，悠哉悠哉地看……漫……画……

有了需求，说干就干。

# Python实现爬虫，第1版

这段时间又重新把Python用起来了，想的是如果什么问题都用Java解决，方式太重了，而且手里常备一个熟练的动态语言也是极其必要的，所以，稍微简单的任务，比如，文本处理、批量修改文件等，都改用Python了。另外，我在github上维护了一个[pythong小工具工程](https://github.com/quanqinle/my-python)，专门存放日常用到的Python脚本，留档下来方便以后遇到类似问题可以拿来直接用。

所以，这次的漫画图片抓取，我计划用Python编码实现。

此前已经编写过类似的网页文字和多媒体爬取脚本，见上面提到github工程中的`download_mp3_and_dialog.py`文件，它使用的是Lxml库。著名的网页抓取方式还有Beautiful Soup和正则等，当然，也可以用更强大了爬虫框架，只是，我的需求太简单，杀鸡焉用牛刀。

Python编码部分不复杂，编程主要的工作其实在分析网页结构，找到图片下载链接这些工作上。

## 网页分析，CSS元素定位

为了下载漫画，我们分析网页需要搞清楚3部分有用的内容。

### 1.动漫每话的网页地址

经过分析，我发现这个网站在组织每话的url时，规则非常简单且规整。比如，第1话是`www.fuckgfw.com/onepiece/0001`，那么，第188话就是`www.fuckgfw.com/onepiece/0188`，以此类推。

再找到《海贼王》当前的最后一话是917话，剩下的就是for循环和字符串拼接了。

> 重要说明！！！
> 
> 本文仅是技术分享，为了避免有人阅读本文后拿人家的网站做测试，本文隐去抓取的网站域名。这事儿咱一个人干也就干了，可不好传播~~

### 2.每话的题目

页面顶部就有漫画每话的题目，在页面源码中搜索后可以确认 `span标签的class属性值title-comicHeading` 在页面中是唯一的，我们就用它来定位元素，然后获取元素上的文本。

标题的CSS Selector语法是 `span.title-comicHeading` 。

![imag](/img/in-post/selenium-lazyload/selenium-lazyload-02.webp)

### 3.每话中图片资源地址

每话都有数量不等的漫画图片，页面识别后找到可以确保唯一性的图片元素定位方法，所有的图片都在标签 ul > li 列表中，所以，我们可以获取图片img列表，然后遍历img列表，逐个下载即可。

定位img列表的CSS Selector语法是 `ul#comicContain li img` 。

![imag](/img/in-post/selenium-lazyload/selenium-lazyload-03.webp)

## 失败了……

将以上的元素定位写入代码，修改后，先缩小话的for循化范围成从第1话开始到第2话结束，调试运行py。

有点小确幸，多谢佛祖保佑，脚本运行通过，没有报错。

![imag](/img/in-post/selenium-lazyload/selenium-lazyload-04.webp)

打开本地磁盘，查看保存的图片。
糟糕，除了前面几张图，后面全部都是一模一样的小图片，以我的多年工作经验来分析——哎呀妈，这些都是占位图啊！

![imag](/img/in-post/selenium-lazyload/selenium-lazyload-05.webp)

打开网页验证下自己的猜想。

![imag](/img/in-post/selenium-lazyload/selenium-lazyload-07.webp)

确实像猜想的那样，当我没有浏览到处于页面下方的图片时，那里只有占位图，当页面向下滚动快要到达占位图时，img标签的src会被替换成真实的资源url，然后页面才加载图片。

## 如何处理懒加载页面？

那么，问题就显而易见了，这个页面使用了“懒加载（lazy load）”，也叫做“延迟加载”。不过，想想又觉得理所应当，像这样包含大量大图的页面，不延迟加载才是怪事。

> 惰性载入（英语：Lazy loading、Infinite Scroll，又称延迟载入、懒载入、无限卷动、瀑布流），是一种设计模式，被运用在软体设计和网页设计当中，对于网页界面，其特征为使用者透过滑鼠，卷动浏览页面，直到页面下方时，就会自动载入更多内容；有多数网站采用这项网页设计，例如Google图片搜索、Google+、Facebook、Twitter、Pinterest和维基百科的Flow讨论系统。也有结合无限卷动和多页，两著特性的网页设计。
> 
> -- Wikipedia

使用关键字“python 页面抓取 懒加载”在Google搜索一番，又使用相似的英文搜索，结果都指向一种解决方式：**通过Selenium滚动页面触发js加载图片**。

既然要用Selenium，那么，我还是换回Java来实现这次的需求吧，谁让咱是 Java + Selenium老手呢（见历史文章），转而去整Python + Selenium的话，又要重新学习，我可不想重复造轮子。

# Java实现爬虫，第2版

很久前，我就已经在github上开源了一个UITest框架，借助Selenium和Appium，地址是 [WebAndAppUITesting](https://github.com/quanqinle/WebAndAppUITesting) 。这么小的需求没必要使用这套框架，从工程里借鉴一些代码即可。

第1版上我需要增加页面滚动的逻辑，按之前的经验需要加入JavaScript执行器的代码，可用的方式大致有以下几种：

![imag](/img/in-post/selenium-lazyload/selenium-lazyload-08.webp)

不多说了，直接上代码吧

```java
package com.uitest;

import java.io.File;
import java.io.InputStream;
import java.net.URL;
import java.nio.charset.Charset;
import java.nio.file.FileAlreadyExistsException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.TimeUnit;

import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

/**
 * 下载 海贼王 漫画
 * 
 * @author quanqinle
 *
 */
public class DownloadOnePiece {

    public static void main(String[] args) throws Exception {
        download();
    }

    public static void download() throws Exception {

        int firstChap  = 001;
        int newestChap = 947;
        String baseUrl = "https://I.cannot.tell.the.real.url/post/10%03d/";
        String baseDir = "D:\\OnePiece\\%03d\\";
        String baseFile = "D:\\OnePiece\\%03d\\%03d-%03d.jpg";
        String chapterName = ""; // 第2话 戴草帽的路飞

        System.setProperty("webdriver.chrome.driver", "C:\\chromedriver.exe");
        ChromeOptions options = new ChromeOptions();
        options.addArguments("headless");
        WebDriver driver = new ChromeDriver(options);
        driver.get("https://I.cannot.tell.you.the.real.url");
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);

        JavascriptExecutor js = (JavascriptExecutor) driver;

        List<WebElement> imgList = null;
        for (int idxChap = firstChap; idxChap <= newestChap; idxChap++) {
            driver.get(String.format(baseUrl, idxChap));
            Thread.sleep(2 * 1000);

            chapterName = driver.findElement(By.cssSelector("span.title-comicHeading")).getText();
            toLog(String.format("# [%03d] %s", idxChap, chapterName));

            createFolder(String.format(baseDir, idxChap));
            imgList = driver.findElements(By.cssSelector("ul#comicContain li img"));
            int imgIndex = 1;
            for (WebElement img : imgList) {
                if (img.getAttribute("id").contains("adBottom") 
                        || img.getAttribute("id").contains("adTop")
                    || img.getAttribute("src").contains("006xpM3Tgy1feta1hkppuj30m8076wgh.jpg")) {
                    // 漫画中间竟然穿插了广告图！
                    continue;
                }

                /**
                 * 因为页面是懒加载，需滚动页面
                 */
                js.executeScript("arguments[0].scrollIntoView();", img);
                Thread.sleep(300);

                String srcUrl = img.getAttribute("src");
                if (srcUrl.contains("pixel.gif")) {
                    toLog(String.format("fail to load img [%03d - %03d]", idxChap, imgIndex++));
                    continue;
                }

                String destFile = String.format(baseFile, idxChap, idxChap, imgIndex);

                toLog(String.format("下载[%03d]：%s", imgIndex, srcUrl));
                imgIndex++;
                downloadFile(srcUrl, destFile);
            } // 图片循环 end

        } // 页面循环 end

        driver.quit();
    }

    /**
     * 下载指定资源到目标文件
     * 
     * @param url
     * @param destFile
     * @return
     * @throws Exception
     */
    private static boolean downloadFile(String url, String destFile) throws Exception {
        try {
            InputStream in = new URL(url).openStream();
            Files.copy(in, Paths.get(destFile));
        } catch (FileAlreadyExistsException e) {
            toLog("文件已存在：" + destFile);
            return false;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * 创建文件夹
     *
     * @param folderName 文件夹名称
     */
    private static void createFolder(String folderName) {
        File dirFile = new File(folderName);
        boolean bFile = dirFile.exists();
        if (bFile == false) {
            bFile = dirFile.mkdirs();
        }
        if (bFile == true) {
            toLog("Create folder successfully! -- " + folderName);
        } else {
            toLog("Create folder error! -- " + folderName);
        }
    }

    /**
     * 写日志
     */
    public static void toLog(String newline) {
        try {
            List<String> lines = Arrays.asList(newline);
            Path file = Paths.get("D:\\onepiece.log");
            System.out.println(newline);
            Files.write(file, lines, Charset.forName("UTF-8"), StandardOpenOption.CREATE, StandardOpenOption.APPEND);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

需要注意的是，为了效率考虑，实际运行抓取图片时，我使用了“无界面的浏览器”，即代码段 `options.addArguments("headless");` 。

调试的时候可以注释该行，在有界面的情况下，观察运行效果。

另外提一句，我当前使用的chrome和driver在headless模式下出现了bug，变量chapterName总是空字符串，而有界面时，没有这个问题。


## 再次运行代码

再次运行代码，这次世界清静了，程序正常运行没报错，图片也正常下载了。只是代码执行比较慢，因为，为了保障页面加载、图片抓取的成功率，加了一些sleep等待。好在使用headless模式运行，把它收起在后台默默执行，也不影响我做别的事情。

一段时间后……

![imag](/img/in-post/selenium-lazyload/selenium-lazyload-10.webp)

看漫画喽
