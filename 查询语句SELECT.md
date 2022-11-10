# SELCT查询语句



## 最基本的SLECT语句

- SELECT 字段1,字段2,... FROM 表名




### 1. 用于伪表

```sql
SELECT 1+1,3*2;
#or
SELECT 1+1,3*2 FROM DUAL; #dual:伪表
```



### 2. 查询表中所有字段（或列）

```sql
SELECT * FROM employees; # *表示表中所有字段（或列）
```



### 3. 查询指定字段

```sql
SELECT name1,name2 FROM employees; 
```



### 4. 列的别名

```sql
SELECT name another_name FROM employees; #name 后跟空格 别名， 结果显示字段名为别名
#or
SELECT name AS another_name FROM employees; #AS全程 alisa, 可以省略
#or
SELECT id*2 "部门 ID" FROM employees; #使用双引号, 用于特殊场景如带空格的别名使用
```



### 5. 去除重复行

```sql
#查询员工表中一共有哪些部门id
SELECT DISTINCT id,name FROM employees; #注意，多字段去重要注意每个字段的行数一致，否则报错
```



### 6. 空值参与运算

```sql
#1. NULL不等于0,'','null'
#2. 空值参与运算结果为NULL
SELECT id*2 FROM employees;#假设id为NULL，则结果为NULL
/* result
+------+
| id*2 |
+------+
| 2002 |
| 2004 |
| 2006 |
| 2008 |
| 2010 |
| 2010 |
| 2010 |
| NULL |
+------+
*/
#3. 解决方案：引入IFNULL, 替换值
SELECT IFNULL(id,0) FROM employees;#假设id为NULL，则结果为0, 将NULL值替换为何值根据实际选择
/* result
+--------------+
| IFNULL(id,0) |
+--------------+
|         1001 |
|         1002 |
|         1003 |
|         1004 |
|         1005 |
|         1005 |
|         1005 |
|            0 |
+--------------+
*/
```



### 7. 着重号 ``

```sql
#当表名或字段等命名使用关键字时，直接引用会报错
SELECT * FROM ORDER;#报错
SELECT * FROM `order`;#正确
```



### 8. 查询常数

```sql
SELECT '常数',123,id,`name` FROM employees;#添加常数(字符串、数字等)字段
/* result
+------+-----+------+-------+
| 常数 | 123 | id   | name  |
+------+-----+------+-------+
| 常数 | 123 | 1001 | Tom   |
| 常数 | 123 | 1002 | jerry |
| 常数 | 123 | 1003 | 汤姆  |
| 常数 | 123 | 1004 | 杰瑞  |
| 常数 | 123 | 1005 | DOG   |
| 常数 | 123 | 1005 | DOG   |
| 常数 | 123 | 1005 | DOG   |
| 常数 | 123 | NULL | jaker |
+------+-----+------+-------+
*/
```



### 9. 显示表结构

```sql
DESCRIBE employees;
#or
DESC employees;
/* result
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | YES  |     | NULL    |       |
| name  | varchar(15) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
*/
```



### 10. 过滤数据

```sql
SELECT * FROM employees WHERE id = 1001; #WHERE限制条件
SELECT * FROM employees WHERE `name` = "TOM";#字符串过滤，个别平台或数据库大小写模糊
```



### 11. 查询表结构

- DESC [表名];

```sql
DESC departments;
/* result
+-----------------+-------------+------+-----+---------+-------+
| Field           | Type        | Null | Key | Default | Extra |
+-----------------+-------------+------+-----+---------+-------+
| department_id   | int         | NO   | PRI | 0       |       |
| department_name | varchar(30) | NO   |     | NULL    |       |
| manager_id      | int         | YES  | MUL | NULL    |       |
| location_id     | int         | YES  | MUL | NULL    |       |
+-----------------+-------------+------+-----+---------+-------+
*/
```





## 运算符



### 1. 算术运算符

```sql
/*
+：加
-：减
*: 乘
/ or div: 求商(除)
% or mod: 求模(取余)
*/

#特殊情况:
SELECT 100+'1' FROM DUAL; #个别环境下结果是101
SELECT 100+'a' FROM DUAL; #将'a'看作0处理， 结果100
SELECT 100+NULL FROM DUAL; #结果NULL
SELECT 100/0 FROM DUAL; #结果NULL
```



### 2. 比较运算符

- 符号运算符

```sql
/*
=			等于运算符
<=>			安全等于运算符
<> or !=	不等于运算符
<			小于运算符
<=			小于等于运算符
>			大于运算符
>=			大于等于运算符
*/

/*
特殊情况:
1. 不同类型比较，字符串存在隐式转换。如果转换数值不成功，则看作0
2. = <=> is区别：is 专门用来判断是否为 NULL，而 = 则是用来判断非NULL以外的所有数据类型使用。而 <=> 则是前两者合起来。
3. != <> 有NULL参与结果都是NULL
*/

SELECT 1 = 2,1 != 2, 1='1', 1='A', 0='a', '1'=NULL, '1'<=>NULL, NULL != NULL FROM DUAL;
/* result
+-------+--------+-------+-------+-------+----------+------------+--------------+
| 1 = 2 | 1 != 2 | 1='1' | 1='A' | 0='a' | '1'=NULL | '1'<=>NULL | NULL != NULL |
+-------+--------+-------+-------+-------+----------+------------+--------------+
|     0 |      1 |     1 |     0 |     1 | NULL     |          0 | NULL         |
+-------+--------+-------+-------+-------+----------+------------+--------------+
*/
```

- 关键字运算符

```sql
/*
- IS NULL					判断为NULL
- IS NOT NULL				判断不为NULL
- ISNULL					ISNULL(a,b)	判断a是否为NULL，是则将其值替换成b
- LEAST					LEAST(a,b) 比较大小，输出小的字段（字符串比较逻辑与memcmp类似）
- GREATEST				GREATEST(a,b) 比较大小，输出大的字段（字符串比较逻辑与memcmp类似
- BETWEEN ... AND ...		比较区间 eg:...WHERE salary BETWWEN 6000 AND 10000; (等价WHERE salary >= 6000 && salary <= 10000;)
- NOT BETWEEN ... AND ...	比较区间 eg:...WHERE salary NOT BETWWEN 6000 AND 10000; (等价WHERE salary < 6000 or salary > 10000;)
- IN						字段值在该集合里 eg:...WHERE id IN(10,20,30);
- NOT IN					字段值不在该集合里 eg:...WHERE id NOT IN(10,20,30);
- LIKE					模糊查询
						% 代表不确定个数的字符
						_ 代表一个不确定的字符
						eg:	...WHERE last_name LIKE '%a%'; #查询last_name包含字符'a'的员工信息
							...WHERE last_name LIKE 'a%'; #查询last_name字符'a'开头的员工信息
							...WHERE last_name LIKE '%a%' AND last_name LIKE '%a%'; #查询last_name包含字符'a'和'e'的员工信息
							...WHERE last_name LIKE '_a%'; #查询last_name第二个字符'a'的员工信息
							...WHERE last_name LIKE '_\_%'; #查询last_name第二个字符是下划线_的员工信息
							...WHERE last_name LIKE '_$_%' ESCAPE '$'; #效果同上，表示'$'为转义字符，但是该用法不常用，了解就好
- REGEXP / RLIKE			正则表达式查询，详细请了解正则表达式使用方法 eg: ...WHERE last_name REGEXP 'ok$';	#查找字段中以'ok'为结尾员工信息
*/
```



### 3. 逻辑运算符

```sql
/*
NOT or ! 			非
AND or && 			与
OR or ||			或
XOR					异或
*/
```



### 4. 位运算符

```sql
/*
&			按位与
|			按位或
^			按位异或
~			按位取反
>>			按位右移
<<			按位左移
*/
```





## 排序与分页



### 1. 排序

- 如果没有使用排序操作，默认是按照添加数据的顺序排序
- 使用 ORDER BY 对查询到的数据进行排序操作
  - ASC 	升序
  - DESC  降序
- ORDER BY 子句在SELECT语句的结尾（WHERE 需要声明在FROM后，ORDER BY 之前）

- 注意，列的别名只能在排序中使用，与SQL关键字执行顺序有关
- 多列排序的规则为按顺序对排序字段进行优先级检测，字段越靠前优先级越高，只有在优先级高的字段值也一样才会比较下一优先级

```sql
/*
单列排序（一级排序）
*/

SELECT employee_id, salary FROM employees ORDER BY employee_id ASC;	#升序
SELECT employee_id, salary FROM employees ORDER BY employee_id DESC; #降序
SELECT employee_id, salary FROM employees ORDER BY employee_id;		#不带显示指定方式，默认升序
SELECT employee_id alise_id, salary FROM employees ORDER BY alise_id ASC; #ORDER BY可使用别名进行排序（正确）
SELECT employee_id alise_id, salary FROM employees WHERE alise_id > 100; #WHERE不可使用别名进行过滤（错误）
```

```sql
/*
多列排序（二级排序、三级...）
*/
SELECT employee_id, salary FROM employees ORDER BY salary DESC,employee_id ASC;#先按照salary进行降序排序，再按照employee_id进行升序排序
```



### 2. 分页

- mysql使用LIMIT 实现数据分页显示

- LIMIT [位置偏移量], [行数]
- 位偏移量x，行数n：按排序第x行开始显示，显示n行

```sql
SELECT employee_id, last_name FROM employees LIMIT 0,20;	#每页显示20条记录，此时显示第一页
SELECT employee_id, last_name FROM employees LIMIT 20,20;	#每页显示20条记录，此时显示第二页
SELECT employee_id, last_name FROM employees LIMIT 40,20;	#每页显示20条记录，此时显示第三页

/*
LIMIT与其他关键字结合使用
过滤salary大于6000的数据并按照降序排序，然后显示第40条后20条数据
*/
SELECT employee_id, last_name, salary 
FROM employees 
WHERE salary > 6000 
ORDER BY salary DESC 
LIMIT 40,20;
```

- MySQL8.0新特性： LIMIT [行数] OFFSET [位置偏移量]

```sql
SELECT employee_id, last_name FROM employees LIMIT 40,20;	#每页显示20条记录，此时显示第三页
#等价
SELECT employee_id, last_name FROM employees LIMIT 20 OFFSET 40;	#每页显示20条记录，此时显示第三页
```

- 扩展：在不同的 DBMS 中使用的关键字可能不同。
  - 在 MySQL、PostgreSQL、MariaDB 和 SQLite 中使用 LIMIT ；
  - SQL Server 和 Access，需要使用 TOP 关键字；
  - 如果是 DB2，使用 FETCH FIRST 5 ROWS ONLY 这样的关键字；
  - 如果是 Oracle，你需要基于 ROWNUM 来统计行数；







## 多表查询



### 1. 笛卡尔积错误

- 多表查询中因为没有建立多表之间连接引起的多表查询的问题就称之为:“笛卡尔积的*错误*”
- 出现原因：①省略了连接条件。 ②连接条件无效。 ③所有表中的行互相连接。

```sql
SELECT employee_id,department_name FROM employees,departments;#未添加连接条件，将会出现M*N条查询结果（M N为两张表的数据行数）
```



### 2. 多表查询实现

- 增加表的连接条件**（n个表的连接至少要有 n-1 个连接条件）**

```sql
SELECT employee_id,department_name 
FROM employees,departments 
WHERE employees.department_id = departments.department_id; #表的连接条件
```

- 如果多表查询中出现多个表都存在的字段，则必须指明此字段所在的表**（规范建议：多表查询时, 每个字段都指明所属表）**

```sql
#错误：department_id两个表都存在
SELECT employee_id,department_name,department_id
FROM employees,departments 
WHERE employees.department_id = departments.department_id; #表的连接条件

#正确：指明字段所属表
SELECT employee_id,department_name,employees.department_id
FROM employees,departments 
WHERE employees.department_id = departments.department_id; #表的连接条件
```

- 可以在FROM中给表取别名，进行优化**（如果给表起了别名，则在后续操作中必须使用表的别名，而不能使用表的原名）**

```sql
SELECT employee_id,department_name,emp.department_id
FROM employees emp,departments  dept
WHERE emp.department_id = dept.department_id; #表的连接条件
```

- 例子：使用employees\departments\locations 三个表来获取名为'Abel'员工的所在城市

```sql
#先展示表结构
DESC employees;
/*
+----------------+-------------+------+-----+---------+-------+
| Field          | Type        | Null | Key | Default | Extra |
+----------------+-------------+------+-----+---------+-------+
| employee_id    | int         | NO   | PRI | 0       |       |
| first_name     | varchar(20) | YES  |     | NULL    |       |
| last_name      | varchar(25) | NO   |     | NULL    |       |
| email          | varchar(25) | NO   | UNI | NULL    |       |
| phone_number   | varchar(20) | YES  |     | NULL    |       |
| hire_date      | date        | NO   |     | NULL    |       |
| job_id         | varchar(10) | NO   | MUL | NULL    |       |
| salary         | double(8,2) | YES  |     | NULL    |       |
| commission_pct | double(2,2) | YES  |     | NULL    |       |
| manager_id     | int         | YES  | MUL | NULL    |       |
| department_id  | int         | YES  | MUL | NULL    |       |
+----------------+-------------+------+-----+---------+-------+
*/
DESC departments;
/*
+-----------------+-------------+------+-----+---------+-------+
| Field           | Type        | Null | Key | Default | Extra |
+-----------------+-------------+------+-----+---------+-------+
| department_id   | int         | NO   | PRI | 0       |       |
| department_name | varchar(30) | NO   |     | NULL    |       |
| manager_id      | int         | YES  | MUL | NULL    |       |
| location_id     | int         | YES  | MUL | NULL    |       |
+-----------------+-------------+------+-----+---------+-------+
*/
DESC locations;
/*
+----------------+-------------+------+-----+---------+-------+
| Field          | Type        | Null | Key | Default | Extra |
+----------------+-------------+------+-----+---------+-------+
| location_id    | int         | NO   | PRI | 0       |       |
| street_address | varchar(40) | YES  |     | NULL    |       |
| postal_code    | varchar(12) | YES  |     | NULL    |       |
| city           | varchar(30) | NO   |     | NULL    |       |
| state_province | varchar(25) | YES  |     | NULL    |       |
| country_id     | char(2)     | YES  | MUL | NULL    |       |
+----------------+-------------+------+-----+---------+-------+
*/
#以上可见员工名存在于employees表，城市信息位于locations表，departments可以将两表搭桥关联，添加关联条件进行查询

SELECT emp.employee_id,emp.last_name,dept.department_name,emp.department_id,lct.city,lct.location_id #查询字段
FROM employees emp,departments dept, locations lct #查询表并起别名
WHERE emp.department_id = dept.department_id AND lct.location_id = dept.location_id AND emp.last_name = 'Abel' #表的连接条件
ORDER BY emp.employee_id; #排序

/* result
+-------------+-----------+-----------------+---------------+--------+-------------+
| employee_id | last_name | department_name | department_id | city   | location_id |
+-------------+-----------+-----------------+---------------+--------+-------------+
|         174 | Abel      | Sales           |            80 | Oxford |        2500 |
+-------------+-----------+-----------------+---------------+--------+-------------+
*/
```



### 3. 多表查询的分类

- 等值连接 vs 非等值连接
  - 使用 = 连接，即等值连接，其他则为非等值连接

```sql
SELECT employee_id,department_name,employees.department_id
FROM employees,departments 
WHERE employees.department_id = departments.department_id; #等值连接，使用=的连接

SELECT e.last_name, e.salary, j.grade_level
FROM employees e, job_grades j
WHERE e.salary BETWEEN j.lowest_sal AND j.highest_sal #非等值连接
```

- 自连接 vs 非自连接
  - 表内的连接为自链接， 查询字段均属于同张表
  - 不同表连接为非自连接，例子可参考多表查询实现小节

```sql
/*
自连接
查询员工id，员工姓名及其管理者id和姓名
*/
DESC employees;
/*表结构
+----------------+-------------+------+-----+---------+-------+
| Field          | Type        | Null | Key | Default | Extra |
+----------------+-------------+------+-----+---------+-------+
| employee_id    | int         | NO   | PRI | 0       |       |
| first_name     | varchar(20) | YES  |     | NULL    |       |
| last_name      | varchar(25) | NO   |     | NULL    |       |
| email          | varchar(25) | NO   | UNI | NULL    |       |
| phone_number   | varchar(20) | YES  |     | NULL    |       |
| hire_date      | date        | NO   |     | NULL    |       |
| job_id         | varchar(10) | NO   | MUL | NULL    |       |
| salary         | double(8,2) | YES  |     | NULL    |       |
| commission_pct | double(2,2) | YES  |     | NULL    |       |
| manager_id     | int         | YES  | MUL | NULL    |       |
| department_id  | int         | YES  | MUL | NULL    |       |
+----------------+-------------+------+-----+---------+-------+
*/
SELECT emp.employee_id, emp.last_name, mgr.employee_id, mgr.last_name 
FROM employees emp, employees mgr #分别起别名
WHERE emp.manager_id = mgr.employee_id; #将员工的管理者id和身为管理者员工的id连接

/* result
+-------------+-------------+-------------+-----------+
| employee_id | last_name   | employee_id | last_name |
+-------------+-------------+-------------+-----------+
|         101 | Kochhar     |         100 | King      |
|         102 | De Haan     |         100 | King      |
|         103 | Hunold      |         102 | De Haan   |
|         104 | Ernst       |         103 | Hunold    |
|         105 | Austin      |         103 | Hunold    |
|         106 | Pataballa   |         103 | Hunold    |
|         107 | Lorentz     |         103 | Hunold    |
|         108 | Greenberg   |         101 | Kochhar   |
+----------------+-------------+------+-----+---------+
*/
```

- 内连接 vs 外连接

  - 合并具有同一列的两个以上的表的行, **结果集中不包含一个表与另一个表不匹配的行**

  - 两个表在连接过程中除了返回满足连接条件的行以外**还返回左（或右）表中不满足条件的 行 ，这种连接称为左（或右） 外连接。**没有匹配的行时, 结果表中相应的列为空(NULL)。

  - 如果是左外连接，则连接条件中左边的表也称为 主表 ，右边的表称为 从表 。 

    如果是右外连接，则连接条件中右边的表也称为 主表 ，左边的表称为 从表 。

    满外连接，左外和右外均查出来

  - SQL92语法实现外连接：使用 +  (但是在MySQL中不支持SQL92语法外连接写法！)

  - SQL99语法实现外连接：使用 JOIN ... ON（详细请看下一节）

```sql
/*
SQL92外连接例子
查询所有的员工的last_name,department_name信息（注意所有，可能有的员工没有部门）
*/

#SQL92语法实内外连接：以上例子均为内连接，可参考，略
#SQL92语法实现外连接：使用 + (但是在MySQL中不支持SQL92语法外连接写法！)
SELECT employee_id, department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id(+);
```



### 4. SQL99的多表查询

#### SQL99实现内连接

- JOIN ... ON ...

```sql
/*
SQL99内连接例子，多表查询实现例子小节转换
使用employees\departments\locations 三个表来获取名为'Abel'员工的所在城市
*/
SELECT e.last_name,d.department_name,l.city #打印的字段
FROM employees e JOIN departments d			#employees加入departments
ON e.department_id = d.department_id		#连接条件
JOIN locations l							#接入locations表
ON d.location_id = l.location_id			#连接条件
WHERE e.last_name = 'Abel';					#过滤信息
```

#### SQL99实现外连接

- LEFT JOIN \ RIGHT OUTER JOIN ... ON ...

```sql
/*
SQL99左外连接例子
查询所有的员工的last_name,department_name信息（注意所有，可能有的员工没有部门）
*/
SELECT e.last_name,d.department_name
FROM employees e LEFT JOIN departments d #employees主表，departments从表
ON e.department_id = d.department_id; #连接条件

/*
SQL99右外连接例子
查询所有的部门的员工last_name,department_name信息（注意所有，可能有的员工没有部门）
*/
SELECT d.department_name, e.last_name
FROM employees e RIGHT OUTER JOIN departments d #departments主表，employees从表
ON e.department_id = d.department_id; #连接条件
```

#### SQL99实现满外连接

- FULL OUTER JOIN ... ON ...

```
/*
SQL99满外连接例子（注MySQL不支持）
*/
SELECT d.department_name, e.last_name
FROM employees e FULL OUTER JOIN departments d #满外连接
ON e.department_id = d.department_id; #连接条件
```

#### UNION操作符使用

##### UNION概念

- 合并查询结果 利用UNION关键字，可以给出多条SELECT语句，并将它们的结果组合成单个结果集。合并 时，两个表对应的列数和数据类型必须相同，并且相互对应。各个SELECT语句之间使用UNION或UNION ALL关键字分隔。
- UNION：操作符返回两个查询的结果集的并集，去除重复记录。
- UNION ALL：操作符返回两个查询的结果集的并集。对于两个结果集的重复部分，不去重。
- 注意：执行UNION ALL语句时所需要的资源比UNION语句少。如果明确知道合并数据后的结果数据 不存在重复数据，或者不需要去除重复的数据，则尽量使用UNION ALL语句，以提高数据查询的效率。

##### JOINS的7中使用

- 内连接 A ∩ B

```sql
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
ON e.`department_id` = d.`department_id`;
```

- 左外连接 A

```sql
SELECT employee_id,last_name,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`;
```

- 右外连接 B

```sql
SELECT employee_id,last_name,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`;
```

- A - A ∩ B

```sql
SELECT employee_id,last_name,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE d.`department_id` IS NULL
```

- B - A ∩ B

```sql
SELECT employee_id,last_name,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL
```

- 满外连接 A U B  或  A + (B - A ∩ B)

```sql
#方式一 A U B:
SELECT employee_id,last_name,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`;
UNION #去重
SELECT employee_id,last_name,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`;

#方式二 A + (B - A ∩ B):
SELECT employee_id,last_name,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`;
UNION ALL #不去重
SELECT employee_id,last_name,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL

```

- (A - A ∩ B) + (B - A ∩ B)

```sql
SELECT employee_id,last_name,department_name
FROM employees e LEFT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE d.`department_id` IS NULL
UNION ALL
SELECT employee_id,last_name,department_name
FROM employees e RIGHT JOIN departments d
ON e.`department_id` = d.`department_id`
WHERE e.`department_id` IS NULL
```



#### MySQL实现满外连接

- 参考上一节JOINS的7中用法之满外连接
