# MYSQL用户基本操作



## 查看当前所有用户

```sql
SELECT user FROM mysql.user;
```





## 创建子用户

```sql
-- 从192.168.122.12登陆的用户
create user 'superboy'@'192.168.122.12' identified by 'iamsuperboy';
-- 从任意ip登陆的用户
create user 'superboy'@'%' identified by 'iamsuperboy';
-- 不做指定默认为'%'
create user 'superboy' identified by 'iamsuperboy';
```



## 修改用户密码

```sql
-- 使用update指令，注意这里的password()需要进行加密
use mysql;
update user set password = password('iamsuperman') where user = 'superboy';
flush privileges;
-- -----------------或者------------------
set password for superboy@'localhost'= password('iamsuperman');
flush privileges;
-- -----------------或者------------------
#当前两者无法成功时使用,且为明文
ALTER USER superboy@localhost IDENTIFIED BY 'newpasswd3';


```



## 赋予用于权限

```sql
-- 赋予部分权限，其中的shopping.*表示对以shopping所有文件操作。
grant select,delete,update,insert on simpleshop.* to superboy@'localhost' identified by 'superboy';
flush privileges;
-- 赋予所有权限
grant all privileges on simpleshop.* to superboy@localhost identified by 'iamsuperboy';
flush privileges;
```



## 撤销用户权限

```sql
-- 撤销update权限
revoke update on simpleshop.* from superboy@localhost;
-- 撤销所有权限
revoke all on simpleshop.* from superboy@localhost;
```



## 删除用户

```sql
use mysql;
delete from user where user='superboy' and host='localhost' ;
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

