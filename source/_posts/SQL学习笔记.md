---
title: SQL学习笔记
date: 2017-06-04 20:21:23
comments: true
tags:
    - Sql
---

### 准备工作
这里我对下面需要用到的表进行初始化（表创建是以MySql为例）

      #学生信息表
      CREATE TABLE students (
        stu_no bigint(20) not null,
        stu_name varchar(65) not null,
        class_id int(11) not null,
        stu_email varchar(65),
        stu_address varchar(255),
        birthday date,
        primary key(stu_no)            
        );

        #学生成绩信息
        CREATE TABLE student_score(
          id bigint(20) not null,
          stu_no bigint(20) not null,
          math_score int(11),
          physics_score int(11),
          );

# 简单介绍
### 什么是数据库  
数据库是一个以某种有组织的方式存储数据集合的容器。通常我们会把我们使用的数据库软件简单的称之为数据库，这是不正确的；数据库的本质是一个容器，我们通过数据库管理软件(DBMS)创建和操纵容器。  

### 什么是表  
表是一种结构化的文件，用来存储特定类型的数据，例如：我们可以将客户的信息、产品的信息存储到不同的表。并且表的名字在一个数据库里是唯一的。

### 什么是列和数据
一张表由至少一个列组成，不同的列可以有不同的数据类型，但是一个列必然只能是一种数据类型，并且数据类型限制了这个列中可以存储的数据。

_应当尽量合理的选用数据类型，这样对我们存储和检索数据是有帮助的。_  

### 什么是行
类似我们使用的表格，每一列有自己的名称，那么每一行就是一条完整的数据。

### 什么是主键
主键是一列或者一组列，其值能够唯一的标识表中的每一行。(我们应当保证每一张表中都有自己的主键，这样可以便于我们管理我们的数据。)
_主键应该具有以下特征：任意两行都不具有相同的主键值；每一行都必须有一个非null的主键值；主键不允许修改或者更新；主键值不能重用。_

### 什么是SQL
SQL(Structured Query Language)结构化查询语言，是一门专门用来与数据库交互的语言。
_所有的DBMS都支持标准SQL，标准SQL是有ANSI标准委员会管理的，但是为了简化某些操作或者增加一个功能，不同的DBMS厂商又对标准SQL进行了一定的扩展。_

# 数据检索
### SELECT语句
每一个SQL语句都由一个或者多个关键字组成，而我们最常用的SQL语句应该就是SELECT语句了。  
例如：  

    SELECT stu_name FROM students;  

表示我要从学生表`students`中查询出所有学生的姓名`stu_name`。

_关键字是SQL的组成部分，例如查询语句里的`SELECT`、`FROM`，关键字作为保留字，不能作为表或者列的名字。_  

_一条正确的SQL语句必须要通过分号`;`分割。_

_SQL语句是不区分大小写的，但是很多开发人员喜欢将SQL的关键字大写，表名和列名小些，这样可以便于阅读和调试。_

上面的SQL中我们检索出了表中的一列，同样的我们也可以检索出一张表中的多列，多个列名中间使用逗号`,`间隔。
例如：  

    SELECT stu_name,stu_no,stu_class FROM students;

表示我从学生表里查询了所有学生的姓名`stu_name`、学号`stu_no`、班级`stu_class`。

我们还可以使用通配符`*`查询出一张表是所有的列，查询出的数据中，列的展示顺序通常是建表时定义的列的物理顺序。但是我们不推荐这样做，因为这样的查询效率很低(在一个项目中绝对不允许通过通配符去查询一张表！！！)。

### DISTINCT关键字
上面我们使用SELECT语句查询出了所有的列，但是这些数据里有一些重复的数据，例如一个班级里有两个名字叫做`liming`的同学，但是我只想知道班级里人的名字，并不需要知道有没有重复的出现，这里我们可以使用DISTINCT关键字。

    SELECT DISTINCT stu_name FROM students;

_注意使用DISTINCT关键字的时候，必须放在所有查询列名的前面。而且DISTINCT关键字是对跟在其后面的所有列去重，而不能对SELECT语句中指定的某一列。_

### 限制查询结果
我们上面的查询语句都是查询出一张表查询列里的所有的数据，但是我们只需要查询出一部分数据时，我们应该如何操作，这里不同的数据库的实现方式是不一样的。下面我们以不同的数据库为例，都查询学生表中的前五条数据中学生的名称。  

SQL Server：  

    SELECT TOP 5 stu_name FROM students;  
DB2：  

    SELECT stu_name FROM students FETCH FIRST 5 ROWS ONLY;  
Oracle：  

    SELECT stu_name FROM students WHERE ROWNUM <= 5;  
MySQL:  

    SELECT stu_name FROM students LIMIT 1 OFFSET 5;  

### 如何使用注释  
我们需要在SQL语句或者SQL文件中添加一些说明性的文字，这些文字并不能被DBMS执行，并且也不需要执行，我们将这些说明性质的文字称之为注释。

    SELECT stu_name    -- 这里是注释  
    FROM students;
 使用`--`表示其之后就是注释性的说明。

    #这里还是注释
    SELECT stu_name      
    FROM students;
使用`#`表示其之后就是注释性的说明。

    ／*SELECT stu_name,stu_no,stu_class
    FROM students;*／
    SELECT stu_name,stu_no,stu_class
    FROM students;
使用`/*`开始到`*/`结束表示中间的内用都是注释。

### 数据排序
如果没有进行明确的排序，那么检索出的数据顺序没有任何意义。(如果不排序，那么查询出来的数据的顺序是数据在底层表中展示出来的顺序，这有可能是最初添加到表中的顺序，但是如果对数据进行过更新或者删除，那么可能会对这个顺序有影响，所有如果没有排序，那么你查询出来的数据就是没有顺序的，不要进行任何假设！！！)

__SQL语句是由子句构成的，有些子句是必须的，有些则是可选的：例如查询时FROM子句就是必须的，如果没有FROM子句这个SQL就是不完整无法执行的，但是接下来我们要使用的排序ORDER BY子句就是可选的，我们在需要排序的时候才使用它，不使用它并不会造成我们的SQL无法执行__  

    SELECT stu_name FROM students ORDER BY stu_name;

上面的SQL表示我们从学生表查询学生的名字，并且按照名字中字母的顺序排列数据。

1. 我们可以对排序的方式进行指定，没有指定的情况下，默认按照升序排列:  

        SELECT stu_name FROM students ORDER BY stu_name ASC; --升序排列  
        SELECT stu_name FROM students ORDER BY stu_name DESC; --降序排列`

2. 我们还可以按照多个列排序

        SELECT stu_name FROM students ORDER BY stu_name ASC,stu_no DESC;

    表示先按照学生名字升序，再按照学号降序，并且参加排序的列并不一定要出现在SELECT子句中。
3. ORDER BY子句一定要是SELECT语句的最后一个子句，否则SQL语句会无法执行。

### 过滤数据
上面我们通过某些关键字或者子句可以限定我们查询表中的前几条数据，但是这很多情况下并不能满足要求，我们更多的是需要筛选出来符合某些条件的数据，这就需要用到WHERE子句了。
例如：

    SELECT stu_name FROM students WHERE stu_name = 'liming';

这里我们将查询出学生表中名字叫做`liming`的记录。  
1. 虽然SQL语句时大小写不敏感的，但是我们存储数据库中的数据时大小写敏感的，比如上面的语句如果数据库中我们存储的名字是`LIMING`，那么我们的查询语句就不会有任何结果！  
2. 如果存储的数据类型为字符串时我们需要将查询条件用`''`包裹起来，其他的数据类型则不同，例如：我要查询学号为1的记录：

    SELECT stu_name FROM students WHERE stu_no = 1;  

#### WHERE子句的操作符
|操作符    |说明		|操作符    |说明     |
|---------|:-------|---------|:-------|
|=        |等于     |>        |大于      |
|<>       |不等于   |>=        |大于等于       |
|!=       |不等于   |!>        |不大于       |
|<        |小于     |BETWEEN   |介于指定的两个值之间       |
|<=       |小于等于  |IS NULL  |该列为NULL值       |
|!<       |不小于   |        |       |

    SELECT stu_name,stu_no FROM students WHERE stu_no BETWEEN 1 AND 10;  

查询出学号介于1到10之间的学生左闭右开。

   SELECT stu_name,stu_no FROM students WHERE stu_email IS NULL;

查询出所有邮箱为NULL的学生数据。_NULL表示无值，这是同0或者空字符串不同的！_

我们想要查询出所有没有邮箱的学生时可能会使用这样的SQL：

    SELECT stu_name,stu_no FROM students WHERE stu_email = ‘’;  

但是这是不正确的，因为这样不会查询出邮箱字段为NULL的列，对于包含NULL的列，在查询的时候一个要格外的注意！

#### 多条件组合查询  
1. 我们查询数据时很可能是通过多个条件去筛选，例如我们需要查询出所有名字叫做`liming`并且邮箱字段不为NULL的数据时，我们可以在WHERE子句中通过AND操作符连接不同的查询条件。

        SELECT stu_name,stu_no FROM students WHERE stu_name = 'liming' AND stu_email IS NULL;

2. OR操作符与AND的操作的功能正好相反：  

        SELECT stu_name,stu_no FROM students WHERE stu_name = 'liming' OR stu_email IS NULL;

	这里我们将查询出名字是`liming`或者邮箱不为NULL的数据。

   _对于既存在AND操作符又存在OR操作符的WHERE子句，我们一定要使用圆括号明确的分组操作符，否则查询结果可能跟我们想要的不同！_

3. IN操作符
表示查询出符合条件的范围的所有结果。

        SELECT stu_name,stu_no FROM students WHERE stu_name IN ('liming','xiaohong') ;

  这条语句我们将查询出名字叫做`liming `和`xiaohong `的记录。

  _上面的语句我们也可以使用OR来实现，`SELECT stu_name,stu_no FROM students WHERE stu_name = 'liming' OR stu_name = 'xiaohong' ;`，但是IN操作符通常情况下比OR执行的更快！_

4. NOT操作符
WHERE子句中的NOT用来否定其后的关键字。例如：

        SELECT stu_name,stu_no FROM students WHERE NOT stu_name = 'liming';

  表示查询出名字不叫作`liming`的所有记录。

        SELECT stu_name,stu_no FROM students WHERE stu_name NOT IN ('liming','xiaohong');

  表示查询出名字不叫作`liming`和`xiaohong`的记录。

  _MariaDB中只支持NOT否定IN、BETWEEN、EXISTS子句！_

5. LIKE关键字  
前面所有的操作符的条件都是已值的值，如果我们只是知道一个值的一部分，想要检索出来所有可能的值的时候就需要用到LIKE关键字，例如：我要检索出来所有名字以`li`开始的学生时：  

        SELECT stu_name FROM students WHERE stu_name LIKE 'li%';  

 1. 这里`%`是一种通配符，表示检索出所有名字以`li`开始的学生；
 2. 如果我们确定我们要检索值的前一部分使用`li%`，后一部分使用`%ing`，中间一部分使用`%in%`；  
 3. `%`通配符可以表示多个字符，`_`通配符表示一个字符，DB2不支持`_`，Microsoft Access需要使用`?`而不是`_`  

   如果不是必须，那么不要使用通配符；如果必须使用，那么请将通配符检索放在所有检索条件的末尾！！！

# 字段的计算

有时候单纯的查询结果并不能满足我们的要求，我们可能需要将两个字符串字段的值进行拼接，例如：学生名字+学生的家庭地址；可能需要对数值字段进行简单的运算，例如：计算学生的总成绩。

### 拼接字段

DB2、Oracle:  

    SELECT stu_name || ':' || stu_address AS stus FROM students;

MySQL:  

    SELECT CONCAT(stu_name, ':', stu_address) AS stus FROM students;

这里DB2和Oracle中可以在两个需要展示的字段中使用`||`表示拼接，而MySQL中需要使用CONCAT函数拼接两个字段；  
两个字段拼接完成后展示的结果中没有对应的名字，这里我们通过`AS`关键字将这一列数据赋予别名`stus`，方便我们使用，这里`AS`关键字是可选的，我们也可以直接在拼接列的后面加空格后使用别名，也是可以的，但是推荐使用`AS`关键字。

### 执行算数运算

    SELECT stu_no，math_score+physics_score FROM student_score;

这里我们将学生的数学成绩和物理成绩进行相加，计算两门课程的总分。    
SQL支持基本算数操作符：

|操作符    |说明	  |
|---------|:-------|
|+        |加      |
|-        |减      |
|*        |乘      |
|/        |除      |


# 函数
虽然所有的DBMS都支持函数，但是每个DBMS都有特定的函数，这同基本的SQL语句是不同的！

### 文本处理函数

|函数                          |说明	  |
|-----------------------------|:-------|
|LEFT()                       |返回字符串左边的字符     |
|LENGTH()                     |返回字符串的长度        |
|LOWER()                      |将字符串转换为小写      |
|LTRIM()（ACCESS使用LCASE()）  |去掉字符串左边的空格      |
|RIGHT()                      |返回字符串右边的字符      |
|RTRIM()                      |去掉字符串右边的空格      |
|UPPER() （ACCESS使用UCASE()） |将字符串转换为大写      |
|TRIM()                       |去掉字符串两边的空格      |

### 日期和时间处理函数

每种DBMS对日期和时间的处理都有其特殊的形式，便于快速有效的排序和过滤，以及节省存储空间。这导致的每种DBMS的对于时间处理都是有差异的，针对同样的条件不同的DBMS我们需要写不同的函数来筛选。

下面以查询出生在1992年的学生名单为例说明每种DBMS的不同：  
SQL Server：

    SELECT stu_name FROM students WHERE DATEPART(yy,birthday) = 1992;

Access：

    SELECT stu_name FROM students WHERE DATEPART('yyyy',birthday) = 1992;

PostgreSQL：

    SELECT stu_name FROM students WHERE DATE_PART(year,birthday) = 1992;

Oracle：

    SELECT stu_name FROM students WHERE to_number(to_char(birthday, 'yyyy')) = 1992;

MySQL、MariaDB:

	SELECT stu_name FROM students WHERE YEAR(birthday) = 1992;


### 数值处理函数

|函数                         |说明	  |
|----------------------------|:-------|
|ABS()                       |返回一个数的绝对值     |
|COS()                       |返回一个角度的余弦        |
|EXP()                       |返回一个数的指数值      |
|PI()                        |返回圆周率      |
|SIN()                       |返回一个角度的正弦      |
|SQRT()                      |返回一个数的平方根      |
|TAN()                       |返回一个角度的正切      |


### 聚集函数

实际生活中我们可能需要需要汇总数据，比如我需要确定表中的行数；表中某一列的和、最大值、最小值或者平均值。这样我们需要通过聚集函数来得到我们需要的结果。

|函数                         |说明	  |
|----------------------------|:-------|
|AVG()                       |返回某列的平均值     |
|COUNT()                       |返回表中记录的行数        |
|MAX()                       |返回某列的最大值      |
|MIN()                        |返回某列的最小值      |
|SUM()                       |返回某列的和      |

1. AVG()函数对表中的列计数后求和，根据总和和列数求的该列的平均值。   
	计算一个所有学生的数学分数的平均值：

	    SELECT AVG(math_score) AS avg_math FROM student_score;

	也可以通过WHERE子句的约束，计算某个班级的数学平均分：  

	    SELECT AVG(math_score) AS avg_math FROM student_score WHERE class_id=1;  

 	_AVG()函数忽略值为NULL的列！_  

2. COUNT()函数对统计表中行的数目。

	    SELECT COUNT(*) AS count FROM student_score;

	我们使用COUNT(*)可以对表中的行进行统计，无论表中的列是NULL或者非NULL值；但是假如我们要统计参加数学考试的人数，缺考的学生的分数值为NULL，那么我们就需要使用下面的SQL：  

		SELECT COUNT(math_score) AS count FROM student_score;
 	_使用COUNT(column)可以对表中相应的列进行统计，并且忽略该行的NULL值。_

3. MAX()返回指定列中的最大值。
	计算出所有学生中的数学最高分：

	 	SELECT MAX(math_score) AS count FROM student_score;

	_MAX()函数忽略值为NULL的列。_
	_对于文本列，MAX()函数返回该列排序后的最后一行。_

4. MIN()返回指定列中的最大值。
	计算出所有学生中的数学最低分：

	 	SELECT MIN(math_score) AS count FROM student_score;

	_MIN()函数忽略值为NULL的列。_
	_对于文本列，MAX()函数返回该列排序后的第一行。_

5. SUM()函数对指定列的值进行求和。  
	计算所有学生的数学总分：

		 SELECT SUM(math_score) AS avg_math FROM student_score;

	  _SUM()函数忽略值为NULL的列。_

6. 以上5个聚集函数都可以结合DISTINCT使用，这里使用DISTINCT同之前是相同的效果，可以去除相同的值。
	_虽然对MIN()和MAX()使用DISTINCT并不会报错，但是并没有任何意义。_  
	_对于COUNT()函数必须使用具体的列的时候才可以使用DISTINCT，对于COUNT(*)无法使用DISTINCT。_


# 分组数据

为了对数据进行分组后进行汇总统计，比如分别计算各个班级的平均成绩，这里我们主要使用SELECT语句的两个子句：GROUP BY子句和HAVING子句。

### 数据的分组

之前我们只能查询一张表里全部数据的，对数据进行汇总计算，如果我们要计算一部分数据的汇总结果，只能通过WHERE条件去限定，这样每次查询都只能汇总一部分，这里通过分组汇总可以一次将所有分组的汇总结果查询出来，比如说查询每个班级的数学平均成绩：

	SELECT class_id, AVG(math_score) AS avg_math FROM student_score GROUP BY class_id;

_1.GROUP BY 子句中可以包含任意数目的列，通过多列我们可以对数据进行更细致的分组。_  
_2.GROUP BY 子句中包含的列都必须是检索列或者有效的表达式，但是不能是聚集函数。_   
_3.大多数SQL实现中不允许GROUP BY 子句中列的数据类型是长度可变的比如文本字段。_  
_4.如果使用GROUP BY 子句那么所有SELECT子句中出现的非聚集函数计算的列都必须在GROUP BY子句中。_ __有些DBMS可以通过配置文件的设置改变这一情况，例如MySQL的SqlMode，但是并不推荐这样做！__  
_5.如果GROUP BY 子句中的列包含NULL值，那么NULL将作为一个分组返回。_
_5.GROUP BY 子句必须出现在WHERE子句之后，ORDER BY子句之前。_   

### 分组数据过滤
GROUP BY子句能够对数据进行分组统计，对统计后的结果在此进行筛选时，则需要使用HAVING子句。例如筛选出平均成句高于80分的班级：

	SELECT class_id, AVG(math_score) AS avg_math FROM student_score GROUP BY class_id HAVING AVG(math_score)>80;

_HAVING和WHERE子句都是对数据进行过滤的，但是WHERE子句用在分组之前，HAVING子句用在分组之后。_

HAVING子句和WHERE子句是可以同时使用的，例如，我要查询出除去1班后，剩余班级中平均成绩高于80的班级：

	SELECT class_id, AVG(math_score) AS avg_math FROM student_score WHERE class_id<>1 GROUP BY class_id HAVING AVG(math_score)>80;

### SQL语句中子句的顺序

|子句            |说明	            |是否必须使用|
|---------------|:-----------------|----------|
|SELECT         |要返回的列或者表达式 |是         |
|FROM           |检索数据的表        |仅在需要从表中查询数据时使用|
|WHERE          |分组之前数据过滤    |否         |
|GROUP by       |分组数据           |仅在需要分组数据时使用     |
|HAVING         |分组之后数据过滤     |否        |
|ORDER BY       |数据按照相应的列排序  |否	      |

# 子查询

### 在WHERE字句中使用子查询
上面我们的查询都是只使用了一张表的数据，但是大多数情况下，我们需要展示的数据可能存在于多张表中，例如我们要查询出一个学生的成绩，但是名字信息在`students`表中，而成绩信息存储在`student_score`表中，两张表中可以通过`stu_no`学号字段关联。  

使用简单查询时，我们需要使用两条SQL先检索出这个学生对应的学号：

	SELECT stu_no, stu_name FROM students WHERE stu_name = 'liming';

假设这里查询出来的`stu_no`为1，那么我们继续从第二张表检索出liming的数学成绩：

	SELECT stu_no, math_score FROM student_score WHERE stu_no=1;
这里我们将第一个查询转变为子查询，我们通过一个SQL就能完成我们的查询工作：

 	SELECT stu_no, math_score FROM student_score WHERE stu_no = (SELECT stu_no FROM students WHERE stu_name = 'liming');

### 在SELECT字句中使用子查询  
我们也可以在`SELECT`字句中使用子查询，将子查询作为结果展示。例如我们需要查询每个学生的总成绩，查询结果要求展示学生的名字。

  SELECT stu_no, stu_name (SELECT math_score+physics_score FROM student_score WHERE  students.stu_no = student_score.stu_no) AS scores FROM students;

# 表联结
我们可以在查询数据库的时候使用`join`关键字对表进行关联查询，在使用这个关键字之间需要简单介绍一下什么是关系表。

### 什么是关系表  
就如刚开始的时候建立的两张表，我们建立学生成绩信息的表`student_score`的时候，不能将所有查询成绩可能用到的并且已经在学生信息表`students`中存在的字段全部建立一遍，这样会造成数据的冗余，不符合关系型数据库表结构设计的规范，所以我们通过 `stu_no`字段对两张表进行关联，那么`students`和 `student_score`就可以称之为关系表。  

### 联结查询  
上面我们将学生的基本信息存储到`students`表中，将学生的成绩信息存储在`student_score`表中，那么我们要在检索学生成绩的同时能够展示出学生的基本信息时，我们需要使用联结查询。

  SELECT students.stu_name, students.class_id, students.stu_email, students.stu_address, student_score.math_score, student_score.physics_score FROM students, student_score WHERE students.stu_no = student_score.stu_no;

上面的查询中，由于需要查询的字段属于不同的两张表，这里我们使用完全限定名来指定需要查询的列属于哪张表，但是我们不必这么做，因为完全限定名通常情况下会比较长，比较繁琐，我们可以使用别名的形式查询：
  SELECT a.stu_name, a.class_id, a.stu_email, a.stu_address, b.math_score, b.physics_score FROM students AS a, student_score AS b WHERE a.stu_no = b.stu_no;

这里我们需要注意我们的WHERE子句，这里我们同样使用WHERE字句来过滤我们的查询结果，对于联结查询，也就是规定两张表依照什么进行关联，如果没有WHERE字句，或者WHERE字句的过滤条件不争取，查询将会产生笛卡尔积。绝大部分情况下笛卡尔积不是我们想要的结果。

### 内联结
上面的联结查询基于两张表的某个字段值的相等，这种联结查询也称之为等值联结，也成为内联结。对于这种联结方式，我们可以采用另外一种等值的方式实现：
  SELECT a.stu_name, a.class_id, a.stu_email, a.stu_address, b.math_score, b.physics_score FROM student_score b INNNER JOIN students a ON a.stu_no = b.stu_no;

_两种查询方式的结果实际是一致的，使用哪种看个人的习惯，我习惯使用`INNER JOIN`的方式进行内联结查询。_
_两张表可以进行关联查询，同样的对于多于两张表，我们也可以进行关联查询，唯一需要注意的就是我们进行表关联的条件是否正确，并且我们需要注意，关联的表越多，则查询效率越低。_

### 自联结
假如我们需要找到`liming`的所有同班同学，我们可以通过子查询:
  SELECT stu_no, stu_name, class_id FROM students WHERE  class_id = (SELECT class_id FROM students WHERE stu_name = 'liming');

同样我们也可以将`students`表关联两次，从而得到我们的结果：
  SELECT a.stu_no, a.stu_name, a.class_id FROM students a INNER JOIN students b ON a.class_id = b.class_id AND b.stu_name = 'liming';

这样将一张表和它自己进行联结查询的方式，称之为自联结。

_通常情况下对于上面的情况，推荐使用自联结，因为DBMS处理自联结比子查询的效率要高，但是具体问题，具体分析。_

### 外联结
使用内联结查询的时候，我们只能将`students`和`student_score`表中同时存在的数据检索出来，如果某位学生转学过来，还没有参加过考试，那么`student_score`表中没有这位同学的记录，这里要想将这位同学的记录展示出来，则需要使用外联结。
  SELECT a.stu_name, a.class_id, a.stu_email, a.stu_address, b.math_score, b.physics_score FROM student_score b RIGHT JOIN students a ON a.stu_no = b.stu_no;

  SELECT a.stu_name, a.class_id, a.stu_email, a.stu_address, b.math_score, b.physics_score FROM students a LEFT JOIN student_score b ON a.stu_no = b.stu_no;

上面两条sql都能达到我们的要求，这两条sql有什么区别呢，对于关联查询，我们通常将FROM字句后紧跟的表称之为主表，JOIN关键字后紧跟的表称之为子表，第一条sql，使用 `RIGHT JOIN` 表示通过关联条件子表存在但是主表不存在的结果会展示出来；第一条sql，使用 `LEFT JOIN` 表示通过关联条件主表存在但是子表不存在的结果会展示出来。

_虽然在JOIN关键字的前面没有使用RIGHT或者LEFT则默认为内联结，但是通常情况下，内联结时请明确指定，使用 INNER JOIN。_

__未完待续。。。__
