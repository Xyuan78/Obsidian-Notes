---
title: Git
tags:
  - git
  - 版本控制
  - 笔记
---

# Git

## 一、Git 常用命令

| 命令 | 说明 |
|------|------|
| `git init` | 初始化仓库 |
| `git status` | 查看本地库状态 |
| `git add 文件名` | 添加到暂存区 |
| `git commit -m "日志信息" 文件名` | 提交到本地库 |
| `git reflog` | 查看历史版本号 |
| `git reset --hard 版本号` | 版本穿梭 |
| `git log` | 查看详细版本号 |

> `ls -la`：Git Bash 下查看隐藏目录

### 1. 创建文件

`vim hello.txt` 使用了 [[vim]] 用法

### 2. 添加暂存区

```bash
git add hello.txt
```

### 3. 提交到本地库

```bash
git commit -m "my first commit" hello.txt
```

### 4. 查看历史版本号

```bash
git reflog
```

### 5. 修改文件

```bash
vim hello.txt
```

### 6. 版本穿梭

```bash
git reflog       # 找到版本号
git reset --hard 版本号
```

`.git` 下面有个 HEAD 指向分支，`refs` 目录找到 `heads`，下面有个 `master`，里面放着版本号

## 二、Git 分支

### 1. 什么是分支

同时推进多个任务，为每个任务创建单独的分支，不会影响主线分支的运行

### 2. 分支的操作

| 命令 | 说明 |
|------|------|
| `git branch 分支名` | 创建分支 |
| `git branch -v` | 查看分支 |
| `git checkout 分支名` | 切换分支 |
| `git merge 分支名` | 把指定的分支合并到当前分支上 |

### 3. 连接 [[GitHub]]
