

# SELCT查询语句





## SELECT 语句的完整结构

- 写在前面

```sql
#方式1(sql92)：
SELECT ...,....,...
FROM ...,...,....
WHERE 多表的连接条件
AND 不包含组函数的过滤条件
GROUP BY ...,...
HAVING 包含组函数的过滤条件
ORDER BY ... ASC/DESC
LIMIT ...,...

#方式2(sql99)：
SELECT ...,....,...
FROM ... JOIN ...
ON 多表的连接条件
JOIN ...
ON ...
WHERE 不包含组函数的过滤条件
AND/OR 不包含组函数的过滤条件
GROUP BY ...,...
HAVING 包含组函数的过滤条件
ORDER BY ... ASC/DESC
LIMIT ...,...

#其中：
#（1）from：从哪些表中筛选
#（2）on：关联多表查询时，去除笛卡尔积
#（3）where：从表中筛选的条件
#（4）group by：分组依据
#（5）having：在统计结果中再次筛选
#（6）order by：排序
#（7）limit：分页
```

- SELECT 语句顺序

  - 关键字的顺序是不能颠倒

  ```sql
  SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT...
  ```

  - SELECT 语句的执行顺序（在 MySQL 和 Oracle 中，SELECT 执行顺序基本相同）

  ```sql
  FROM -> WHERE -> GROUP BY -> HAVING -> SELECT 的字段 -> DISTINCT -> ORDER BY -> LIMIT
  ```

  ```sql
  #example:
  SELECT DISTINCT player_id, player_name, count(*) as num # 顺序 5
  FROM player JOIN team ON player.team_id = team.team_id # 顺序 1
  WHERE height > 1.80 # 顺序 2
  GROUP BY player.team_id # 顺序 3
  HAVING num > 2 # 顺序 4
  ORDER BY num DESC # 顺序 6
  LIMIT 2 # 顺序 7
  ```

- 在 SELECT 语句执行这些步骤的时候，每个步骤都会产生一个 虚拟表 ，然后将这个虚拟表传入下一个步 骤中作为输入。需要注意的是，这些步骤隐含在 SQL 的执行过程中，对于我们来说是不可见的。



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
<=>			安全等于运算符 (可以用来对 NULL 进行判断，两者都为 NULL 时返回值为 1)
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

##### JOINS的7种使用

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



#### SQL99语法新特性

##### 1. 自然连接

- NATURAL JOIN 
- 它会帮你字段查询两张连接表中**所有相同的字段**，然后进行等值连接

```sql
#SQL92
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
ON e.`department_id` = d.`department_id`
AND e.`manager_id` = d.`manager_id`;

#SQL99
SELECT employee_id,last_name,department_name
FROM employees e NATURAL JOIN departments d;
```

##### 2. USING连接

- 当我们进行连接的时候，SQL99还支持使用 USING 指定数据表里的 同名字段 进行等值连接。但是只能配 合JOIN一起使用。

```sql
SELECT employee_id,last_name,department_name
FROM employees e ,departments d
WHERE e.department_id = d.department_id;
#等价
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
USING (department_id);
```



#### 注意

- 我们要 控制连接表的数量 。多表连接就相当于嵌套 for 循环一样，非常消耗资源，会让 SQL 查询性能下 降得很严重，因此不要连接不必要的表。在许多 DBMS 中，也都会有最大连接表的限制。
- 【强制】**超过三个表禁止 join**。需要 join 的字段，数据类型保持绝对一致；多表关联查询时， 保证被关联的字段需要有**索引**。
- 说明：即使双表 join 也要注意表索引、SQL 性能。







## 函数



### 单行函数



#### 1. 基本函数

| 函数                | 用法                                                         |
| ------------------- | ------------------------------------------------------------ |
| ABS(x)              | 返回x的绝对值                                                |
| SIGN(X)             | 返回X的符号。正数返回1，负数返回-1，0返回0                   |
| PI()                | 返回圆周率的值                                               |
| CEIL(x)             | CEILING(x) 返回大于或等于某个值的最小整数                    |
| FLOOR(x)            | 返回小于或等于某个值的最大整数                               |
| LEAST(e1,e2,e3…)    | 返回列表中的最小值                                           |
| GREATEST(e1,e2,e3…) | 返回列表中的最大值                                           |
| MOD(x,y)            | 返回X除以Y后的余数                                           |
| RAND()              | 返回0~1的随机值                                              |
| RAND(x)             | 返回0~1的随机值，其中x的值用作种子值，相同的X值会产生相同的随机<br/>数 |
| ROUND(x)            | 返回一个对x的值进行四舍五入后，最接近于X的整数               |
| ROUND(x,y)          | 返回一个对x的值进行四舍五入后最接近X的值，并保留到小数点后面Y位 |
| TRUNCATE(x,y)       | 返回数字x截断为y位小数的结果                                 |
| SQRT(x)             | 返回x的平方根。当X的值为负数时，返回NULL                     |



#### 2. 数学函数

| 函数                 | 用法                                                         |
| -------------------- | ------------------------------------------------------------ |
| RADIANS(x)           | 将角度转化为弧度，其中，参数x为角度值                        |
| DEGREES(x)           | 将弧度转化为角度，其中，参数x为弧度值                        |
| SIN(x)               | 返回x的正弦值，其中，参数x为弧度值                           |
| ASIN(x)              | 返回x的反正弦值，即获取正弦为x的值。如果x的值不在-1到1之间，则返回NULL |
| COS(x)               | 返回x的余弦值，其中，参数x为弧度值                           |
| ACOS(x)              | 返回x的反余弦值，即获取余弦为x的值。如果x的值不在-1到1之间，则返回NULL |
| TAN(x)               | 返回x的正切值，其中，参数x为弧度值                           |
| ATAN(x)              | 返回x的反正切值，即返回正切值为x的值                         |
| ATAN2(m,n)           | 返回两个参数的反正切值                                       |
| COT(x)               | 返回x的余切值，其中，X为弧度值                               |
| POW(x,y)，POWER(X,Y) | 返回x的y次方                                                 |
| EXP(X)               | 返回e的X次方，其中e是一个常数，2.718281828459045             |
| LN(X)，LOG(X)        | 返回以e为底的X的对数，当X <= 0 时，返回的结果为NULL          |
| LOG10(X)             | 返回以10为底的X的对数，当X <= 0 时，返回的结果为NULL         |
| LOG2(X)              | 返回以2为底的X的对数，当X <= 0 时，返回NULL                  |



#### 3. 进制转换函数

| 函数          | 用法                     |
| ------------- | ------------------------ |
| BIN(x)        | 返回x的二进制编码        |
| HEX(x)        | 返回x的十六进制编码      |
| OCT(x)        | 返回x的八进制编码        |
| CONV(x,f1,f2) | 返回f1进制数变成f2进制数 |



#### 4. 字符串函数

| 函数                              | 用法                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| ASCII(S)                          | 返回字符串S中的第一个字符的ASCII码值                         |
| CHAR_LENGTH(s)                    | 返回字符串s的字符数。作用与CHARACTER_LENGTH(s)相同           |
| LENGTH(s)                         | 返回字符串s的字节数，和字符集有关                            |
| CONCAT(s1,s2,......,sn)           | 连接s1,s2,......,sn为一个字符串                              |
| CONCAT_WS(x, s1,s2,......,sn)     | 同CONCAT(s1,s2,...)函数，但是每个字符串之间要加上            |
| INSERT(str, idx, len, replacestr) | 将字符串str从第idx位置开始，len个字符长的子串替换为字符串replacestr |
| REPLACE(str, a, b)                | 用字符串b替换字符串str中所有出现的字符串a                    |
| UPPER(s) 或 UCASE(s)              | 将字符串s的所有字母转成大写字母                              |
| LOWER(s) 或LCASE(s)               | 将字符串s的所有字母转成小写字母                              |
| LEFT(str,n)                       | 返回字符串str最左边的n个字符                                 |
| RIGHT(str,n)                      | 返回字符串str最右边的n个字符                                 |
| LPAD(str, len, pad)               | 用字符串pad对str最左边进行填充，直到str的长度为len个字符     |
| RPAD(str ,len, pad)               | 用字符串pad对str最右边进行填充，直到str的长度为len个字符     |
| LTRIM(s)                          | 去掉字符串s左侧的空格                                        |
| RTRIM(s)                          | 去掉字符串s右侧的空格                                        |
| TRIM(s)                           | 去掉字符串s开始与结尾的空格                                  |
| TRIM(s1 FROM s)                   | 去掉字符串s开始与结尾的s1                                    |
| TRIM(LEADING s1 FROM s)           | 去掉字符串s开始处的s1                                        |
| TRIM(TRAILING s1 FROM s)          | 去掉字符串s结尾处的s1                                        |
| REPEAT(str, n)                    | 返回str重复n次的结果                                         |
| SPACE(n)                          | SPACE(n)                                                     |
| STRCMP(s1,s2)                     | STRCMP(s1,s2)                                                |
| SUBSTR(s,index,len)               | 返回从字符串s的index位置其len个字符，作用与SUBSTRING(s,n,len)、 MID(s,n,len)相同 |
| LOCATE(substr,str)                | 返回字符串substr在字符串str中首次出现的位置，作用于POSITION(substr IN str)、INSTR(str,substr)相同。未找到，返回0 |
| ELT(m,s1,s2,…,sn)                 | 返回指定位置的字符串，如果m=1，则返回s1，如果m=2，则返回s2，如 果m=n，则返回sn |
| FIELD(s,s1,s2,…,sn)               | 返回字符串s在字符串列表中第一次出现的位置                    |
| FIND_IN_SET(s1,s2)                | 返回字符串s1在字符串s2中出现的位置。其中，字符串s2是一个以逗号分 隔的字符串 |
| REVERSE(s)                        | 返回s反转后的字符串                                          |
| NULLIF(value1,value2)             | 比较两个字符串，如果value1与value2相等，则返回NULL，否则返回 value1 |

**注意：MySQL中，字符串的位置是从1开始的。**



#### 5. 日期和时间函数

##### 5.1 获取时间

| 函数                                                         | 用法                            |
| ------------------------------------------------------------ | ------------------------------- |
| CURDATE() ，CURRENT_DATE()                                   | 返回当前日期，只包含年、 月、日 |
| CURTIME() ， CURRENT_TIME()                                  | 返回当前时间，只包含时、 分、秒 |
| NOW() / SYSDATE() / CURRENT_TIMESTAMP() / LOCALTIME() / LOCALTIMESTAMP() | 返回当前系统日期和时间          |
| UTC_DATE()                                                   | 返回UTC（世界标准时间） 日期    |
| UTC_TIME()                                                   | 返回UTC（世界标准时间） 时间    |

##### 5.2 日期与时间戳的转换

| 函数                     | 用法                                                         |
| ------------------------ | ------------------------------------------------------------ |
| UNIX_TIMESTAMP()         | 以UNIX时间戳的形式返回当前时间。SELECT UNIX_TIMESTAMP() - >1634348884 |
| UNIX_TIMESTAMP(date)     | 将时间date以UNIX时间戳的形式返回。                           |
| FROM_UNIXTIME(timestamp) | 将UNIX时间戳的时间转换为普通格式的时间                       |

##### 5.3 获取月份、星期、星期数、天数等函数

| 函数                                     | 用法                                             |
| ---------------------------------------- | ------------------------------------------------ |
| YEAR(date) / MONTH(date) / DAY(date)     | 返回具体的日期值                                 |
| HOUR(time) / MINUTE(time) / SECOND(time) | 返回具体的时间值                                 |
| MONTHNAME(date)                          | 返回月份：January，..                            |
| DAYNAME(date)                            | 返回星期几：MONDAY，TUESDAY.....SUNDAY           |
| WEEKDAY(date)                            | 返回周几，注意，周1是0，周2是1，。。。周日是6    |
| QUARTER(date)                            | 返回日期对应的季度，范围为1～4                   |
| WEEK(date) ， WEEKOFYEAR(date)           | 返回一年中的第几周                               |
| DAYOFYEAR(date)                          | 返回日期是一年中的第几天                         |
| DAYOFMONTH(date)                         | 返回日期位于所在月份的第几天                     |
| DAYOFWEEK(date)                          | 返回周几，注意：周日是1，周一是2，。。。周六是 7 |
| EXTRACT(type FROM date)                  | EXTRACT(type FROM date)                          |

##### 5.4  时间和秒钟转换的函数

| 函数                 | 用法                                                         |
| -------------------- | ------------------------------------------------------------ |
| TIME_TO_SEC(time)    | 将 time 转化为秒并返回结果值。转化的公式为： 小时*3600+分钟 *60+秒 |
| SEC_TO_TIME(seconds) | 将 seconds 描述转化为包含小时、分钟和秒的时间                |

##### 5.5  计算日期和时间的函数

| 函数                                                         | 用法                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| DATE_ADD(datetime, INTERVAL expr type)， ADDDATE(date,INTERVAL expr type) | 返回与给定日期时间相差INTERVAL时 间段的日期时间              |
| DATE_SUB(date,INTERVAL expr type)， SUBDATE(date,INTERVAL expr type) | 返回与date相差INTERVAL时间间隔的 日期                        |
| ADDTIME(time1,time2)                                         | 返回time1加上time2的时间。当time2为一个数字时，代表的是 秒 ，可以为负数 |
| SUBTIME(time1,time2)                                         | 返回time1减去time2后的时间。当time2为一个数字时，代表的 是 秒 ，可以为负数 |
| DATEDIFF(date1,date2)                                        | 返回date1 - date2的日期间隔天数                              |
| TIMEDIFF(time1, time2)                                       | 返回time1 - time2的时间间隔                                  |
| FROM_DAYS(N)                                                 | 返回从0000年1月1日起，N天以后的日期                          |
| TO_DAYS(date)                                                | 返回日期date距离0000年1月1日的天数                           |
| LAST_DAY(date)                                               | 返回date所在月份的最后一天的日期                             |
| MAKEDATE(year,n)                                             | 针对给定年份与所在年份中的天数返回一个日期                   |
| MAKETIME(hour,minute,second)                                 | 将给定的小时、分钟和秒组合成时间并返回                       |
| PERIOD_ADD(time,n)                                           | 返回time加上n后的时间                                        |

##### 5.6 日期的格式化与解析

| 函数                              | 用法                                       |
| --------------------------------- | ------------------------------------------ |
| DATE_FORMAT(date,fmt)             | 按照字符串fmt格式化日期date值              |
| TIME_FORMAT(time,fmt)             | 按照字符串fmt格式化时间time值              |
| GET_FORMAT(date_type,format_type) | 返回日期字符串的显示格式                   |
| STR_TO_DATE(str, fmt)             | 按照字符串fmt对str进行解析，解析为一个日期 |



| 字符    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| %Y      | 4位数字表示年份                                              |
| %y      | 表示两位数字表示年份                                         |
| %M      | 月名表示月份（January,....）                                 |
| %m      | 两位数字表示月份 （01,02,03。。。）                          |
| %b      | 缩写的月名（Jan.，Feb.，....）                               |
| %c      | 数字表示月份（1,2,3,...）                                    |
| %D      | 英文后缀表示月中的天数 （1st,2nd,3rd,...）                   |
| %d      | 两位数字表示月中的天数(01,02...)                             |
| %e      | 数字形式表示月中的天数 （1,2,3,4,5.....）                    |
| %H      | 两位数字表示小数，24小时制 （01,02..）                       |
| %h 和%I | 两位数字表示小时，12小时制 （01,02..）                       |
| %k      | 数字形式的小时，24小时制(1,2,3)                              |
| %l      | 数字形式表示小时，12小时制 （1,2,3,4....）                   |
| %i      | 两位数字表示分钟（00,01,02）                                 |
| %S 和%s | 两位数字表示秒(00,01,02...)                                  |
| %W      | 一周中的星期名称（Sunday...）                                |
| %a      | 一周中的星期缩写（Sun.， Mon.,Tues.，..）                    |
| %w      | 以数字表示周中的天数 (0=Sunday,1=Monday....)                 |
| %j      | 以3位数字表示年中的天数(001,002...)                          |
| %U      | 以数字表示年中的第几周， （1,2,3。。）其中Sunday为周中第一 天 |
| %u      | 以数字表示年中的第几周， （1,2,3。。）其中Monday为周中第一 天 |
| %T      | 24小时制                                                     |
| %p      | AM或PM                                                       |
| %%      | 表示%                                                        |



#### 6. 流程处理函数

| 函数                                                         | 用法                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| IF(value,value1,value2)                                      | 如果value的值为TRUE，返回value1， 否则返回value2             |
| IFNULL(value1, value2)                                       | 如果value1不为NULL，返回value1，否 则返回value2              |
| CASE WHEN 条件1 THEN 结果1 WHEN 条件2 THEN 结果2 .... [ELSE resultn] END | 相当于Java的if...else if...else...                           |
| CASE expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 .... [ELSE 值n] END | CASE expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 .... [ELSE 值n] END |

```sql
SELECT
	CASE
WHEN 1 > 0 THEN
	'1 > 0'
WHEN 2 > 0 THEN
	'2 > 0'
ELSE
	'3 > 0'
END;

/* result 
+-----------------------------------------------------------------+
| 1 > 0                                                           |
+-----------------------------------------------------------------+
*/
```



#### 7. 加密与解密函数

| 函数                        | 用法                                                         |
| --------------------------- | ------------------------------------------------------------ |
| PASSWORD(str)               | 返回字符串str的加密版本，41位长的字符串。加密结果 不可 逆 ，常用于用户的密码加密 |
| MD5(str)                    | 返回字符串str的md5加密后的值，也是一种加密方式。若参数为 NULL，则会返回NULL |
| SHA(str)                    | 从原明文密码str计算并返回加密后的密码字符串，当参数为 NULL时，返回NULL。 SHA加密算法比MD5更加安全 。 |
| ENCODE(value,password_seed) | 返回使用password_seed作为加密密码加密value                   |
| DECODE(value,password_seed) | 返回使用password_seed作为加密密码解密value                   |



#### 8. MySQL信息函数

| 函数                                                   | 用法                                                      |
| ------------------------------------------------------ | --------------------------------------------------------- |
| VERSION()                                              | 返回当前MySQL的版本号                                     |
| CONNECTION_ID()                                        | 返回当前MySQL服务器的连接数                               |
| DATABASE()，SCHEMA()                                   | 返回MySQL命令行当前所在的数据库                           |
| USER()，CURRENT_USER()、SYSTEM_USER()， SESSION_USER() | 返回当前连接MySQL的用户名，返回结果格式为 “主机名@用户名” |
| CHARSET(value)                                         | 返回字符串value自变量的字符集                             |
| COLLATION(value)                                       | 返回字符串value的比较规则                                 |

#### 9. 其他类别

| 函数                           | 用法                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| FORMAT(value,n)                | 返回对数字value进行格式化后的结果数据。n表示 四舍五入 后保留 到小数点后n位 |
| CONV(value,from,to)            | 将value的值进行不同进制之间的转换                            |
| INET_ATON(ipvalue)             | 将以点分隔的IP地址转化为一个数字(\# 以“192.168.1.100”为例，计算方式为192乘以256的3次方，加上168乘以256的2次方，加上1乘以256，再加上 100。) |
| INET_NTOA(value)               | 将数字形式的IP地址转化为以点分隔的IP地址                     |
| BENCHMARK(n,expr)              | 将表达式expr重复执行n次。用于测试MySQL处理expr表达式所耗费 的时间 |
| CONVERT(value USING char_code) | 将value所使用的字符编码修改为char_code                       |





### 聚合函数

- 聚合函数作用于一组数据，并对一组数据返回一个值。

- 注意：MySQL中聚合函数不能嵌套，Oracle可以


#### 1. 常见的几个聚合函数

##### 1.1 AVG / SUM

- 只适用于数值类型的字段（变量）

```sql
SELECT AVG(salary), SUM(salary)
FROM employees;

/*
+-------------+-------------+
| AVG(salary) | SUM(salary) |
+-------------+-------------+
| 6461.682243 |   691400.00 |
+-------------+-------------+
1 row in set
*/
```

##### 1.2 MAX / MIN

- 适用于数值、字符串、日期时间类型的字段（变量）

```sql
SELECT MIN(salary), MAX(salary)
FROM employees;

/*
+-------------+-------------+
| MIN(salary) | MAX(salary) |
+-------------+-------------+
|        2100 |       24000 |
+-------------+-------------+
1 row in set
*/
```

##### 1.3 COUNT

- 作用：计算指定字段在查询结构中出现的个数（不包含NULL值）
- 统计标中的记录数，使用COUNT(*), COUNT(1), COUNT(具体字段)，哪个效率更高（不同引擎不同，MISAM三者效率相同O(1)，InnoDB则COUNT(\*) = COUNT(1) > COUNT(字段)
- COUNT和**带偏移Offset的LIMIT**不能同时使用（SELECT COUNT(*) FROM table LIMIT 2,3） COUNT结果是NULL

```sql
#计算employees表中salary字段的条数， 并计算这个表总条数
SELECT COUNT(salary), COUNT(*), COUNT(department_id)
FROM employees;

/*
+---------------+----------+----------------------+
| COUNT(salary) | COUNT(*) | COUNT(department_id) |
+---------------+----------+----------------------+
|           107 |      107 |                  106 |
+---------------+----------+----------------------+
1 row in set
*/
```



#### 2. GROUP BY

- 将表中数据分为若干组
- SELECT中出现的非组函数的字段必须声明在GROUP BY中。 反之， GROUP BY 中声明的字段可以不出现在SELECT中

```sql
#计算各个部门的平均工资和总工资
SELECT department_id, AVG(salary), SUM(salary)
FROM employees
GROUP BY department_id;
/*
+---------------+--------------+-------------+
| department_id | AVG(salary)  | SUM(salary) |
+---------------+--------------+-------------+
| NULL          |  7000.000000 |     7000.00 |
|            10 |  4400.000000 |     4400.00 |
|            20 |  9500.000000 |    19000.00 |
|            30 |  4150.000000 |    24900.00 |
|            40 |  6500.000000 |     6500.00 |
|            50 |  3475.555556 |   156400.00 |
|            60 |  5760.000000 |    28800.00 |
|            70 | 10000.000000 |    10000.00 |
|            80 |  8955.882353 |   304500.00 |
|            90 | 19333.333333 |    58000.00 |
|           100 |  8600.000000 |    51600.00 |
|           110 | 10150.000000 |    20300.00 |
+---------------+--------------+-------------+
*/

#查询各个department_id，job_id的平均工资
SELECT department_id, job_id, AVG(salary)
FROM employees
GROUP BY department_id, job_id;
/*
+---------------+------------+--------------+
| department_id | job_id     | AVG(salary)  |
+---------------+------------+--------------+
|            90 | AD_PRES    | 24000.000000 |
|            90 | AD_VP      | 17000.000000 |
|            60 | IT_PROG    |  5760.000000 |
|           100 | FI_MGR     | 12000.000000 |
|           100 | FI_ACCOUNT |  7920.000000 |
|            30 | PU_MAN     | 11000.000000 |
|            30 | PU_CLERK   |  2780.000000 |
|            50 | ST_MAN     |  7280.000000 |
|            50 | ST_CLERK   |  2785.000000 |
|            80 | SA_MAN     | 12200.000000 |
|            80 | SA_REP     |  8396.551724 |
| NULL          | SA_REP     |  7000.000000 |
|            50 | SH_CLERK   |  3215.000000 |
|            10 | AD_ASST    |  4400.000000 |
|            20 | MK_MAN     | 13000.000000 |
|            20 | MK_REP     |  6000.000000 |
|            40 | HR_REP     |  6500.000000 |
|            70 | PR_REP     | 10000.000000 |
|           110 | AC_MGR     | 12000.000000 |
|           110 | AC_ACCOUNT |  8300.000000 |
+---------------+------------+--------------+
*/

#错误写法：job_id没出现在GROUP BY中 无法进行正确分组
SELECT department_id, job_id, AVG(salary)
FROM employees
GROUP BY department_id;
```

- MYSQL中GROUP BY中使用WITH ROLLUP（多一组，将整体看作一组）。**注意ROLLUP 和 ORDER BY是互斥的，不可同时使用。**

```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id WITH ROLLUP; #多一条记录， 全部的平均数
+---------------+--------------+
| department_id | AVG(salary)  |
+---------------+--------------+
| NULL          |  7000.000000 |
|            10 |  4400.000000 |
|            20 |  9500.000000 |
|            30 |  4150.000000 |
|            40 |  6500.000000 |
|            50 |  3475.555556 |
|            60 |  5760.000000 |
|            70 | 10000.000000 |
|            80 |  8955.882353 |
|            90 | 19333.333333 |
|           100 |  8600.000000 |
|           110 | 10150.000000 |
| NULL          |  6461.682243 |
+---------------+--------------+
```



#### 3. HAVING

- 用来过滤数据
- **如果过滤条件中使用了聚合函数，则必须使用HAVING来替代WHERE。否则，报错。**
- HAVING必须声明在GROUP BY后面
- 在开发中，使用HAVING的前提是SQL中使用了GROUP BY

```sql
#查询各个部门中最高工资比1000高的部门信息
SELECT department_id, MAX(salary)
FROM employees
GROUP BY department_id
HAVING MAX(salary) > 10000;
/*
+---------------+-------------+
| department_id | MAX(salary) |
+---------------+-------------+
|            20 |       13000 |
|            30 |       11000 |
|            80 |       14000 |
|            90 |       24000 |
|           100 |       12000 |
|           110 |       12000 |
+---------------+-------------+
*/
```

- 当过滤条件中有聚合函数时，则此过滤条件必须声明在HAVING中
- 当过滤条件中没有聚合函数时， 则此过滤条件声明在WHERE中或HAVING中都可以， 但是建议声明在WHERE（**效率更高**）

```sql
#方式1 效率更高
SELECT department_id, MAX(salary)
FROM employees
WHERE department_id IN (20,30)
GROUP BY department_id
HAVING MAX(salary) > 10000;

#方式1 不建议
SELECT department_id, MAX(salary)
FROM employees
GROUP BY department_id
HAVING MAX(salary) > 10000 AND department_id IN (20,30);
```





## 子查询

- 子查询指一个查询语句嵌套在另一个查询语句内部的查询
- **在SELECT中， 除了GROUP BY 和LIMIT 外， 其他位置都可以声明子查询**
- 当可以使用自连接的时候，建议使用自连接，因为相同复杂度下自连接处理比子查询快（具体情况具体分析）、

```sql
#查询工资大于 Abel工资的员工
SELECT
	last_name,
	salary
FROM
	employees
WHERE
	salary > (
		SELECT
			salary
		FROM
			employees
		WHERE
			last_name = 'Abel'
	);
```

- 子查询（内查询）在主查询之前一次执行完成。 
- 子查询的结果被主查询（外查询）使用 。
-  注意事项 
  - 子查询要包含在括号内
  -  将子查询放在比较条件的右侧
  -  单行操作符对应单行子查询，多行操作符对应多行子查询



### 1. 子查询的分类

#### 1.1 单行 vs 多行 子查询

- 我们按内查询的结果返回一条还是多条记录，将子查询分为 单行子查询 、 多行子查询 。

##### 1.1.2 单行子查询

- 操作符：=, >, >=, <, <=, <>

```sql
#查询与141号员工的manager_id，department_id相同的其他员工的employee_id，manager_id，department_id
SELECT
	employee_id,
	manager_id,
	department_id
FROM
	employees
WHERE
	manager_id = (
		SELECT
			manager_id
		FROM
			employees
		WHERE
			employee_id = 141
	)
AND department_id = (
	SELECT
		department_id
	FROM
		employees
	WHERE
		employee_id = 141
)
and employee_id != 141;

#查询最低工资大于50号部门最低工资的部门id和其最低工资
SELECT
	department_id,
	MIN(salary)
FROM
	employees
GROUP BY
	department_id
HAVING
	MIN(salary) > (
		SELECT
			MIN(salary)
		FROM
			employees
		WHERE
			department_id = 110
	);

#题目：显式员工的employee_id,last_name和location。
#其中，若员工department_id与location_id为1800的department_id相同，
#则location为’Canada’，其余则为’USA’。
SELECT
	employee_id,
	last_name,
	CASE department_id
WHEN (
	SELECT
		department_id
	FROM
		departments
	WHERE
		location_id = 1800
) THEN
	'Canada'
ELSE
	'USA'
END "location",
 department_id
FROM
	employees;
```

##### 1.1.2 多行子查询 

| 操作符 | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| IN     | 等于列表中的**任意一个**                                     |
| ANY    | 需要和单行比较操作符一起使用，和子查询返回的**某一个**值比较（或 成立） |
| ALL    | 需要和单行比较操作符一起使用，和子查询返回的**所有**值比较 （与 成立） |
| SOME   | 实际上是ANY的别名，作用相同，一般常使用ANY                   |

- IN

```sql
#查询与141号和160号员工的manager_id，department_id相同的其他员工的employee_id，manager_id，department_id
SELECT
	employee_id,
	manager_id,
	department_id
FROM
	employees
WHERE
	manager_id IN (
		SELECT
			manager_id
		FROM
			employees
		WHERE
			employee_id IN (141, 160)
	)
AND department_id IN (
	SELECT
		department_id
	FROM
		employees
	WHERE
		employee_id IN (141, 160)
)
and employee_id != 141 AND employee_id != 160;
```

- ANY / ALL （ANY相当于 **或**， ALL相当于 **与**）

```sql
#ANY
#题目：返回其它job_id中比job_id为‘IT_PROG’部门任一工资低的员工的员工号、
#姓名、job_id 以及salary
SELECT
	employee_id,
	last_name,
	job_id,
	salary
FROM
	employees
WHERE
	salary < ALL (
		SELECT
			salary
		FROM
			employees
		WHERE
			job_id = 'IT_PROG'
	)
AND job_id != 'IT_PROG';

#题目：返回其它job_id中比job_id为‘IT_PROG’部门所有工资低的员工的员工号、
#姓名、job_id 以及salary
SELECT
	employee_id,
	last_name,
	job_id,
	salary
FROM
	employees
WHERE
	salary < ALL (
		SELECT
			salary
		FROM
			employees
		WHERE
			job_id = 'IT_PROG'
	)
AND job_id != 'IT_PROG';

#题目：查询平均工资最低的部门id
#MySQL中聚合函数是不能嵌套使用的。
SELECT
	department_id,
	AVG(salary)
FROM
	employees
GROUP BY
	department_id
HAVING
	AVG(salary) <= ALL (
		SELECT
			AVG(salary)
		FROM
			employees
		GROUP BY
			department_id
		HAVING
			department_id IS NOT NULL
	)
AND department_id IS NOT NULL;
```



#### 1.2 相关 vs 不相关 子查询

##### 1.2.1 相关子查询

- 我们按内查询是否被执行多次，将子查询划分为 相关(或关联)子查询 和 不相关(或非关联)子查询 。 
- 子查询从数据表中查询了数据结果，如果这个数据结果只执行一次，然后这个数据结果作为主查询的条件进行执行，那么这样的子查询叫做不相关子查询。 
- 同样，如果子查询需要执行多次，即采用循环的方式，先从外部查询开始，每次都传入子查询进行查询，然后再将结果反馈给外部，这种嵌套的执行方式就称为相关子查询。

![](D:\git_clone\MySQL_Note\MySQL_Note\相关子查询.png)

相关子查询example: （以上例子均为不相关子查询）

```sql
#题目：查询员工中工资大于本部门平均工资的员工的last_name,salary和其department_id
#方式1：使用相关子查询
SELECT
	last_name,
	salary,
	department_id
FROM
	employees e1
WHERE
	salary > (
		SELECT
			AVG(salary)
		FROM
			employees e2
		WHERE
			department_id = e1.department_id
	);

#方式2：在FROM中声明子查询, 创建一个虚拟表进行比较查询
SELECT
	last_name,
	salary,
	department_id
FROM
	employees e1
JOIN (
	SELECT
		department_id,
		AVG(salary) avg_sal
	FROM
		employees
	GROUP BY
		department_id
) t_dept_avg_sal USING (department_id)
WHERE
	e1.salary > t_dept_avg_sal.avg_sal
AND e1.department_id = t_dept_avg_sal.department_id;


#题目：查询员工的id,salary,按照department_name 降序排序
SELECT
department_id,
	employee_id,
	salary
FROM
	employees e
ORDER BY
	(
		SELECT
			department_name
		FROM
			departments d
		WHERE
			e.department_id = d.department_id
	)
	DESC;
```



##### 1.2.2 EXISTS

- EXISTS 与 NOT EXISTS 关键字

- 关联子查询通常也会和 EXISTS操作符一起来使用，用来检查在子查询中是否存在满足条件的行。 
- **如果在子查询中不存在满足条件的行：** 
  - 条件返回 FALSE
  - 继续在子查询中查找 
- **如果在子查询中存在满足条件的行：**
  -  不在子查询中继续查找
  - 条件返回 TRUE 
- NOT EXISTS关键字表示如果不存在某种条件，则返回TRUE，否则返回FALSE。



EXISTS example:

```sql
#题目：查询公司管理者的employee_id，last_name，job_id，department_id信息(是管理者的员工)
#方式1：自连接
SELECT DISTINCT mgr.employee_id,mgr.last_name,mgr.job_id,mgr.department_id
FROM employees emp JOIN employees mgr
ON emp.manager_id = mgr.employee_id;

#方式2：子查询

SELECT employee_id,last_name,job_id,department_id
FROM employees
WHERE employee_id IN (
			SELECT DISTINCT manager_id
			FROM employees
			);

#方式3：使用EXISTS
SELECT employee_id,last_name,job_id,department_id
FROM employees e1
WHERE EXISTS (
	       SELECT *
	       FROM employees e2
	       WHERE e1.`employee_id` = e2.`manager_id`
	     );
```

NOT EXISTS example:

```sql
#题目：查询departments表中，不存在于employees表中的部门的department_id和department_name
SELECT
	department_id,
	department_name
FROM
	departments d
WHERE
	NOT EXISTS (
		SELECT
			*
		FROM
			employees e
		WHERE
			e.department_id = d.department_id
	);
```

