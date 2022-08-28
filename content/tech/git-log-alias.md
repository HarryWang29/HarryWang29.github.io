---
title: "Git Log Alias"
date: 2022-08-28T23:17:00+08:00
draft: false
description: "终端git log格式化"
tags: 
  - "git"
---

## 设置
```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

## 效果
![](https://user-images.githubusercontent.com/8288067/187081467-382b278e-4bc1-4f12-9d1a-ce29eed72944.png)
base on [gorm](https://github.com/go-gorm/gorm)