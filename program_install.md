# clickhouse

clickhouse安装按照官网提供的完整版步骤进行。

# mysql

## 安装

### **方法一：通过 APT 官方源安装（简单快捷）**

适用于快速安装稳定版本（如 Ubuntu 22.04 默认安装 MySQL 8.0）。

#### **步骤 1：更新软件包索引**

```bash
sudo apt update
```

#### **步骤 2：安装 MySQL 服务器**

```bash
sudo apt install mysql-server
```

安装过程中会提示输入 root 密码（如果没有提示，后续可通过命令设置）。

#### **步骤 3：验证安装**

```bash
sudo systemctl status mysql  # 检查服务状态
mysql --version  # 查看版本
```

若服务未启动，可手动启动：

```bash
sudo systemctl start mysql
sudo systemctl enable mysql  # 设置开机自启
```

#### **步骤 4：配置安全选项**

```bash
sudo mysql_secure_installation
```

按提示设置：

- 移除匿名用户
- 禁止 root 远程登录
- 移除测试数据库
- 刷新权限表

## 异常情况处理

如果通过 APT 安装 MySQL 时没有提示设置 root 密码，默认会使用 **auth_socket** 认证插件，这意味着 root 用户只能通过系统用户（如 sudo）登录，而无需密码。以下是设置 root 密码的详细步骤：

### **步骤 1：以系统管理员身份登录 MySQL**

```bash
sudo mysql  # 无需密码，通过系统用户权限登录
```

成功登录后，会进入 MySQL 命令行提示符：

```mysql
Welcome to the MySQL monitor...
mysql>
```

### **步骤 2：修改 root 用户认证方式并设置密码**

#### **方法一：使用 caching_sha2_password 认证（推荐）**

```sql
-- 修改 root 用户密码并切换到密码认证
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'your_new_password';

-- 刷新权限使更改生效
FLUSH PRIVILEGES;

-- 退出 MySQL
EXIT;
```

#### **方法二：使用 mysql_native_password 认证（兼容性更好）**

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_new_password';
FLUSH PRIVILEGES;
EXIT;
```

### **步骤 3：验证密码登录**

退出 MySQL 后，尝试用密码登录：

```bash
mysql -u root -p
```

输入刚设置的密码，若成功登录，则配置完成。

### **步骤 4：（可选）禁用 auth_socket 认证**

若希望 root 始终通过密码登录，需修改配置文件：

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

在 `[mysqld]` 部分添加或修改：

```conf
[mysqld]
skip-grant-tables = 0  # 确保不禁用权限表
```

重启 MySQL 服务：

```bash
sudo systemctl restart mysql
```