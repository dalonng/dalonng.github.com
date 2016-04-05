---
layout: post
title: 防止push 到主线的 git hook
published: true
category: git
tags:
    - git
---

# 防止push 到主线的 git hook

## 开启 pre-push hook
```Shell
# 在工程目录下
➜  mv .git/hooks/pre-push.sample .git/hooks/pre-push
```

## 打开`pre-push`文件 添加如下代码
```Shell
if [[ $remote_ref == *"master"* ]]
then
	echo "好好看看在push"
	exit 1
fi
```
