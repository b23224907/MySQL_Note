# MYSQL用户基本操作



## Ubuntu20.04 安装mysql

```
sudo apt update
sudo apt install mysql-server
```



## 查看当前所有用户

```sql
SELECT user FROM mysql.user;
```





## 创建子用户

```sql
-- 从192.168.122.12登陆的用户
create user 'username'@'192.168.122.12' identified by 'password';
-- 从任意ip登陆的用户
create user 'username'@'%' identified by 'password';
-- 不做指定默认为'%'
create user 'username' identified by 'password';


```



## 修改用户密码

```sql
-- 使用update指令，注意这里的password()需要进行加密
use mysql;
update user set password = password('password') where user = 'username';
flush privileges;
-- -----------------或者------------------
set password for username@'localhost'= password('password');
flush privileges;
-- -----------------或者------------------
#当前两者无法成功时使用,且为明文
ALTER USER username@localhost IDENTIFIED BY 'newpassword';

#mysql8.0 想要使用明文密码，需指定加密方式为mysql_native_password
alter user 'mycli'@'%' identified with mysql_native_password by 'mysql2324907';
```



## 赋予用于权限

```sql
-- 赋予部分权限，其中的shopping.*表示对以shopping所有文件操作。
grant select,delete,update,insert on simpleshop.* to 'username'@'localhost' identified by 'password';
flush privileges;
-- 赋予所有权限
grant all privileges on simpleshop.* to 'username'@'localhost' identified by 'password';
flush privileges;

#mysql8.0
grant all privileges on *.* to 'mycli'@'%';
flush privileges;
```



## 撤销用户权限

```sql
-- 撤销update权限
revoke update on simpleshop.* from username@localhost;
-- 撤销所有权限
revoke all on simpleshop.* from username@localhost;
```



## 删除用户

```sql
use mysql;
delete from user where user='username' and host='localhost' ;
flush privileges;
```



## 登陆操作

### 1.本地登录MySQL

```sql
mysql -u root -p   #root是用户名，输入这条命令按回车键后系统会提示你输入密码
```

### 2.指定端口号登录MySQL数据库

```sql
mysql -u root -p -P 3306  #注意指定端口的字母P为大写
```

### 3.指定IP地址和端口号登录MySQL数据库

```sql
mysql -h ip -u root -p -P 3306 
#例如：mysql -h 127.0.0.1 -u root -p -P 3306
```



## 卸载mysql问题

### 1. 因安装问题导致mysql卸载出错

```
命令行执行mysql命令时出错，提示：

Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'
1
原因是mysql配置文件有问题，避免麻烦，这里直接卸载重装，注意备份数据和自定义的配置文件。

依次执行如下命令
删除依赖包：
sudo rm -rf /var/lib/mysql/ -R

删除配置文件：
sudo rm -rf /etc/mysql/ -R

卸载相关软件：
sudo apt autoremove mysql* --purge
sudo apt remove apparmor

清理卸载残留
sudo apt-get autoremove
sudo apt-get clean
```





## Mysql无法远程访问问题



### 因mysql只监听127.0.01解决方法

- 环境：mysql8.0，ubuntu，已关闭防火墙

```
打开配置文件编辑
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

修改内容
bind-address           = 0.0.0.0

重新运行mysql
sudo systemctl restart mysql
```





## 修改密码加密方式

### mysql8.0默认密码加密, 修改成不需要加密

```sql
use mysql;
update user set plugin='mysql_native_password' where User='uesrname';
sudo systemctl restart mysql
```

