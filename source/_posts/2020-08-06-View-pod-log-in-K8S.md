---
layout:       post
title:        "k8s | 查看k8s中应用的运行log"
subtitle:     "假设我们有一个应用运行在k8s中，本文记录查看应用运行日志的方法"
date:         2020-08-06
updated:      2020-08-06
author:       "权芹乐"
catalog:      true
tags:
    - k8s
    - kubernetes
---

[toc]

自从应用的部署从tomcat转移到kubernetes之后，再也不能像下面这样一句命令查看日志了：
```
tail -f /usr/local/tomcat6/logs/catalina.out
```

在我记住k8s操作步骤之前，暂时记录下来以备忘。

## 1. 找到pod
```
kubectl get pods -n=test -o wide
// or
kubectl get pods --namespace=test -o wide

NAME                       READY   STATUS    RESTARTS   AGE   IP        NODE        NOMINATED NODE   READINESS GATES
mysql-df95554c9-hszrl      1/1     Running   0          39d   a.b.c.d   k8s-node4   <none>           <none>
my-app-7c8fb75b76-spm8n    2/2     Running   0          45m   a.b.c.d   k8s-node1   <none>           <none>
```

* `-n=test`: 指定命名空间
* `-o wide`: 显示更多内容

## 2. 找到container
当pod上有多个container时，需要明确查看哪个。

查找container的方式有两个，
1. 在应用部署的对应yaml文件中找到containers->name
2. 在pod详情中找Container:
```
kubectl describe pod my-app-7c8fb75b76-spm8n -n=test
```

如果缺省这步的话，查看日志可能会遇到下面的提示：
```
[root@k8s-master ~]# kubectl logs -f my-app-7c8fb75b76-spm8n -n test --tail=500 --v=1
Error from server (BadRequest): a container name must be specified for pod my-app-7c8fb75b76-spm8n, choose one of: [my-app filebeat]
```

## 3. 查看日志
```
kubectl logs -f my-app-7c8fb75b76-spm8n -c partner-kpi -n test --tail=500 --v=1
```
* `-c partner-kpi`: container容器
* `--tail=500`: 显示最新500行。可以换成`--since=1h`，这个也很实用
* `--v=1`: 日志级别1

另外，k8s的命令中=号可以省略。