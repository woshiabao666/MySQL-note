# 一、Linux环境下MySQL

## 1.1 SQL大小写规范

### 1.1.1 **Windows和Linux平台区别**

> **在 SQL 中，关键字和函数名是不用区分字母大小写的，比如 SELECT、WHERE、ORDER、GROUP BY 等关键字，以及 ABS、MOD、ROUND、MAX 等函数名。**
>
> **不过在 SQL 中，你还是要确定大小写的规范，因为在 Linux 和 Windows 环境下，你可能会遇到不同的大小写问题。 windows系统默认大小写不敏感 ，但是 linux系统是大小写敏感的**

> **MySQL在Linux下数据库名、表名、列名、别名大小写规则**
>
> - **==数据库名、表名、表的别名、变量名是严格区分大小写的；==**
> - **==关键字、函数名称在 SQL 中不区分大小写；==**
> - **==列名（或字段名）与列的别名（或字段别名）在所有的情况下均是忽略大小写的；==**
>
> **`MySQL在Windows的环境下全部不区分大小写`**

### 1.1.2 SQL编写建议

- **关键字和函数名称全部大写；**
- **数据库名、表名、表别名、字段名、字段别名等全部小写；**
- **SQL 语句必须以分号结尾。**



# 二、MySQL目录结构与表结构管理

# 三、用户与权限管理

## 3.1 用户管理

### 3.1.1 登录MySQL服务器

> **启动MySQL服务后，可以通过mysql命令来登录MySQL服务器**

```sql
mysql –h hostname|hostIP –P port –u username –p DatabaseName –e "SQL语句"

mysql -uroot -p -hlocalhost -P3306 mysql -e "select host,user from user"
```

> - **`-h参数` 后面接主机名或者主机IP，hostname为主机，hostIP为主机IP。**
> - -**`P参数` 后面接MySQL服务的端口，通过该参数连接到指定的端口。MySQL服务的默认端口是3306，不使用该参数时自动连接到3306端口，port为连接的端口号**。
> - **`-u参数` 后面接用户名，username为用户名。**
> - **`-p参数` 会提示输入密码。**
> - **`DatabaseName参数` 指明登录到哪一个数据库中。如果没有该参数，就会直接登录到MySQL数据库中，然后可以使用USE命令来选择数据库。**
> - **`-e参数` 后面可以直接加SQL语句。登录MySQL服务器以后即可执行这个SQL语句，然后退出MySQL服务器。**

### 3.1.2 创建用户

```sql
CREATE USER 用户名 [IDENTIFIED BY '密码'][,用户名 [IDENTIFIED BY '密码']];
```

> - **用户名参数表示新建用户的账户，由 用户（User） 和 主机名（Host） 构成；**
> - **“[ ]”表示可选，也就是说，可以指定用户登录时需要密码验证，也可以不指定密码验证，这样用户可以直接登录。不过，不指定密码的方式不安全，不推荐使用。如果指定密码值，这里需要使用IDENTIFIED BY指定明文密码值。**
> - **CREATE USER语句可以同时创建多个用户。**
> - **==flush privileges==**

- **举例**

```sql
CREATE USER zhang3 IDENTIFIED BY '123123'; # 默认host是 %

CREATE USER 'kangshifu'@'localhost' IDENTIFIED BY '123456';
```

### 3.1.3 修改用户

```sql
UPDATE mysql.user SET USER='li4' WHERE USER='wang5'; 
FLUSH PRIVILEGES;
```

### 3.1.4 删除用户

> **1.使用DROP方式删除（推荐）**

```sql
DROP USER user[,user]…;
```

- **举例**

```sql
DROP USER li4 ; 	# 默认删除host为%的用户 
DROP USER 'kangshifu'@'localhost';
```

> **2.使用DELETE方式删除**

```sql
DELETE FROM mysql.user WHERE Host=’hostname’ AND User=’username’;
FLUSH PRIVILEGES;
```

- **举例**

```sql
DELETE FROM mysql.user WHERE Host='localhost' AND User='Emily'; 
FLUSH PRIVILEGES;
```

### 3.1.5 设置当前用户密码

> **1.使用ALTER USER命令来修改当前用户密码**

```sql
ALTER USER USER() IDENTIFIED BY 'new_password';
```

> **2. 使用SET语句来修改当前用户密码**

```sql
SET PASSWORD='new_password';
```

### 3.1.6 修改其它用户密码

> **1.** **使用ALTER语句来修改普通用户的密码**

```sql
ALTER USER user [IDENTIFIED BY '新密码'] 
[,user[IDENTIFIED BY '新密码']]…;
```

> **2.使用SET命令来修改普通用户的密码**

```sql
SET PASSWORD FOR 'username'@'hostname'='new_password';
```



## 3.2 权限管理

# 四、逻辑架构

## 4.1 MySQL执行流程









































