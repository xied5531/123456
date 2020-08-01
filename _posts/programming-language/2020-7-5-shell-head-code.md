---
title: shell脚本，通用标准头信息
date: 2020-7-5 09:30:57
tags: shell
---

## 标准头信息

```bash
# cat /root/test.sh 
#!/usr/bin/env bash

self_fullname=$(readlink -f $0)
self_basename="$(basename ${self_fullname})"
self_dirname="$(readlink -f $(dirname ${self_fullname}))"

echo "self_fullname=$self_fullname"
echo "self_basename=$self_basename"
echo "self_dirname=$self_dirname"

```

## 输出内容


```
# sh /root/test.sh 
self_fullname=/root/test.sh
self_basename=test.sh
self_dirname=/root
```
