---
layout:     post
title:      Linux权限——`chmod 777 file`是什么意思
subtitle:   Linux
date:       2020-03-23
author:     CDz
catalog: true
tags:
    - Linux
---

# Linux权限概念

```bash
chmod 777 file
```
是什么意思？

这里有两层意思：
1. 权限
2. 给那些人权限

Linux的权限逻辑是，1️⃣给那些人权限，2️⃣给的这些人能有什么样的操作权限

操作权限：
* r:读，简写数字4
* w:写，简写数字2
* x:执行，简写数字1
任意两种权限可以相加：比如给一个读写权限rw:4+2=6

而我们看到是三个数字组成，这是一种省略写法，其实是：

```bash
chmod u:7 g:7 o:7 file
```

* U:文件所有者
* G:文件所有者所在组
* O:other
所以如果我们设置一个文件权限为：

```bash
chmod 600 file
```

意思为：
* 此文件创建者的权限为rw（读写权限r=4,w=2）
* 此文件创建者所在组没有任何权限
* 其他人没有任何权限
