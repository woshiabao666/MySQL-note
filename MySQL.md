

## 一、分组与聚合函数

### 1.1 group by

- **SELECT中出现的非组函数的字段必须出现在GROUP BY 中；反之，GROUP BY 中声明的字段不一定出现在SELECT 中**

### 1.2 HAVING

- **如果过滤条件中出现了组函数，则必须使用HAVING来替换WHERE，否则会报错，且HAVING放在GROUP BY 的后面**
- **当过滤条件当中没有聚合函数，则此过滤条件声明在WHERE或者HAVING中都可以；==建议使用WHERE（效率更高）==**

### 1.3 聚合函数与单行函数

- **MySQL中单行函数可以嵌套使用，==聚合函数不可以嵌套使用（Oracle可以）==**



### 1.3 SQL执行顺序

- **==FROM -> WHERE -> GROUP BY -> HAVING -> 聚合函数 -> SELECT -> DISTINCT -> ORDER BY -> LIMIT==**



## 二、子查询 

- **==空值问题==**

- **==在查询语句中，除了GROUP BY 与 LIMIT 外，其他位置都可以声明子查询==**

- **关联子查询通常会和==EXISTS==操作符一起来使用，用来检查子查询中是否存在满足条件的行，==如果子查询中不存在条件满足的行，条件返回FALSE并继续查找；如果子查询中存在满足条件的行，不在子查询中继续查找，条件返回TRUE==**

  ```sql
  -- 查询公司管理者的employee_id,last_name,job_id,department_id信息
  SELECT
  	emp1.employee_id,
  	emp1.last_name,
  	emp1.job_id,
  	emp1.department_id 
  FROM
  	employees emp1 
  WHERE
  	EXISTS (
  	SELECT
  		*
  	FROM 
  		employees emp2 
  	WHERE
  	emp1.employee_id = emp2.manager_id)
  ```

  

## 三、约束

- **概述**

```sql
-- 查看约束
select * from information_schema.table_constraints where table_name = 'employees'
```



## 四、变量，流程控制，游标

### 4.1 系统变量

> 系统变量分为==全局系统变量（需要添加 global 关键字）==以及==会话系统变量（需要添加 session关键字）==，
>
> 有时也把全局系统变量简称为全局变量，有时也把会话系统变量称为local变量。**如果不写，默认**
>
> **会话级别。**静态变量（在 MySQL 服务实例运行期间它们的值不能使用 set 动态修改）属于特殊的全局系
>
> 统变量。

#### 4.1.1 查看系统变量

```sql
#查看所有全局变量 
SHOW GLOBAL VARIABLES; 
#查看所有会话变量 
SHOW SESSION VARIABLES; 
SHOW VARIABLES;
```

```sql
#查看满足条件的部分系统变量。 
SHOW GLOBAL VARIABLES LIKE '%标识符%'; 
#查看满足条件的部分会话变量 
SHOW SESSION VARIABLES LIKE '%标识符%';
```

#### 4.1.2 修改系统变量

- **方式一：修改MySQL配置文件，继而修改MySQL系统变量的值**
- **方式二：在MySQL运行期间，使用"set"命令重新设置系统变量的值**
- **==全局系统变量：针对于当前数据库实例是有效的，一旦重启MySQL服务就失效了==**

```sql
-- 为全局变量赋值
#方式一：
SET @@global.变量名=变量值; 
#方式2： 
SET GLOBAL 变量名=变量值; 


-- 为某个会话变量赋值 
#方式1： 
SET @@session.变量名=变量值; 
#方式2： 
SET SESSION 变量名=变量值;
```

### 4.2 用户变量

> **用户变量是用户自己定义的，作为 MySQL 编码规范，MySQL 中的用户变量以 一个“@” 开头。根据作用**
>
> **范围不同，又分为 `会话用户变量` 和 `局部变量` 。**

- **==会话用户变量：作用域和会话变量一样，只对当前连接会话有效。==**
- **==局部变量：只在 BEGIN 和 END 语句块中有效。局部变量只能在存储过程和函数中使用==**

#### 4.2.1 **会话用户变量**

```sql
#方式1：“=”或“:=” 
SET @用户变量 = 值; 
SET @用户变量 := 值; 

#方式2：“:=” 或 INTO关键字 
SELECT @用户变量 := 表达式 [FROM 等子句]; 
SELECT 表达式 INTO @用户变量 [FROM 等子句];

# 查看用户变量
SELECT @用户变量
```

#### 4.2.2 局部变量

> **定义：可以使用 DECLARE 语句定义一个局部变量**
>
> **作用域：仅仅在定义它的 BEGIN ... END 中有效**
>
> **位置：只能放在 BEGIN ... END 中，而且只能放在第一句**



```sql
BEGIN
#声明局部变量 
DECLARE 变量名1 变量数据类型 [DEFAULT 变量默认值]; 
DECLARE 变量名2,变量名3,... 变量数据类型 [DEFAULT 变量默认值]; 
#为局部变量赋值 
SET 变量名1 = 值; 
SELECT 值 INTO 变量名2 [FROM 子句]; 
#查看局部变量的值 
SELECT 变量1,变量2,变量3; 
END
```

#### 4.2.3 小结

|              | 作用域            | 定义位置            | 语法                      |
| ------------ | ----------------- | ------------------- | ------------------------- |
| 会话用户变量 | 当前会话          | 会话的任何地方      | 加@符号，不用指定类型     |
| 局部变量     | 定义在BEGIN END中 | BEGIN END的第一句话 | 一般不用加@，需要指定类型 |



### 4.3 流程控制

#### 4.3.1 分支结构 IF

- IF语法结构

```sql
IF 表达式1 THEN 操作1 
[ELSEIF 表达式2 THEN 操作2]…… 
[ELSE 操作N] 
END IF
```

- **IF相关案例**

```sql
-- 声明存储过程“update_salary_by_eid1”，定义IN参数emp_id，输入员工编号。判断该员工
-- 薪资如果低于8000元并且入职时间超过5年，就涨薪500元；否则就不变
DELIMITER $
CREATE PROCEDURE update_salary_by_eid1 ( IN emp_id INT ) BEGIN
	DECLARE
		emp_salary DOUBLE ( 8, 2 );
	DECLARE
		hire_time DOUBLE;
		
		# 员工工作时长
	SELECT
		TIMESTAMPDIFF( YEAR, emp.hire_date,CURRENT_DATE ) INTO hire_time 
	FROM
		employees emp 
	WHERE
		emp.employee_id = emp_id;
		
		# 员工薪资
	SELECT
		emp.salary INTO emp_salary 
	FROM
		employees emp 
	WHERE
		emp.employee_id = emp_id;
		
		# 判断
	IF emp_salary < 8000 AND hire_time > 5 THEN
		UPDATE employees 
		SET salary = salary + 500 
		WHERE
			employee_id = emp_id;
		ELSE SELECT
			'不符合加薪条件';
		
	END IF;
	
END $
DELIMITER;
```



#### 4.3.2 分支结构 CASE

- **CASE语法结构一：**

```sql
#情况一：类似于switch 
CASE 表达式 
WHEN 值1 THEN 结果1或语句1(如果是语句，需要加分号) 
WHEN 值2 THEN 结果2或语句2(如果是语句，需要加分号) 
... 
ELSE 结果n或语句n(如果是语句，需要加分号) 
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）
```

- **CASE语法结构二：**

```sql
#情况二：类似于多重if 
CASE WHEN 条件1 THEN 结果1或语句1(如果是语句，需要加分号) 
WHEN 条件2 THEN 结果2或语句2(如果是语句，需要加分号) 
... 
ELSE 结果n或语句n(如果是语句，需要加分号) 
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）
```



- **相关案例**

```sql
#举例1:基本使用
DELIMITER //
CREATE PROCEDURE test_case()
BEGIN

	#演示1：case ... when ...then ...
	/*
	declare var int default 2;
	
	case var
		when 1 then select 'var = 1';
		when 2 then select 'var = 2';
		when 3 then select 'var = 3';
		else select 'other value';
	end case;
	*/
	
	#演示2：case when ... then ....
	DECLARE var1 INT DEFAULT 10;
	CASE 
	WHEN var1 >= 100 THEN SELECT '三位数';
	WHEN var1 >= 10 THEN SELECT '两位数';
	ELSE SELECT '个数位';
	END CASE;

END //

DELIMITER ;


```

#### 4.3.3 循环结构 LOOP

> **LOOP循环语句用来重复执行某些语句。LOOP内的语句一直重复执行直到循环被退出（使用LEAVE子**
>
> **句），跳出循环过程。【初始化条件，循环条件，循环体，迭代条件】**

- **基本格式**

```sql
[loop_label:] LOOP
	循环执行语句
END LOOP [loop_label]
```

- **案例一**

```sql
DECLARE id INT DEFAULT 0; 
add_loop:LOOP 
SET id = id +1; 
IF id >= 10 THEN LEAVE add_loop; 
END IF;
END LOOP add_loop;
```

#### 4.3.4 循环结构 WHILE

> **WHILE语句创建一个带条件判断的循环过程。WHILE在执行语句执行时，先对指定的表达式进行判断，如**
>
> **果为真，就执行循环内的语句，否则退出循环。**

```sql
[while_label:] WHILE 循环条件 DO 
循环体 
END WHILE [while_label];
```

- **案例**

```sql
DELIMITER // 
CREATE PROCEDURE test_while() 
BEGIN
    DECLARE i INT DEFAULT 0; 
    WHILE i < 10 DO 
    SET i = i + 1; 
    END WHILE; 
    SELECT i; 
    END // 
DELIMITER ; 
#调用 
CALL test_while();
```

#### 4.3.5 循环结构 REPEAT

> **REPEAT语句创建一个带条件判断的循环过程。与WHILE循环不同的是，REPEAT 循环首先会执行一次循**
>
> **环，然后在 UNTIL 中进行表达式的判断，如果满足条件就退出，即 END REPEAT；如果条件不满足，则会**
>
> **就继续执行循环，直到满足退出条件为止**

```sql
[repeat_label:]REPEAT
		循环体的语句;
UNTIL 结束循环的条件表达式; 
END REPEAT [repeat_label]
```

- **案例**

```sql
DELIMITER // 
CREATE PROCEDURE test_repeat() 
BEGIN
    DECLARE i INT DEFAULT 0; 
    REPEATSET i = i + 1; 
    UNTIL i >= 10 
    END REPEAT; 
    SELECT i; 
END // 
DELIMITER ;
```

#### 4.3.6 循环结构 LEAVE

> **LEAVE语句：可以用在循环语句内，或者以 BEGIN 和 END 包裹起来的程序体内，表示跳出循环或者跳出**
>
> **程序体的操作。如果你有面向过程的编程语言的使用经验，你可以把 LEAVE 理解为 break**

```sql
DELIMITER // 
CREATE PROCEDURE leave_begin(IN num INT) 
begin_label: BEGIN 
	IF num<=0 
		THEN LEAVE begin_label; 
    ELSEIF num=1 
    	THEN SELECT AVG(salary) FROM employees; 
    ELSEIF num=2 
    	THEN SELECT MIN(salary) FROM employees; 
    ELSE
    	SELECT MAX(salary) FROM employees; 
    END IF; SELECT COUNT(*) FROM employees; END // DELIMITER ;
```

#### 4.3.7 循环结构 **ITERATE**

> **ITERATE语句：只能用在循环语句（LOOP、REPEAT和WHILE语句）内，表示重新开始循环，将执行顺序转到语句段开头处。如果你有面向过程的编程语言的使用经验，你可以把 ITERATE 理解为 continue，意思为“再次循环”。**



### 4.4 游标

> ​	游标，提供了一种灵活的操作方式，让我们能够对结果集中的每一条记录进行定位，并对指向的记录中的数据进行操作的数据结构。**游标让** **SQL** **这种面向集合的语言有了面向过程开发的能力。**
>
> ​	在 SQL 中，游标是一种临时的数据库对象，可以指向存储在数据库表中的数据行指针。这里游标充当了指针的作用,我们可以通过操作游标来对数据行进行操作。

#### 4.4.1 游标使用方式

> **游标必须在声明处理程序之前被声明，并且变量和条件还必须在声明游标或处理程序之前被声明。如果我们想要使用游标，一般需要经历四个步骤。不同的 DBMS 中，使用游标的语法可能略有不同。**

##### 4.4.1.1 声明游标

- **适用于 MySQL，SQL Server，DB2 和 MariaDB的声明方式**

```sql
DECLARE cursor_name CURSOR FOR select_statement;
```

- **适用于Oracle 或者 PostgreSQL**

```sql
DECLARE cursor_name CURSOR IS select_statement;
```

##### 4.4.1.2 打开游标

> **当我们定义好游标之后，如果想要使用游标，必须先打开游标。打开游标的时候 SELECT 语句的查询结果集就会送到游标工作区，为后面游标的逐条读取结果集中的记录做准备**。

```sql
OPEN cursor_name
```

##### 4.4.1.3 使用游标

> **这句的作用是使用 cursor_name 这个游标来读取当前行，并且将数据保存到 var_name 这个变量中，游标指针指到下一行。如果游标读取的数据行有多个列名，则在 INTO 关键字后面赋值给多个变量名即可**
>
> **==注意：var_name必须在声明游标之前就定义好。==**

```sql
FETCH cursor_name INTO var_name [, var_name] ...
```

```sql
FETCH cur_emp INTO emp_id, emp_sal ;
```

> **==游标的查询结果集中的字段数，必须跟 INTO 后面的变量数一致，否则，在存储过程执行的时候，MySQL 会提示错误==**

##### 4.4.1.4 关闭游标

> **有 OPEN 就会有 CLOSE，也就是打开和关闭游标。当我们使用完游标后需要关闭掉该游标。因为==游标会占用系统资源 ，如果不及时关闭，游标会一直保持到存储过程结束==，影响系统运行的效率。而关闭游标会释放系统资源**

```sql
CLOSE cursor_name
```

#### 4.4.2 案例

```sql
-- 创建存储过程“get_count_by_limit_total_salary()”，声明IN参数 limit_total_salary，DOUBLE类型；声明
-- OUT参数total_count，INT类型。函数的功能可以实现累加薪资最高的几个员工的薪资值，直到薪资总和
-- 达到limit_total_salary参数的值，返回累加的人数给total_count。

DELIMITER $
CREATE PROCEDURE get_count_by_limit_total_salary(IN limit_total_salary DOUBLE,OUT total_count INT)
BEGIN
DECLARE sum_salary DOUBLE DEFAULT 0.0;#记录总工资
DECLARE cursor_salary DOUBLE DEFAULT 0.0;#记录某个员工的工作
DECLARE emp_count INT DEFAULT 0;#记录循环个数
#定义游标
DECLARE emp_cursor CURSOR FOR SELECT salary FROM employees ORDER BY salary DESC;
#打开游标
OPEN emp_cursor;

REPEAT
	#使用游标（从游标中获取数据）
	FETCH emp_cursor INTO cursor_salary;
	
	SET sum_salary = sum_salary + cursor_salary;
	SET emp_count = emp_count + 1;
	
	UNTIL sum_salary >= limit_total_salary
END REPEAT;
	
	SET total_count = emp_count;
	CLOSE emp_cursor;
END $
DELIMITER ;

#调用
CALL get_count_by_limit_total_salary(200000,@total_count);
SELECT @total_count;
```

 ### 4.5  触发器

> **MySQL从 5.0.2 版本开始支持触发器。MySQL的触发器和存储过程一样，都是嵌入到MySQL服务器的一段程序**
>
> **触发器是由 事件来触发 某个操作，这些事件包括 `INSERT` 、 `UPDATE` 、 `DELETE` 事件。所谓事件就是指用户的动作或者触发某项行为。如果定义了触发程序，当数据库执行这些语句时候，就相当于事件发生了，就会自动激发触发器执行相应的操作**
>
> **当==对数据表中的数据执行插入、更新和删除操作，需要自动执行一些数据库逻辑时，可以使用触发器来实现==**

#### 4.5.1 触发器的使用

- **创建触发器**

```sql
-- 创建名称为before_insert的触发器，向test_trigger数据表插入数据之前，向test_trigger_log数据表中插入before_insert的日志信息。
DELIMITER // 
CREATE TRIGGER before_insert 
BEFORE INSERT ON test_trigger 
FOR EACH ROW 
BEGIN
	INSERT INTO test_trigger_log (t_log) 
	VALUES('before_insert'); 
END // 
DELIMITER ;

-- 向test_trigger数据表中插入数据
INSERT INTO test_trigger (t_note) VALUES ('测试 BEFORE INSERT 触发器');

-- 查看test_trigger_log数据表中的数据
SELECT * FROM test_trigger_log;
```



## 五、











