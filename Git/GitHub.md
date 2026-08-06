---
title: GitHub 远程仓库操作
tags:
  - git
  - github
  - 远程仓库
  - 笔记
---

# GitHub 远程仓库操作

## 一、创建远程仓库

### 1. 创建远程仓库别名

```bash
git remote -v                     # 查看当前所有远程仓库地址别名
git remote add 别名 远程地址        # 添加远程仓库别名（通常和文件夹名一致）
```

### 2. 推送本地分支到远程仓库

```bash
git push 别名 分支
```

### 3. 拉取

```bash
git pull 别名 分支
```

### 4. 克隆

```bash
git clone 地址
```
