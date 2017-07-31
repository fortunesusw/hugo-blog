---
title: git备忘
<!-- subtitle: git 常用命令 -->
date: 2017-07-31
tags: ["git"]
---

>git 常用命令

<!--more-->

## 查询远程分支

```git
git ls-remote --heads origin
```

## 删除远程分支

```git
git push origin --delete <branch>
```

## 创建远程分支

```git
git checkout -b <branch>
git push -u origin <branch>
```

## 放弃本地修改

```git
git reset HEAD~1
git reset --soft HEAD^
```

## 本地项目推送到github

```
1. github上先创建一个项目
2. 本地git init
3. git add .
4. git commit -m "init project" .
5. git git remote add origin git@github.com:$username/$project.git
6. git remote -v  #看状态
7. git push -u origin master
```


