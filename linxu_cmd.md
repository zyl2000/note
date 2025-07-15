# Linux

## 概念

### dpkg

`dpkg` 是 Debian 系 Linux 发行版（如 Ubuntu、Debian、Linux Mint）中用于管理软件包的基础命令行工具，全称为 **Debian Package Manager**。它主要用于直接操作 `.deb` 格式的软件包文件，提供安装、卸载、查询等功能。

### lsb

**LSB** 是 **Linux Standard Base**（Linux 标准基础）的缩写，是一套由 **Linux 基金会**主导制定的规范，旨在解决不同 Linux 发行版之间的兼容性问题，推动 Linux 生态的标准化。

## 操作

### other

```bash
who am i #查询原始用户名
lsb_release -a #查看当前操作系统相关信息
```

### su

```bash
sudo su root # 切换到root用户
```

### dpkg

```bash
dpkg -l | grep zip #列出系统中与 zip 相关的已安装软件包
```

