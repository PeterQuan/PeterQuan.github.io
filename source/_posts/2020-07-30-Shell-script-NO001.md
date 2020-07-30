---
layout:       post
title:        "Shell | 删除最老的一份备份"
subtitle:     "Linux Shell脚本积累 #1。为了避免备份太占空间，定期清理老的备份"
date:         2020-07-30
updated:      2020-07-30
author:       "权芹乐"
catalog:      true
tags:
    - Linux
    - Shell
    - Script
---

[toc]

## 备份文件夹多于5个的话，删除最老的那个文件夹

假设：`/home/backup/`下会定时生成备份目录，为省空间，最多允许保存`5`份备份，如果超过则删除最老的一份。

```sh
#!/bin/sh

SAVED_NUM=5
num=`ls -l /home/backup/ | grep '^d' | wc -l`;
if [ $num -ge $SAVED_NUM ]
then
    rmfolder=`ls -tr /home/backup/ | head -n1`
    echo "To be deleted: ${rmfolder}"
    rm -rf "/home/backup/${rmfolder}"
fi
```
