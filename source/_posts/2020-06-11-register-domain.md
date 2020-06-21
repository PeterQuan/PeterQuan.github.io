---
layout:       post
title:        "在porkbun注册域名"
subtitle:     "注册域名，并使用cloudflare的免费CDN服务"
date:         2020-06-11
author:       "Quan Qinle"
multilingual: false
catalog:      true
tags:
    - 域名
    - DNS
    - CDN
---

# 选择域名服务商

+ 开始时，考虑了`namecheap`、`namesilo`、`godaddy`
+ 网上搜索比较后，优先`namesilo`
+ 最终通过比价，选择`porkbun`。对于初级、低端域名玩家，*便宜*就是吸引力。

> 域名服务商价格比较工具：[www.domcomp.com](https://www.domcomp.com/)

另外，到期后，我考虑转到`namesilo`。

# 在`porkbun`注册域名

[porkbun](https://porkbun.com/)的特点：
1. 便宜
2. Free WHOIS Privacy：免费的Whois隐私保护
3. Free SSL Certificates：免费的SSL

操作过程简单，不再赘述。

# 增改`DNS RECORDS`

点击`DNS RECORDS`，可以看到默认已经提供了一些记录，它们都指向`porkbun`的默认网址。

因为我暂时只搭了博客，所以，添加一条DNS记录指向博客，其他的暂时保持原样。

+ 添加`CNAME类型`的记录
+ `Host`填写blog，这个域名已在git中配置
+ `Answer`填写我的github.io地址

> 【Tip】网址DNS记录最常用的类型是CNAME与ALIAS，简单说明二者的区别：
> + CNAME：解析到其他域名
> + ALIAS：解析到ip

至此，域名注册和绑定已经完成，可以正常使用了。

如果想使用CDN，需要继续下面的步骤。

# 把nameservers转移到`cloudflare`

先看[porkbun官方操作指引](https://kb.porkbun.com/article/22-how-to-change-your-nameservers)

> tip: [cloudflare](https://dash.cloudflare.com/)提供免费的CDN服务

1. 登陆`cloudflare`，添加自己的域名

2. `cloudflare`将自动扫描 DNS 记录
    + **黄色云朵**表示该解析通过 CDN 访问（隐藏网站真实 IP 地址，保护原站安全。）
    + **灰色云朵**表示不通过 CDN 访问（相当于只使用 cloudflare 的 DNS 功能）
    + 点击云朵可以切换状态。
    + 手工添加缺失的记录

3. 在原域名供应商`porkbun`处，点击`AUTHORITATIVE NAMESERVERS`并修改

删除以下所有
```
curitiba.porkbun.com
fortaleza.porkbun.com
maceio.porkbun.com
salvador.porkbun.com
```

替换为
```
dalary.ns.cloudflare.com
rohin.ns.cloudflare.com
```

4. 剩下就是等待`cloudflare`变成激活状态：`Status: Activity`