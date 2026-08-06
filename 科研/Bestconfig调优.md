---
title: Bestconfig 调优 — MySQL + BenchBase 环境配置
tags:
  - 科研
  - mysql
  - benchbase
  - 数据库
  - ubuntu
  - 备份
  - 笔记
---

# Bestconfig 调优 — MySQL + BenchBase 环境配置

## 一、Ubuntu 上的设置

### 1. 目前机器上的各种路径

#### Java / Maven

- Java 23 （2024-09-17）`/usr/lib/jvm/jdk23/bin/java`
- Maven 3.8.7 `/usr/bin/mvn`

#### MySQL

- MySQL 8.0.45
- 目录：`/usr/bin/mysql`
- 用户及密码：`root` / `Ks15639605518@`，`benchbase` / `Ks15639605518@`

#### Benchbase

- 路径：主文件夹下有 `benchbase/target/benchbase-mysql`

```bash
# BenchBase 的 MySQL TPC-C 配置文件路径
~/benchbase/target/benchbase-mysql/config/mysql/sample_tpcc_config.xml

# 编译命令
java -jar benchbase.jar -b tpcc -c config/mysql/sample_tpcc_config.xml --create=true --load=true --execute=true
```

### 2. MySQL 备份

#### 第一步：停止 MySQL 服务

```bash
sudo systemctl stop mysql
sudo systemctl status mysql    # 验证已停止（显示 inactive）
```

#### 第二步：删除旧备份

```bash
sudo rm -rf /var/lib/mysql_backup
sudo ls -ld /var/lib/mysql_backup   # 验证删除成功
```

#### 第三步：重新创建备份目录 + 完整备份

```bash
# 1. 重新创建备份目录
sudo mkdir -p /var/lib/mysql_backup

# 2. 完整复制 MySQL 数据目录（-a 保留所有属性）
# 用 `.` 代替 `*`，确保 #innodb_redo 等隐藏目录被完整复制
sudo cp -a /var/lib/mysql/. /var/lib/mysql_backup/

# 3. 修复备份目录权限
sudo chown -R mysql:mysql /var/lib/mysql_backup
```

#### 第四步：验证备份完整性

```bash
# 检查是否包含 #innodb_redo（MySQL 8.0+ 核心）
sudo ls -la /var/lib/mysql_backup | grep "#innodb_redo"

# 检查关键文件
sudo ls /var/lib/mysql_backup | grep -E "ibdata1|ib_logfile|#innodb_redo"
```

#### 第五步：重启 MySQL 服务

```bash
sudo systemctl start mysql
sudo systemctl status mysql   # 验证正常启动（显示 active (running)）
```

### 3. MySQL 恢复

#### 第一步：停止 MySQL 服务

```bash
sudo systemctl stop mysql
sudo systemctl status mysql
```

#### 第二步：完整恢复备份数据

```bash
sudo cp -a /var/lib/mysql_backup/. /var/lib/mysql/
```

#### 第三步：修复恢复后的数据目录权限

```bash
sudo chown -R mysql:mysql /var/lib/mysql
```

#### 第四步：对比备份前后

```bash
# 目录大小对比
echo "=== MySQL 目录大小对比 ==="
sudo du -sh /var/lib/mysql
sudo du -sh /var/lib/mysql_backup

# 文件数量对比
echo "=== 文件/目录数量对比 ==="
echo "原目录总数：$(sudo find /var/lib/mysql | wc -l)"
echo "备份目录总数：$(sudo find /var/lib/mysql_backup | wc -l)"
```
