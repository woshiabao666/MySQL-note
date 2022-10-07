# 一、 索引

## 1.1 索引及其优缺点

> MySQL官方对索引的定义为：**==索引（Index）是帮助MySQL高效获取数据的数据结构==**。

### 1.1.1 优点

> - **提高数据检索的效率，降低数据库的IO成本**
> - **通过创建唯一索引，==可以保证数据库表中每一行数据的唯一性==**
> - **在实现数据的参考完整性方面，可以 加速表和表之间的连接 。换句话说，对于有依赖关系的子表和父表联合查询时，可以提高查询速度。**
> - **在使用分组和排序子句进行数据查询时，可以显著 减少查询中分组和排序的时 间 ，降低了CPU的消耗。**

### 1.1.2 缺点

> - **创建索引和维护索引要耗费时间 ，并且随着数据量的增加，所耗费的时间也会增加**
> - **索引需要占 磁盘空间 ，除了数据表占数据空间之外，每一个索引还要占一定的物理空间**
> - **虽然索引大大提高了查询速度，同时却会降低更新表的速度 。==当对表中的数据进行增加、删除和修改的时候，索引也要动态地维护，这样就降低了数据的维护速度。==**

## 1.2 InnoDB的索引

### 1.2.1 索引设计方案

![](E:\阶段性资料\笔记\pic\QQ截图20220828213248.png)

![](E:\阶段性资料\笔记\pic\QQ截图20220828215246.png)

- **==record_type==：记录头信息的一项属性，表示记录的类型， 0 表示普通记录、 2 表示最小记录、 3 表示最大记录、1 表示存放目录项记录**
- **==next_record== ：记录头信息的一项属性，表示下一条地址相对于本条记录的地址偏移量，我们用箭头来表明下一条记录是谁**

- **各个列的值 ：这里只记录在 index_demo 表中的三个列，分别是 c1 、 c2 和 c3 。**
- **其他信息 ：除了上述3种信息以外的所有信息，包括其他隐藏列的值以及记录的额外信息**

### 1.2.2 InnoDB的索引方案

- **迭代一次**

![](E:\阶段性资料\笔记\pic\QQ截图20220828215634.png)

> **从图中可以看出来，我们新分配了一个编号为30的页来专门存储目录项记录。这里再次强调目录项记录和普通的用户记录的不同点：**
>
> - **==目录项记录的 record_type 值是1，而 普通用户记录 的 record_type 值是0==**
> - **==目录项记录只有主键值和页的编号两个列==，而==普通的用户记录的列是用户自己定义的，可能包含很多列==，另外还有InnoDB自己添加的隐藏列。**
> - **两者用的是一样的数据页，都会为主键值生成 Page Directory （页目录），从而在按照主键值进行查找时可以使用 二分法 来加快查询速度**

- **迭代三次**

![](E:\阶段性资料\笔记\pic\QQ截图20220828220434.png)

### 1.2.3 B+树

> **一个B+树的节点其实可以分成好多层，规定最下边的那层，也就是存放我们用户记录的那层为第 0 层，之后依次往上加。所以一般情况下，我们 用到的B+树都不会超过4层 ，那我们通过主键值去查找某条记录最多只需要做4个页面内的查找（查找3个目录项页和一个用户记录页），又因为在每个页面内有所谓的 Page Directory （页目录），所以在页面内也可以通过 二分法 实现快速定位记录。**



## 1.3 常见索引概念

### 1.3.1 聚簇索引

**1.特点**:**使用记录主键值的大小进行记录和页的排序**

> - **页内 的记录是按照主键的大小顺序排成一个 单向链表**
> - **各个存放 用户记录的页 也是根据页中用户记录的主键大小顺序排成一个 双向链表 。**
> - **存放目录项记录的页 分为不同的层次，在同一层次中的页也是根据页中目录项记录的主键大小顺序排成一个双向链表** 。

**==2.B+树的叶子节点存储的是完整的用户记录==**

**`优点`**

> - **数据访问更快 ，因为聚簇索引将索引和数据保存在同一个B+树中，因此从聚簇索引中获取数据比非聚簇索引更快**
> - **聚簇索引对于主键的 排序查找 和 范围查找 速度非常快**
> - **按照聚簇索引排列顺序，查询显示一定范围数据的时候，由于数据都是紧密相连，数据库不用从多个数据块中提取数据，所以 节省了大量的io操作**

**`缺点`**

>- **插入速度严重依赖于插入顺序 ，按照主键的顺序插入是最快的方式，否则将会出现页分裂，严重影响性能。因此，对于InnoDB表，==我们一般都会定义一个自增的ID列为主键==**
>- **更新主键的代价很高 ，因为将会导致被更新的行移动。因此，==对于InnoDB表，我们一般定义主键为不可更新==**
>- **二级索引访问需要两次索引查找 ，第一次找到主键值，第二次根据主键值找到行数据**

### 1.3.2 二级索引（辅助索引、非聚簇索引）

![](E:\阶段性资料\笔记\pic\QQ截图20220901184130.png)

> - **==非聚簇索引叶子节点存储的不是完整的用户记录，而是 `当前列信息`  +  `主键`  这两个列的值==**
> - **==目录项记录中不再是`主键 + 页号`的搭配，而变成了`当前列信息 + 页号`这两个列的值==**

**==回表：==**

> **我们根据这个以c2列大小排序的B+树只能确定我们要查找记录的主键值，所以如果我们想根据c2列的值查找到完整的用户记录的话，仍然需要到 聚簇索引中再查一遍，这个过程称为回表 。也就是根据c2列的值查询一条完整的用户记录需要使用到 2 棵B+树！**



### 1.3.3 联合索引

![](E:\阶段性资料\笔记\pic\QQ截图20220901190441.png)

- **==每条目录项都是由`c2,c3,页号`组成个条记录都按照`c2`的值进行排序，如果`c2`值相同再按照`c3`的值排序==**
- **==B+树的叶子节点处的用户记录由`c2,c3和主键c1列`组成==**

> **我们也可以同时以多个列的大小作为排序规则，也就是同时为多个列建立索引，比方说我们想让B+树按照 c2和c3列 的大小进行排序，这个包含两层含义：**
>
> - **==先把各个记录和页按照c2列进行排序。==**
> - **==在记录的c2列相同的情况下，采用c3列进行排序==**
>
> **注意一点，以c2和c3列的大小为排序规则建立的B+树称为 联合索引 ，本质上也是一个二级索引。它的意思与分别为c2和c3列分别建立索引的表述是不同的，不同点如下：**
>
> - **建立 联合索引 只会建立如上图一样的1棵B+树。**
> - **为c2和c3列分别建立索引会分别以c2和c3列的大小为排序规则建立2棵B+树。**

### 1.3.4 InnoDB的B+树索引的注意事项

- **`根页面位置万年不动`**
- **`内节点中目录项记录的唯一性`**

> **==我们需要把主键值添加到二级索引内节点中的目录项记录，这样就能保证B+树每一层节点中各条`目录项记录`这个字段是唯一的==**

![](E:\阶段性资料\笔记\pic\QQ截图20220901192144.png)

- **`一个页面最少存储2条记录`** 



## 1.4 MyISAM的索引方案

> **MyISAM引擎使用B+树作为索引结构，==叶子节点的data域存放的`数据记录地址`，MyISAM索引方案虽然也是树形结构，但是却将`索引与数据分开存储`==**
>
> - **将表中的记录==`按照插入顺序单独存储在文件中`==，称之为`数据文件`。在插入数据的时候不会刻意按照主键大小排序，所以不能使用二分法查找**
> - **使用MyISAM存储引擎的表会把索引信息存储到另一个`索引文件`中，`MyISAM`会单独为表的主键创建一个索引，但是存储内容是==主键值 + 数据记录地址==组合**

![](E:\阶段性资料\笔记\pic\QQ截图20220901204132.png)

## 1.5 MyISAM与InnoDB对比

**MyISAM的索引方式都是“非聚簇”的，与InnoDB包含一个聚簇索引不同**

> - **在InnoDB存储引擎中，我们只需要根据主键值对 聚簇索引进行一次查找就能找到对应的记录，而在==MyISAM中却需要进行一次 回表 操作，意味着MyISAM中建立的索引相当于全部都是 二级索引==。**
> - **==InnoDB的数据文件本身就是索引文件，而MyISAM索引文件和数据文件是分离的 ，索引文件仅保存数据记录的地址==**
> -  **InnoDB的非聚簇索引data域存储相应记录 主键的值 ，而MyISAM索引记录的是 地址 。换句话说，InnoDB的所有非聚簇索引都引用主键作为data域**
> - **MyISAM的回表操作是十分 快速 的，因为是拿着地址偏移量直接到文件中取数据的，反观InnoDB是通过获取主键之后再去聚簇索引里找记录，虽然说也不慢，但还是比不上直接用地址去访问**
> - **==InnoDB要求表 必须有主键 （ MyISAM可以没有 ）。如果没有显式指定，则MySQL系统会自动选择一个可以非空且唯一标识数据记录的列作为主键。如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整型==**

# 二、InnoDB数据存储结构

## 2.1 数据库的存储结构：页

> **索引信息以及数据记录都是存储在页结构中，索引是在存储引擎上实现的，MySQL服务器的`存储引擎`负责对表中的数据进行读取与写入工作，不同的存储引擎`存放格式`一般是不同的**

### 2.1.1 磁盘与内存交互基本单位：页

> - **InnoDB将出书划分为若干个页，==InnoDB中页的大小默认为16K==**
>
> - **以`页`为`基本单位`，也就是说一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB刷新到磁盘中。==在数据库中，不论读一行还是读多行，都是将这些所在的页进行加载，数据库管理存储空间的基本单位是页（page），数据库I/O的最小单位是页==**
> - **==记录是按照行来存储，读取数据库是以页为单位，否则读取一次效率会很低==**

![](E:\阶段性资料\笔记\pic\QQ截图20220903161906.png)

### 2.1.2 页结构概述

> **页之间通过`双向链表`相关联即可，每个数据页中的记录会按照主键值从小到大组成一个`单向链表`，每个数据也都会为存储在它里面的记录生成一个`页目录`，在通过主键查找某条记录的时候可以在也目录中`使用二分法`快速定位到对应的槽，然后再遍历该槽对应的分组中的记录即可快速找到指定的记录**

### 2.2.2 页的上层结构

![](E:\阶段性资料\笔记\pic\QQ截图20220903163601.png)

- **`区（Extent）`：比页大一级，在InnoDB中，一个区会分配==64个连续的页==，因为InnoDB中页的默认大小是16KB，所以一个区的大小是64 * 16 KB = 1 MB**
- **`段（Segment）`：由一个或多个区组成，区在文件系统是一个连续分配的空间（InnoDB中是连续的64个页），==段是数据库中的分配单位，不同类型的数据库对象以不同的段形式存在==**
- **`表空间（Tablespace）`是一个逻辑容器，表空间存储的对象是段，在一个表空间中可以有一个或多个段，但是一个段只能属于一个表空间。数据库由一个或多个表空间组成，表空间划分为`系统表空间` `用户表空间`  `撤销表空间`等**

## 2.2 页的内部结构

# 三、索引的创建与设计原则

## 3.1 索引的分类

> **MySQL的索引包括普通索引、唯一性索引、全文索引、单列索引、多列索引和空间索引等**
>
> - **从`功能逻辑`上说，索引主要有 4 种，分别是==普通索引、唯一索引、主键索引、全文索引==**
> - **按照`物理实现`方式 ，索引可以分为 2 种：==聚簇索引和非聚簇索引==**
> - **按照 `作用字段`个数进行划分，分成==单列索引和联合索引==**

### 3.1.1 普通索引

> **创建普通索引，不附加任何限制条件，单纯用于提高查询效率，这类索引可以创建在任何数据类型当中**

### 3.1.2 唯一性索引

> **使用`UNIQUE`参数设置索引为唯一性索引，限制该字段的值必须是唯一的，==但允许有空值，在一张数据表里允许有多个唯一索引==**

 ### 3.1.3 主键索引

> **主键索引就是一种==特殊的唯一性索引，增加了不为空的约束，也就是NOT NULL + UNIQUE，一张表里最多有一个主键索引==**

### 3.1.4 单列索引

> **在表中单个字段上创建索引，单列索引只根据该字段进行索引。单列索引可以是普通索引，也可以是唯一性索引，还可以是全文索引，==只要保证该索引对应一个字段即可==，一个表可以有多个单列索引**

### 3.1.5 多列索引

> **多列索引实在表的`多个字段组合`上创建的一个索引，该索引指向创价时对应的多个字段，可以通过这几个字段进行查询，但是==只有查询条件中使用了这些字段中的第一个字段才会被使用==，使用组合索引时遵循`最左前缀原则`**

### 3.1.6 全文索引  

![](E:\阶段性资料\笔记\pic\QQ截图20220912095516.png)

### 3.1.7 空间索引

![](E:\阶段性资料\笔记\pic\QQ截图20220912100156.png)

### 3.1.8 小结

> **InnoDB** **：支持 B-tree、Full-text 等索引，不支持 Hash索引**
>
> **MyISAM ：支持 B-tree、Full-text 等索引，不支持 Hash 索引**
>
> **Memory** ：**支持 B-tree、Hash 等索引，不支持 Full-text 索引**

## 3.2 索引的创建与删除

### 3.2.1 创建表的时候创建索引

```sql
CREATE TABLE dept( dept_id INT PRIMARY KEY AUTO_INCREMENT, dept_name VARCHAR(20) );

CREATE TABLE emp (
	emp_id INT PRIMARY KEY AUTO_INCREMENT,
	emp_name VARCHAR ( 20 ) UNIQUE,
	dept_id INT,
CONSTRAINT emp_dept_id_fk FOREIGN KEY ( dept_id ) REFERENCES dept ( dept_id ) 
); 
```

![](E:\阶段性资料\笔记\pic\QQ截图20220912110519.png)



**如果显式创建表时创建索引的话，基本语法格式如下：**

```sql
CREATE TABLE table_name [col_name data_type] 
[UNIQUE | FULLTEXT | SPATIAL] [INDEX | KEY] [index_name] (col_name [length]) [ASC | DESC]
```



- **`UNIQUE` 、 `FULLTEXT` 和 `SPATIAL` 为可选参数，分别表示唯一索引、全文索引和空间索引**
- **`INDEX` 与 `KEY` 为同义词，两者的作用相同，用来指定创建索引**
- **`index_name` 指定索引的名称，为可选参数，如果不指定，那么MySQL默认`col_name`为索引名**
- **`col_name` 为需要创建索引的字段列，该列必须从数据表中定义的多个列中选择**
- **`length` 为可选参数，表示索引的长度，只有字符串类型的字段才能指定索引长度**
- **`ASC` 或 `DESC` 指定升序或者降序的索引值存储**



1.**创建普通索引**

```sql
CREATE TABLE book (
	book_id INT,
	book_name VARCHAR ( 100 ),
	AUTHORS VARCHAR ( 100 ),
	info VARCHAR ( 100 ),
	COMMENT VARCHAR ( 100 ),
	year_publication YEAR,
	INDEX ( year_publication ) 
);
```

2.**创建唯一索引**

```sql
# 声明有唯一索引的字段，在添加数据时，要保证唯一性，但是可以添加 NULL 值
CREATE TABLE test1 ( 
    id INT NOT NULL, 
    NAME VARCHAR ( 30 ) NOT NULL, 
    UNIQUE INDEX uk_idx_id ( id ) 
);
```

3.**主键索引**

```sql
CREATE TABLE student ( 
    id INT ( 10 ) UNSIGNED AUTO_INCREMENT, 
    student_no VARCHAR ( 200 ), 
    student_name VARCHAR ( 200 ), 
    PRIMARY KEY ( id ) 
);

# 修改主键索引：必须先删除掉(drop)原索引，再新建(add)索引
# 删除主键索引
ALTER TABLE student drop PRIMARY KEY ; 
```

4.**创建单列索引**

```sql
CREATE TABLE test2 ( 
    id INT NOT NULL, 
    NAME CHAR ( 50 ) NULL, 
    INDEX single_idx_name ( NAME ( 20 )) 
);
```

5.**创建组合索引**

```sql
CREATE TABLE test3 (
	id INT ( 11 ) NOT NULL,
	NAME CHAR ( 30 ) NOT NULL,
	age INT ( 11 ) NOT NULL,
	info VARCHAR ( 255 ),
	INDEX multi_idx ( id, NAME, age ) 
);
```

6. **创建全文索引**

```sql
CREATE TABLE test4 (
	id INT NOT NULL,
	NAME CHAR ( 30 ) NOT NULL,
	age INT NOT NULL,
	info VARCHAR ( 255 ),
	FULLTEXT INDEX futxt_idx_info ( info (50)) 
) ENGINE = MyISAM;

# 在MySQL5.7及之后版本中可以不指定最后的ENGINE了，因为在此版本中InnoDB支持全文索引
CREATE TABLE articles ( 
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY, 
    title VARCHAR (200), body TEXT, 
    FULLTEXT index (title, body) 
) ENGINE = INNODB ;

CREATE TABLE `papers` ( 
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT, 
    `title` varchar(200) DEFAULT NULL, 
    `content` text, PRIMARY KEY (`id`), 
    FULLTEXT KEY `title` (`title`,`content`) 
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

# 全文索引用match+against方式查询
SELECT * FROM papers WHERE MATCH(title,content) AGAINST (‘查询字符串’);

```

> 1. 使用全文索引前，搞清楚版本支持情况；
> 2. 全文索引比 like + % 快 N 倍，但是可能存在精度问题；
> 3. 如果需要全文索引的是大量数据，建议先添加数据，再创建索引。



7.创建空间索引

> 空间索引创建中，要求空间类型的字段必须为`非空`

```sql
CREATE TABLE test5( 
    geo GEOMETRY NOT NULL, 
    SPATIAL INDEX spa_idx_geo(geo) 
) ENGINE=MyISAM;
```



### 3.2.2 已经存在的表创建索引

> **在已经存在的表中创建索引可以使用==ALTER TABLE==语句或者==CREATE INDEX语句==**



**1.** **使用==ALTER TABLE==语句创建索引 ==ALTER TABLE==语句创建索引的基本语法如下：**

```sql
ALTER TABLE table_name ADD [UNIQUE | FULLTEXT | SPATIAL] [INDEX | KEY] 
[index_name] (col_name[length],...) [ASC | DESC]
```



2.**==CREATE INDEX==语句可以在已经存在的表上添加索引，在MySQL中，==CREATE INDEX==被映射到一个==ALTER TABLE==语句上，基本语法结构为：**

```sql
CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name 
ON table_name (col_name[length],...) [ASC | DESC]
```



```sql
# 创建表
create table book5(
    book_id INT,
    book_name varchar(100),
    authors varchar(100),
    info varchar(100),
    comment varchar(100),
    year_publication year
);

# 使用ALTER TABLE语句创建索引
ALTER TABLE book5 ADD INDEX idx_cmt ( COMMENT );
ALTER TABLE book5 ADD UNIQUE uk_idx_bname ( book_name );

# 使用CREATE INDEX添加索引 
create unique index uk_idx_info on book5(info);
create index mul_bid_bname on book5(book_id,book_name);
```

### 3.2.3 删除索引

**1.使用==ALTER TABLE==删除索引 ==ALTER TABLE==删除索引的基本语法格式如下：**

```sql
ALTER TABLE table_name DROP INDEX index_name;
```

**2.使用==DROP INDEX==语句删除索引==DROP INDEX==删除索引的基本语法格式如下：**

```sql
DROP INDEX index_name ON table_name;
```

> **注意：** **添加==AOTU_INCREMENT==约束字段的==唯一索引==不能被删除**



## 3.3 MySQL8.0索引新特性

### 3.3.1 支持降序索引（仅限InnoDB）

> **==MySQL8.0版本前仍然是升序索引，使用时进行反向扫描，降低了数据库的效率==，某些情况下我们需要使用降序索引。例如：如果一个查询需要对多个列进行排序，且顺序要求不一致，那么使用降序索引将会避免数据库使用额外的文件排序操作**

```sql
CREATE TABLE ts1(
    a int,
    b int,
    index idx_a_b(a,b desc)
);
```



### 3.3.2 隐藏索引

> 在MySQL 5.7版本及之前，只能通过显式的方式删除索引。此时，如果发现删除索引后出现错误，又只能通过显式创建索引的方式将删除的索引创建回来。如果数据表中的数据量非常大，或者数据表本身比较大，这种操作就会消耗系统过多的资源。
>
> 从MySQL 8.x开始支持 `隐藏索引（invisible indexes` ，`只需要将待删除的索引设置为隐藏索引，使查询优化器不再使用这个索引（即使使用force index（强制使用索引），优化器也不会使用该索引），确认将索引设置为隐藏索引后系统不受任何响应，就可以彻底删除索引`。 ==这种通过先将索引设置为隐藏索引，再删除索引的方式就是软删除== 
>
> **同时，如果像验证某个索引删除之后==查询性能的影响==，就可以先隐藏该索引**

- **==注意：主键不能被设置为隐藏索引，当表中没有显式主键时，表中第一个唯一非空索引会成为隐式主键，也不能设置为隐藏索引==**



1.**创建表时直接创建** 在MySQL中创建隐藏索引通过SQL语句`INVISIBLE`来实现，其语法形式如下：

```sql
CREATE TABLE tablename( 
    propname1 type1[CONSTRAINT1], 
    propname2 type2[CONSTRAINT2], 
    ……
    propnamen typen, 
    INDEX [indexname](propname1 [(length)]) INVISIBLE 
);

# 上述语句比普通索引多了一个关键字INVISIBLE，用来标记索引为不可见索引
```



2.**在已经存在的表上创建**

```sql
CREATE INDEX indexname ON tablename(propname[(length)]) INVISIBLE;
```



3.**通过ALTER TABLE语句创建**

```sql
ALTER TABLE tablename ADD INDEX indexname (propname [(length)]) INVISIBLE;
```



4.**切换索引可见状态** 已存在的索引可通过如下语句切换可见状态

```sql
ALTER TABLE tablename ALTER INDEX index_name INVISIBLE; #切换成隐藏索引 
ALTER TABLE tablename ALTER INDEX index_name VISIBLE; #切换成非隐藏索引
```

> **注意: ==当索引被隐藏时，它的内容仍然是和正常索引一样实时更新的。如果一个索引需要长期被隐藏，那么可以将其删除，因为索引的存在会影响插入、更新和删除的性能==**



## 3.4 索引的设计原则

### 3.4.1 哪些情况适合创建索引

**1.==字段的数值有唯一性的限制==**

> ​	**索引本身可以起到约束的作用，比如唯一性索引，主键索引，如果数据表中`某个字段是唯一的`。那么就可以创建`唯一性索引`，或者`主键索引`。**



**2.==频繁作为WHERE查询条件的字段==**

> ​	**某个字段在SELECT语句的 WHERE 条件中经常被使用到，那么就需要给这个字段创建索引了。尤其是在数据量大的情况下，创建普通索引就可以大幅提升数据查询的效率。**

```sql
select course_id,class_id,create_time from student_info where student_id = '169240' # 0.276s

create INDEX idx_stu_id on student_info(student_id) # 添加索引后

select course_id,class_id,create_time from student_info where student_id = '14277' # 0.002s
```



**3.==经常GROUP BY 和 ORDER BY 的列==**

> ​	**索引就是让数据按照某种顺序进行存储或检索，因此当我们使用 GROUP BY 对数据进行分组查询，或者使用 ORDER BY 对数据进行排序的时候，就需要对==分组或者排序的字段进行索引== 。如果待排序的列有多个，那么可以在这些列上建立==组合索引==**

```sql
-- 实例1
select student_id,count(*) as num from student_info group by student_id limit 100 #0.001s

alter table student_info drop index idx_stu_id; #删除索引后

select student_id,count(*) as num from student_info group by student_id limit 100 #0.587s

-- 实例2
select student_id,count(*) FROM student_info group by student_id order by create_time desc #0.833s

alter table student_info add index idx_stu_id_cre_time(student_id,create_time desc) #组合索引，降序索引

select student_id,count(*) FROM student_info group by student_id order by create_time desc #0.349s

```

**4.==UPDATE、DELETE的WHERE条件列==**

> ​	**对数据按照某个条件进行查询后再进行 UPDATE 或 DELETE 的操作，如果对 WHERE 字段创建了索引，就能大幅提升效率。原理是因为我们需要先根据 WHERE 条件列检索出来这条记录，然后再对它进行更新或删除。==如果进行更新的时候，更新的字段是非索引字段，提升的效率会更明显，这是因为非索引字段更新不需要对索引进行维护==**

```sql
update student_info set student_id = '10221' where name = 'qwer' # 0.499s

create index idx_name on student_info (name);

update student_info set student_id = '10221' where name = 'qwer' # 0.001s
```

**5.==DISTINCT字段需要创建索引==**

> ​	**有时候我们需要对某个字段进行去重，使用 DISTINCT，那么对这个字段创建索引，也会提升查询效率**

```sql
SELECT DISTINCT student_id FROM `student_info`; #0.511s

create index idx_stu_id on student_info(student_id)

SELECT DISTINCT student_id FROM `student_info`; #0.357ss
```

**6.==多表JOIN连接操作时，创建索引注意事项==**

> - **首先， ==连接表的数量尽量不要超过 3 张，因为每增加一张表就相当于增加了一次嵌套的循环==，数量级增长会非常快，严重影响查询的效率。**
> - **其次， ==对 WHERE 条件创建索引== ，因为 WHERE 才是对数据条件的过滤。如果在数据量非常大的情况下，没有 WHERE 条件过滤是非常可怕的。**
> - **最后， 对用于连接的字段创建索引 ，并且该字段在多张表中的类型必须一致 。比如 course_id 在 student_info 表和 course 表中都为 int(11) 类型，而不能一个为 int 另一个为 varchar 类型。**

**7.==使用列的类型小的创建索引==**

> ​	**这里的==类型大小==指的是该类型表示的数据范围的大小，数据类型越小，在查询时进行的比较操作越快，`索引占用的存储空间越少`，`一个数据也就可以放下更多的记录，从而减少磁盘I/O带来的性能损耗`，`可以把更多的数据页缓存在内存中，加快读写效率`**

**8.==使用字符串前缀创建索引==**

>  	**假设一个字符串很长，当我们为这个字符串建立索引时，我们可以截取字段前面一部分内容建立索引，这个就叫做==前缀索引==，既节约了时间又减少了字符串的比较时间**
>
> ​	**但是，截取多少字段成为了一个问题，截取得多了，达不到节省索引存储空间的目的；截取得少了，重复内容太多，字段的散列度(选择性)会降低；以下是计算`字段在全部数据中的选择度`：**
>
> ```sql
> count(distinct left(列名, 索引长度))/count(*)
> # 例如：
> select count(distinct left(address,10)) / count(*) as sub10, -- 截取前10个字符的选择度 
> count(distinct left(address,15)) / count(*) as sub11, -- 截取前15个字符的选择度 
> count(distinct left(address,20)) / count(*) as sub12, -- 截取前20个字符的选择度 
> count(distinct left(address,25)) / count(*) as sub13 -- 截取前25个字符的选择度
> from shop;
> ```
>
> **通过不同长度去计算，与全表的选择性对比**,==一般长度为20的索引，区分度会高达90%以上==
>
> 
>
> ​	**拓展：Alibaba《Java开发手册》**
>
> - **【 `强制` 】在 varchar 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本**
>   **区分度决定索引长度。**

```sql
create table shop(address varchar(120) not null); 

alter table shop add index(address(12));
```

**9.==区分度高(散列性高)的列适合作为索引==**

> ​	**`列的基数`是指某一列中不重复数据的个数，在记录行数一定的情况下，列的基数越大，该列中的值越分散，所以我们需要为列基数大的列建立索引。可以使用公式**:
>
> ```sql
>  select count (distinct a)/count(*) from temp1
> ```
>
> ==越接近1越好，一般超过33%就是比较高效的索引了==

**10.==使用最频繁的列放到联合索引的左侧==**

> 

**11.==在多个字段都要创建索引的情况下，联合索引优于单值索引==**

> 

### 3.4.2 限制索引的数目

> ​	**`建议单张表索引数目不要超过6个`**
>
> - **每个索引都要占用磁盘空间，索引越多，需要的磁盘空间越大**
> - **索引会影响 INSERT DELETE UPDATE 等语句的性能，表中数据更改的同时，索引也会调整和更新，会造成负担**



### 3.4.3 不适合创建索引的情况

**1.在where中使用不到的字段，不要设置索引**

**2.数据量小的表最好不要使用索引**

**3.有大量重复数据的列上不要建立索引**

> **当数据重复度大于10%的时候，不需要对这个字段使用索引**

**4.避免对经常更新的表创建过多的索引**

**5.不建议用无序的值作为索引**

> **例如身份证，UUID，MD5等无序长字符串，在索引比较的时候需要转为ASCII，并且插入时可能造成页分裂**

**6.删除不再使用或者很少使用的索引**

**7.不要定义冗余或重复的索引**



# 四、性能分析工具的使用

## 4.1 数据库服务器优化步骤

> ​	**当遇到数据库调优问题，整个流程划分成了`观察（show status）`和`行动（action）`**



![](E:\阶段性资料\笔记\pic\QQ截图20220913102900.png)



**小结：**

![](E:\阶段性资料\笔记\pic\QQ截图20220913112456.png)





## 4.2 查看系统参数

> 在MySQL中，可以使用 `SHOW STATUS` 语句查询一些MySQL数据库服务器的性能参数，执行频率；`SHOW STATUS`语句语法如下：
>
> ```sql
> SHOW [GLOBAL|SESSION] STATUS LIKE '参数';
> ```

**常用的性能参数如下：**

**• `Connections`：连接MySQL服务器的次数。** 

**• `Uptime`：MySQL服务器的上线时间。** 

**• `Slow_queries`：慢查询的次数。** 

**• `Innodb_rows_read`：Select查询返回的行数** 

**• `Innodb_rows_inserted`：执行INSERT操作插入的行数** 

**• `Innodb_rows_updated`：执行UPDATE操作更新的行数** 

**• `Innodb_rows_deleted`：执行DELETE操作删除的行数** 

**• `Com_select`：查询操作的次数。** 

**• `Com_insert`：插入操作的次数。对于批量插入的 INSERT 操作，只累加一次。** 

**• `Com_update`：更新操作的次数。** 

**• `Com_delete`：删除操作的次数。**



## 4.3 统计SQL的查询成本：last_query_cost

> ​	**一条sql语句在执行前需要确定查询执行计划，如果存在多种执行计划的话，MySQL会计算每个执行计划所需要的成本，从中选择`成本最小`的作为最终执行的计划**
>
> ​	**如果我们想要查看某条SQL语句的查询成本，可以在执行完这条SQL语句之后，通过查看当前会话中的`last_query_cost`变量值来得到当前查询的成本，这个查询成本对应的是==SQL语句所需要读取的页的数量==**

```sql
SELECT student_id, class_id, NAME, create_time FROM student_info WHERE id = 900000; #0.002s
SHOW STATUS LIKE 'last_query_cost%';
```

|  Variable_name  |   value    |
| :-------------: | :--------: |
| Last_query_cost | `1.000000` |

```sql
SELECT student_id, class_id, NAME, create_time FROM student_info WHERE id BETWEEN 900001 AND 900100; #0.003s
SHOW STATUS LIKE 'last_query_cost%';
```

|  Variable_name  |    value    |
| :-------------: | :---------: |
| Last_query_cost | `20.290751` |

- **页的数量是刚才的 `20` 倍，但是查询的效率并没有明显的变化，实际上这两个 SQL 查询的时间基本上一样，就是因为采用了顺序读取的方式将页面一次性加载到缓冲池中，然后再进行查找。虽然 `页 数量（last_query_cost）增加了不少` ，但是通过缓冲池的机制，并 `没有增加多少查询时间`**

> **SQL查询是一个动态的过程，从页的加载角度来看，可以得到以下两点结论：**
>
> ​	**1.==位置决定效率==，如果页就在数据库缓存中，那么效率是最高的**
>
> ​	**2.==批量决定效率==,如果从磁盘中对单一页进行读取效率是很低的（10ms左右），而采取顺序读取，批量对页进行读取，平均每页的读取效率会高很多甚		至高于单个页面的随机读取**



## 4.4 定位慢查询日志

> ​	**MySQL慢查询日志用来记录`响应时间超过阈值`的语句，运行时间超过`long_query_time`值得SQL会被记录到慢查询日志中，`long_query_time`默认值为`10`，意思是运行10秒以上（不含10秒）的语句。**
>
> ​	**默认情况下MySQL没有开启`慢查询日志`，需要我们手动设置这个参数，开启慢查询日志会对性能带来一定的影响**



### 4.4.1 开启慢查询日志参数

**1.开启`slow_query_log`**

```sql
SHOW VARIABLES LIKE 'slow_query_log%';
```

|     Variable_name     |            value             |
| :-------------------: | :--------------------------: |
|   `slow_query_log`    |             `ON`             |
| `slow_query_log_file` | `D:\mysql\Date\MSI-slow.log` |



**2.修改long_query_time阈值**

```sql
show variables like '%long_query_time%';
```

|   Variable_name   |    value    |
| :---------------: | :---------: |
| `long_query_time` | `10.000000` |



```sql
set long_query_time=1;
show variables like '%long_query_time%';

set global long_query_time = 1; #全局
show global variables like '%long_query_time%';
```

|   Variable_name   |    value    |
| :---------------: | :---------: |
| `long_query_time` | `10.000000` |

### 4.4.2 查看慢查询数目 

```sql
SHOW GLOBAL STATUS LIKE '%Slow_queries%';
```

| Variable_name  |    value    |
| :------------: | :---------: |
| `Slow_queries` | `10.000000` |

- 找到慢查询日志

```txt
D:\mysql\mysql-8.0.19-winx64\bin\mysqld, Version: 8.0.19 (MySQL Community Server - GPL). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
D:\mysql\mysql-8.0.19-winx64\bin\mysqld, Version: 8.0.19 (MySQL Community Server - GPL). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
# Time: 2022-09-14T23:10:39.352280Z
# User@Host: root[root] @ localhost [::1]  Id:     9
# Query_time: 1.959146  Lock_time: 0.000061 Rows_sent: 2000000  Rows_examined: 2000000
use atguigudb_2;
SET timestamp=1663197037;
select * from student;
```

## 4.5 查看SQL执行成本：SHOW PROFILE

> ​	**`SHOW PROFILE`是MySQL提供可以分析当前会话SQL都做了什么，执行的资源消耗情况，可用于sql调优的测量。`默认情况下处于关闭状态`，并保存最近15次的运行结果。**

```sql
show variables like 'profiling'; # 查看状态
set profiling = 'ON'; #开启
```

| Variable_name | value |
| :-----------: | :---: |
|  `profiling`  | `ON`  |



- **查看sql执行成本**

```sql
SHOW PROFILES；
```

| QUERY_ID | Duration   | QUERY                                      |
| -------- | ---------- | ------------------------------------------ |
| 1        | 0.0013375  | show variables like 'profiling'            |
| 2        | 0.000092   | show profiling                             |
| 3        | 0.46251175 | select * from student where stuno = 568552 |

### 4.5.1 SHOW PROFILE 常用查询参数

- **还可以指定query id的开销`show profile for query 2`,在`SHOW PROFILE`中还能查看不同部分的开销**

```sql
show profile cpu,block io for query 3;
```

![](E:\阶段性资料\笔记\pic\QQ截图20220915081409.png)

> **① ALL：显示所有的开销信息。** 
>
> **② BLOCK IO：显示块IO开销。** 
>
> **③ CONTEXT SWITCHES：上下文切换开销。** 
>
> **④ CPU：显示CPU开销信息。** 
>
> **⑤ IPC：显示发送和接收开销信息。** 
>
> **⑥ MEMORY：显示内存开销信息。** 
>
> **⑦ PAGE FAULTS：显示页面错误开销信息。** 
>
> **⑧ SOURCE：显示和Source_function，Source_file， Source_line相关的开销信息。** 
>
> **⑨ SWAPS：显示交换次数开销信息。**



## 4.6 分析查询工具：EXPLAIN

### 4.6.1 基本语法

```sql
EXPLAIN SELECT select_options 
#或者
DESCRIBE SELECT select_options
```

| 列名            | 描述                                                      |
| --------------- | --------------------------------------------------------- |
| `id`            | 在一个大的查询语句中每个SELECT关键字都对应一个 `唯一的id` |
| `select_type`   | SELECT关键字对应的那个查询的类型                          |
| `table`         | 表名                                                      |
| `possible_keys` | 可能用到的索引                                            |
| `key`           | 实际上使用的索引                                          |
| `key_len`       | 实际使用到的索引长度                                      |
| `ref`           | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息    |
| `rows`          | 预估的需要读取的记录条数                                  |
| `filtered`      | 某个表经过搜索条件过滤后剩余记录条数的百分比              |
| `Extra`         | 一些额外的信息                                            |



### 4.6.2 EXPLAIN 各列的作用

**1.`table`**

> ​	**不论我们的查询语句有多复杂，里边儿 包含了多少个表 ，到最后也是需要对每个表进行 单表访问 的，所以MySQL规定EXPLAIN语句输出的每条记录都对应着某个单表的访问方法，该条记录的table列代表着该表的表名（有时不是真实的表名字，可能是简称）。**



**2.`id`**

> - **id如果相同，可以认为是一组，从上往下顺序执行**
> - **在所有组中，id值越大，优先级越高，越先执行**
> - **关注点：id号每个号码，表示一趟独立的查询, 一个sql的查询趟数越少越好**



**3.`select_type`**

> ​	**一条复杂的查询语句里面可以包含多个`SELECT`关键字，`每个SELECT关键字代表着一个小的查询语句`，每个`SELECT`关键字的`FROM`子句中可以包含若干张表，`每一张表都对应着执行计划输出中的一条记录`，对于同一条`SELECT`关键字中的表来说，它们的id值是相同的**
>
> ​	**MYSQL为每一个SELECT关键字代表的小查询定义了==select_type==属性，我们可以通过某个小查询==select_type==属性就可以知道`这个小查询在整个SQL中扮演了什么角色`**



- `UNION`或者`UNION ALL`

```sql
EXPLAIN SELECT * FROM s1 UNION SELECT * FROM s2;
```

| id   | select_type | table          | partitions | type | possible_key | key    | key_len | ref    | rows   | filtered | Extra           |
| ---- | ----------- | -------------- | ---------- | ---- | ------------ | ------ | ------- | ------ | ------ | -------- | --------------- |
| 1    | `PRIMARY`   | `s1`           | (NULL)     | ALL  | (NULL)       | (NULL) | (NULL)  | (NULL) | 9895   | 100.00   | (NULL)          |
| 2    | `UNION`     | `s2`           | (NULL)     | ALL  | (NULL)       | (NULL) | (NULL)  | (NULL) | 9895   | 100.00   | (NULL)          |
| 3    | (NULL)      | `UNION RESULT` | <union1,2> | ALL  | (NULL)       | (NULL) | (NULL)  | (NULL) | (NULL) | (NULL)   | Using temporary |



- **`子查询`**：如果子查询不能转化为对应的`semi-join`的形式，并且子查询不是相关子查询，那么该子查询第一个`SELECT`关键字代表的哪个查询的`SELECT_TYPE`就是`SUBQUERY`

> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2) OR key3 = 'a';
> ```
>
> | id   | select_type | table | partitions | type  | possible_key | key      | key_len | ref    | rows | filtered | Extra       |
> | ---- | ----------- | ----- | ---------- | ----- | ------------ | -------- | ------- | ------ | ---- | -------- | ----------- |
> | 1    | `PRIMARY`   | `s1`  | (NULL)     | ALL   | idx_key3     | (NULL)   | (NULL)  | (NULL) | 9895 | 100.00   | Using where |
> | 2    | `SUBQUERY`  | `s2`  | (NULL)     | index | idx_key1     | idx_key1 | 303     | (NULL) | 9895 | 100.00   | Using index |



> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2 WHERE s1.key2 = s2.key2) OR key3 = 'a'; #相关子查询
> ```
>
> | id   | select_type          | table | partitions | type   | possible_key      | key      | key_len | ref         | rows | filtered | Extra       |
> | ---- | -------------------- | ----- | ---------- | ------ | ----------------- | -------- | ------- | ----------- | ---- | -------- | ----------- |
> | 1    | `PRIMARY`            | `s1`  | (NULL)     | ALL    | idx_key3          | (NULL)   | (NULL)  | (NULL)      | 9895 | 100.00   | Using where |
> | 2    | `DEPENDENT SUBQUERY` | `s2`  | (NULL)     | eq_ref | idx_key2,idx_key1 | idx_key2 | 5       | db1.s1.key2 | 1    | 10.00    | Using where |



> > **相关子查询**
>
> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2 WHERE key1 = 'a' UNION SELECT key1 FROM s1 WHERE key1='b');
> ```
>
> | id   | select_type          | table        | partitions | type | possible_key | key      | key_len | ref    | rows   | filtered | Extra                   |
> | ---- | -------------------- | ------------ | ---------- | ---- | ------------ | -------- | ------- | ------ | ------ | -------- | ----------------------- |
> | 1    | `PRIMARY`            | `s1`         | (NULL)     | ALL  | (NULL)       | (NULL)   | (NULL)  | (NULL) | 9895   | 100.00   | Using where             |
> | 2    | `DEPENDENT SUBQUERY` | `s2`         | (NULL)     | ref  | idx_key1     | idx_key1 | 303     | const  | 1      | 100.00   | Using where,Using index |
> | 3    | `DEPENDENT UNION`    | `s1`         | (NULL)     | ref  | idx_key1     | idx_ket1 | 303     | const  | 1      | 100.00   | Using where,Using index |
> | 4    | `UNION RESULT`       | `<union2,3>` | (NULL)     | ALL  | (NULL)       | (NULL)   | (NULL)  | (NULL) | (NULL) | (NULL)   | Using temporary         |



> > **派生表子查询，`DERIVED`**
>
> ```sql
> EXPLAIN SELECT * FROM (SELECT key1, count(*) as c FROM s1 GROUP BY key1) AS derived_s1 where c > 1;
> ```
>
> | id   | select_type | table        | partitions | type  | possible_key | key      | key_len | ref    | rows | filtered | Extra       |
> | ---- | ----------- | ------------ | ---------- | ----- | ------------ | -------- | ------- | ------ | ---- | -------- | ----------- |
> | 1    | `PRIMARY`   | `<derived2>` | (NULL)     | ALL   | idx_key3     | (NULL)   | (NULL)  | (NULL) | 9895 | 33.33    | Using where |
> | 2    | `DERIVED`   | `s1`         | (NULL)     | index | idx_key1     | idx_key1 | (NULL)  | (NULL) | 9895 | 100.00   | Using index |



> > **当查询优化器执行包含子查询的语句时，选择将子查询物化之后与外层查询进行连接查询时，该子查询对应的`select_type`属性就是`MATERIALIZED`**
>
> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 IN (SELECT key1 FROM s2); #子查询被转化为了物化表
> ```
>
> | id   | select_type    | table         | partitions | type   | possible_key        | key                 | key_len | ref         | rows | filtered | Extra       |
> | ---- | -------------- | ------------- | ---------- | ------ | ------------------- | ------------------- | ------- | ----------- | ---- | -------- | ----------- |
> | 1    | `SIMPLE`       | `s1`          | (NULL)     | ALL    | idx_key1            | (NULL)              | (NULL)  | (NULL)      | 9895 | 100.00   | Using where |
> | 2    | `SIMPLE`       | `<subquery2>` | (NULL)     | eq_ref | <auto_distinct_key> | <auto_distinct_key> | 303     | db1.s1.key2 | 1    | 100.00   | (Null)      |
> | 3    | `MATERIALIZED` | `s2`          | (NULL)     | index  | idx_key1            | idx_key1            | 303     | (NULL)      | 9895 | 100.00   | Using index |



4.`partitions` (可略)



5.`type ☆`

> ​	**执行计划的一条记录就代表MySQL对某个表的`执行查询时的访问方法`，又称为`访问类型`**
>
> ​	**完整的访问方法如下： `system` ， `const` ， `eq_ref` ， `ref` ， `fulltext `， `ref_or_null` ， `index_merge `，` unique_subquery` ， `index_subquery` ， `range` ， `index `，` ALL`** 



> **==1.system==:当表中`只有一条记录`并且该表使用的存储引擎的统计数据是精确的，比如`MyISAM，Memory`，那么该表的访问方法就是`system`**
>
> **==2.const==:当根据主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问方法就是`const`**
>
> ```sql
> explain select * from temp1 where id = 0 #id是主键
> ```
>
> **==3.eq_ref==：连接查询时，如果`被驱动表`是通过`主键或者唯一二级索引列等值匹配的方式进行访问`（`如果该主键或者唯一二级索引是联合索引的话，所有的索引列都必须进行等值比较`），那么该被驱动表的访问方法就是`eq_ref`**
>
> ```sql
> EXPLAIN SELECT * FROM s1 INNER JOIN s2 ON s1.id = s2.id;
> ```
>
> | id   | select_type | table | partitions | type     | possible_key | key     | key_len | ref         | rows | filtered | Extra  |
> | ---- | ----------- | ----- | ---------- | -------- | ------------ | ------- | ------- | ----------- | ---- | -------- | ------ |
> | 1    | `SIMPLE`    | `s1`  | (NULL)     | ALL      | (Null)       | (NULL)  | (NULL)  | (NULL)      | 9895 | 100.00   | (Null) |
> | 2    | `SIMPLE`    | `s1`  | (NULL)     | `eq_ref` | PRIMARY      | PRIMARY | 4       | db1.s1.key2 | 1    | 100.00   | (Null) |
>
> **==4.ref,ref_or_null==:当通过`普通的二级索引列与常量进行等值匹配时来查询`某个表，那么对该表的访问访问就是`ref`或者`ref_or_null`**
>
> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 = 'a';
> EXPLAIN SELECT * FROM s1 WHERE key1 = 'a' or key1 IS NULL;
> ```
>
> **==5.index_merge==:单表访问方法时，在某些场景下可以使用`Intersection`、`Union`、`Sort-Union`这三种索引合并的方式来执行查询**
>
> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 = 'a' OR key3 = 'a';
> ```
>
> **==6.unique_subquery==:是针对一些包含`IN`的子查询的查询语句中，如果查询优化器决定将`IN`子查询转换为`EXISTS`子查询，而且子查询可以使用到`主键`进行等值匹配的话，那么该子查询执行计划的`type`就是`unique_subquery`**
>
> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key2 IN (SELECT id FROM s2 where s1.key1 = s2.key1) OR key3 = 'a';
> ```
>
> | id   | select_type          | table | partitions | type              | possible_key     | key     | key_len | ref    | rows | filtered | Extra       |
> | ---- | -------------------- | ----- | ---------- | ----------------- | ---------------- | ------- | ------- | ------ | ---- | -------- | ----------- |
> | 1    | `PRIMARY`            | `s1`  | (NULL)     | ALL               | idx_key3         | (NULL)  | (NULL)  | (NULL) | 9895 | 100.00   | Using where |
> | 2    | `DEPENDENT SUBQUERY` | `s2`  | (NULL)     | `unique_subquery` | PRIMARY,idx_key1 | PRIMARY | 4       | func   | 1    | 10.00    | Using where |
>
> **==7.range==:如果使用索引获取某些`范围区间`的记录，那么就可能是`range`**
>
> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 IN ('a', 'b', 'c');
> EXPLAIN SELECT * FROM s1 WHERE key1 > 'a' AND key1 < 'b';
> ```
>
> **==8.index==:当我们使用`索引覆盖`，但是需要扫描全部的索引记录时，该表的访问方法就是`index`**
>
> ```sql
> EXPLAIN SELECT key_part2 FROM s1 WHERE key_part3 = 'a';
> ```
>
> **==9.All==:全表扫描**

**`小结`：结果值最好到最坏依次顺序**

**`system` > `const` > `er_ref` > `ref` > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > `range` > `index` > `All`**

**SQL性能优化的目标：至少达到`range`级别**

 

**6.`possible_keys和key`**

> **`possible_keys`列表示在某个查询语句中，对某个表执行`单表查询时可能用到的索引`，但不一定被查询使用；`key`列表示`实际用到的索引`有哪些，如果为NULL，则没有使用索引**



**7.`key_len`**

> **实际使用到的索引长度（字节数）是否充分利用上了索引，`值越大越好，主要针对于联合索引有参考意义`**



**8.`ref`**

> **`当索引列等值查询时，与索引进行等值匹配的对象信息`**

```sql
EXPLAIN SELECT * FROM s1 WHERE key1 = 'a';
```

| id   | select_type | table | partitions | type | possible_key | key    | key_len | ref     | rows | filtered | Extra  |
| ---- | ----------- | ----- | ---------- | ---- | ------------ | ------ | ------- | ------- | ---- | -------- | ------ |
| 1    | `SIMPLE`    | `s1`  | (NULL)     | ref  | idx_key1     | (NULL) | 303     | `const` | 1    | 100.00   | (Null) |

```sql
EXPLAIN SELECT * FROM s1 INNER JOIN s2 ON s1.id = s2.id;
```

| id   | select_type | table | partitions | type     | possible_key | key     | key_len | ref               | rows | filtered | Extra  |
| ---- | ----------- | ----- | ---------- | -------- | ------------ | ------- | ------- | ----------------- | ---- | -------- | ------ |
| 1    | `SIMPLE`    | `s1`  | (NULL)     | ALL      | PRIMARY      | (NULL)  | (NULL)  | (NULL)            | 9895 | 100.00   | (Null) |
| 2    | `SIMPLE`    | `s2`  | (NULL)     | `eq_ref` | PRIMARY      | PRIMARY | 4       | atguigudb_2.s1.id | 1    | 100.00   | (Null) |

   

**9.`rows`**

> **`预估的需要读取的记录条数，值越小越好`**



**10.`filtered`**

> - **`某个表经过搜索条件后过滤剩余记录条数的百分比；`**



> - **`如果使用的时索引执行的单表扫描，那么计算时需要估计出满足除使用到对应索引的搜索条件外的其他搜索条件`**
>
> ```sql
> EXPLAIN SELECT * FROM s1 WHERE key1 > 'z' AND common_field = 'a';
> ```
>
> | id   | select_type | table | partitions | type  | possible_key | key      | key_len | ref      | rows | filtered | Extra                              |
> | ---- | ----------- | ----- | ---------- | ----- | ------------ | -------- | ------- | -------- | ---- | -------- | ---------------------------------- |
> | 1    | `SIMPLE`    | `s1`  | (NULL)     | range | idx_key1     | idx_key1 | 303     | `(Null)` | 382  | `10.00`  | Using index condition; Using where |



> - **对于单表查询来说，这个`filtered`列的值没有什么意义，我们`更关注在连接查询钟驱动表对应的执行计划记录的filtered值`，他决定了被驱动表要执行的次数`（rows * filtered）`**
>
> ```sql
> EXPLAIN SELECT * FROM s1 INNER JOIN s2 ON s1.key1 = s2.key1 WHERE s1.common_field = 'a';
> ```
>
> | id   | select_type | table | partitions | type  | possible_key | key      | key_len | ref                 | rows | filtered | Extra       |
> | ---- | ----------- | ----- | ---------- | ----- | ------------ | -------- | ------- | ------------------- | ---- | -------- | ----------- |
> | 1    | `SIMPLE`    | `s1`  | (NULL)     | ALL   | idx_key1     | (NULL)   | (NULL)  | (NULL)              | 9895 | 10.00    | Using where |
> | 2    | `SIMPLE`    | `s2`  | (NULL)     | `ref` | idx_key1     | idx_key1 | 303     | atguigudb_2.s1.key1 | 1    | 100.00   | (Null)      |

**11.`Extra`**

> **额外信息**
>
> ```sql
> EXPLAIN SELECT 1; # No table used
> 
> EXPLAIN SELECT * FROM s1 WHERE 1 != 1; # impossible where
> 
> EXPLAIN SELECT * FROM s1 WHERE common_field = 'a'; # Using where
> 
> EXPLAIN SELECT MIN(key1) FROM s1 WHERE key1 = 'abcdefg'; # No matching min/max row
> 
> EXPLAIN SELECT key1 FROM s1 WHERE key1 = 'a'; # Using index
> 
> EXPLAIN SELECT * FROM s1 WHERE key1 > 'z' AND key1 LIKE '%b'; # Using index condition
> 
> EXPLAIN SELECT * FROM s1 INNER JOIN s2 ON s1.common_field = s2.common_field; # Using join buffer (Block Nested Loop)
>  
> EXPLAIN SELECT * FROM s1 LEFT JOIN s2 ON s1.key1 = s2.key1 WHERE s2.id IS NULL; # Not exists
> 
> EXPLAIN SELECT * FROM s1 WHERE key1 = 'a' OR key3 = 'a'; # Using intersect(...) 、 Using union(...) 和 Using sort_union(...)
> 
> EXPLAIN SELECT * FROM s1 LIMIT 0; # Zero limit
>  
> EXPLAIN SELECT * FROM s1 ORDER BY key1 LIMIT 10; # Using filesort
> 
> EXPLAIN SELECT DISTINCT common_field FROM s1; # Using temporary
> ```

**小结：**

- **EXPLAIN不考虑各种Cache**
- **EXPLAIN不能显示MySQL在执行查询时所作的优化工作**
- **EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况**
- **部分统计信息是估算的，并非精确值**

### 4.6.3 EXPLAIN的进一步使用

1.**EXPLAIN四种输出格式**

> **这里谈谈EXPLAIN的输出格式。EXPLAIN可以输出四种格式： `传统格式` ， `JSON格式` ， `TREE格式` 以及 `可视化输出` 。用户可以根据需要选择适用于自己的格式。**

- **传统格式**



- **JSON格式**

> > **JSON格式是四种格式里面`输出信息最详尽`的格式，里面包含了执行成本信息**
>
> ```sql
>  # EXPLAIN FORMAT=JSON SELECT ....
>  
> EXPLAIN format = json SELECT
> * 
> FROM
> 	s1
> 	INNER JOIN s2 ON s1.key1 = s2.key2 
> WHERE
> 	s1.common_field = 'a'
> 
> ```
>
> ```json
> {
>   "query_block": {
>     "select_id": 1,
>     "cost_info": {
>       "query_cost": "1360.07"
>     },
>     "nested_loop": [
>       {
>         "table": {
>           "table_name": "s1",
>           "access_type": "ALL",
>           "possible_keys": [
>             "idx_key1"
>           ],
>           "rows_examined_per_scan": 9895,
>           "rows_produced_per_join": 989,
>           "filtered": "10.00",
>           "cost_info": {
>             "read_cost": "914.80",
>             "eval_cost": "98.95",
>             "prefix_cost": "1013.75",
>             "data_read_per_join": "1M"
>           },
>           "used_columns": [
>             "id",
>             "key1",
>             "key2",
>             "key3",
>             "key_part1",
>             "key_part2",
>             "key_part3",
>             "common_field"
>           ],
>           "attached_condition": "((`atguigudb_2`.`s1`.`common_field` = 'a') and (`atguigudb_2`.`s1`.`key1` is not null))"
>         }
>       },
>       {
>         "table": {
>           "table_name": "s2",
>           "access_type": "eq_ref",
>           "possible_keys": [
>             "idx_key2"
>           ],
>           "key": "idx_key2",
>           "used_key_parts": [
>             "key2"
>           ],
>           "key_length": "5",
>           "ref": [
>             "atguigudb_2.s1.key1"
>           ],
>           "rows_examined_per_scan": 1,
>           "rows_produced_per_join": 989,
>           "filtered": "100.00",
>           "index_condition": "(`atguigudb_2`.`s1`.`key1` = `atguigudb_2`.`s2`.`key2`)",
>           "cost_info": {
>             "read_cost": "247.38",
>             "eval_cost": "98.95",
>             "prefix_cost": "1360.08",
>             "data_read_per_join": "1M"
>           },
>           "used_columns": [
>             "id",
>             "key1",
>             "key2",
>             "key3",
>             "key_part1",
>             "key_part2",
>             "key_part3",
>             "common_field"
>           ]
>         }
>       }
>     ]
>   }
> }
> ```

- **TREE格式**

> **TREE格式是8.0.16版本之后引入的新格式，主要根据查询的 各个部分之间的关系 和 各部分的执行顺序 来描述如何查询。**
>
> ```markdown
> -> Nested loop inner join  (cost=1360.08 rows=990)
>     -> Filter: ((s1.common_field = 'a') and (s1.key1 is not null))  (cost=1013.75 rows=990)
>         -> Table scan on s1  (cost=1013.75 rows=9895)
>     -> Single-row index lookup on s2 using idx_key2 (key2=s1.key1), with index condition: (s1.key1 = s2.key2)  (cost=0.25 rows=1)
> 
> ```
>
> 

- **可视化输出**

> **workbench**



### 4.6.4 分析优化器执行计划：trace





# 五、索引优化与查询优化

> **SQL优化的技术有很多，但是大方向上完全可以分为`物理查询优化`和`逻辑查询优化`两大块**
>
> - **物理查询优化是通过`索引`和`表连接方式`等技术进行优化**
> - **逻辑优化方式是通过`SQL等价变化`提升查询效率**



## 5.1 索引失效案例

### 5.1.1 全值匹配效率高



### 5.1.2 最佳做前缀法则

> **	最左优先，在检索数据时从联合索引的最左边开始匹配**
>
> ​	**MySQL可以为为多个字段创建索引，`一个索引可以包括16个字段`，对于多列索引，`过滤条件要使用索引必须按照索引建立时的顺序`,一旦跳过某个字段，索引后面的字段都无法使用**





### 5.1.3 主键插入顺序

> ​	**对于一个使用`InnoDB`存储引擎的表来说，在我们没有显式的创建索引，表中的数据实际上都是存储在`聚簇索引`的叶子节点，而记录又是存储在数据页中的，数据页和记录又是按照记录`主键值从小到大`的顺序进行排序，所以如果我们`插入`的记录的`主键值时依次增大`的话，那么我们每插满一个数据页就换到下一个数据页继续插入，但是如果让`主键值随机取值`那么就比较麻烦了**
>
> ​	**如果当前页数据已满，再进行插入操作我们就需要将当前`页面分裂`成两个页面，将本页中的数据一些记录移动到新创建的这个页面中。所以我们`尽量让主键依次递增`，减少性能损耗.所以建议让主键具有`AUTO_INCREMENT`，让存储引擎自己为表生成主键，而不是我们手动插入**



### 5.1.4 计算、函数、类型转换（自动或者手动）导致索引失效



### 5.1.5 范围条件右边的列索引失效

> ​	**在应用开发中，例如金额查询，日期查询往往都是范围查询。应该将查询条件放置where语句最后（创建联合索引中，务必把范围涉及到的字段写在最后）**

```sql
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE student.age=30 AND student.classId>20 AND student.name = 'abc' ;#索引可能会失效

EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE student.age=30 AND student.name = 'abc' AND student.classId>20 ;#优化后
```



### 5.1.6 不等于（!=或者<>）索引失效

### 5.1.7 is null 可以使用索引 ，is not null无法使用索引

> ​	**最好在设计数据表的时候将`字段设置为 NOT NULL 约束`，比如可以将INT类型的字段默认值设置为0，将字符类型的默认值设置为空字符串`''`。同理在查询中，`not like`也无法使用索引**



### 5.1.8 `like以通配符%开头导致索引失效`



### 5.1.9 OR前后存在非索引的列，索引失效

> ​	**`在WHERE子句中，如果在OR前的条件进行了索引，而在OR后的条件列没有进行索引，那么索引会失效`，也就是说，==OR前后的两个条件中的列都是索引时，查询中才能使用索引==**
>
> ​	**因为OR的含义就是满足一个即可，因此`只有一个条件进行了索引是没有意义的`，==只要有条件列没有进行索引，就会进行全表扫描==，因此索引的条件也会失效**



### 5.1.10 数据库和表的字符集统一使用utf8mb4



## 5.2 关联查询优化



> **结论一：对于内连接来讲，如果连接条件中只能有一个字段有索引，则有索引的字段所在的表会被作为被驱动表**
>
> **结论二：对于内连接来说，在两个表的连接条件都存在索引的情况下，会选择小表作为驱动表。`小表驱动大表`**



### 5.2.1 JOIN语句的原理

> ​	**join方式连接多个表，本质就是各个表之间的数据的循环匹配。MySQL5.5版本前，MySQL只支持嵌套循环（Nested Loop Join）。MySQL5.5之后引入了BNLJ算法来优化嵌套循环**



**1.`Simple Nested-Loop Join`**

![](E:\阶段性资料\笔记\pic\QQ截图20221003211249.png)

> **效率较低，笛卡尔积运算，A表100条数据，B表1000条数据则进行10万次运算**

| **开销统计**         | SNLJ          |
| -------------------- | ------------- |
| **外表扫描次数**     | `1`           |
| **内表扫描次数**     | `A`           |
| **读取记录数**       | `A + (B * A)` |
| **JOIN比较次数**     | `B * A`       |
| **回表读取记录次数** | `0`           |



**2.`Index Nested-Loop Join`（索引嵌套循环连接）**

> ​	**Index Nested-Loop Join优化思路是为了`减少内层表数据的匹配次数`，所以要求`被驱动表`上必须有==索引==才行，通过外层表匹配条件直接与内层表索引进行匹配，避免和内存表的每条数据进行比较**



![](E:\阶段性资料\笔记\pic\QQ截图20221004155414.png)



| **开销统计**         | SNLJ          | INLJ                     |
| -------------------- | ------------- | ------------------------ |
| **外表扫描次数**     | `1`           | `1`                      |
| **内表扫描次数**     | `A`           | `0`                      |
| **读取记录数**       | `A + (B * A)` | `A + B (maych)`          |
| **JOIN比较次数**     | `B * A`       | `A * Index (Height)`     |
| **回表读取记录次数** | `0`           | `B (match)(if possible)` |



**3.`Block Nested-Loop Join`(块嵌套循环连接)**

> ​	**不再逐条回去驱动表的数据，而是一块一块的获取，引入到`join buffer 缓冲区`，将驱动表join相关的部分数据缓存到join buffer中，然后全表扫描被驱动表，被驱动表的每一条记录一次性和join buffer中的所有驱动表记录进行匹配，将简单的嵌套循环中的多次比较合并成一次，降低了被驱动表的访问频率**



![](E:\阶段性资料\笔记\pic\QQ截图20221004162521.png)



| **开销统计**         | SNLJ          | INLJ                     | BNLJ                                           |
| -------------------- | ------------- | ------------------------ | ---------------------------------------------- |
| **外表扫描次数**     | `1`           | `1`                      | `1`                                            |
| **内表扫描次数**     | `A`           | `0`                      | `A * used_column_size/join_buffer_siaze + 1`   |
| **读取记录数**       | `A + (B * A)` | `A + B (maych)`          | `A + B * (used_column_size/join_buffer_siaze)` |
| **JOIN比较次数**     | `B * A`       | `A * Index (Height)`     | `B * A`                                        |
| **回表读取记录次数** | `0`           | `B (match)(if possible)` | `0`                                            |



**参数设置：**

- **blick_nested_loop**

> ​	**`show variables like '%optimizer_switch%'`查看`block_nested_loop`状态**



- **join_buffer_size**

> ​	**驱动表能否一次性加载完，就要看join buffer能不能存储所有的数据，默认情况下`join_buffer_size=256k`,最大可申请4个G**
>
> ```sql
> show VARIABLES like '%join_buffer%'
> ```



**小结：**

> **1.整体效率：INLJ > BNLJ > SNLJ**
>
> **2.永远用小结果集驱动大结果集（本质上是减少外层循环的数据数量）**
>
> **3.保证被驱动表的JOIN字段已经创建了索引**
>
> **4.需要JOIN 的字段，数据类型保持绝对一致**
>
> **5.INNER JOIN 时，`MySQL会自动将小结果集的表选为驱动表`**
>
> **6.适当增大`join buffer size`的大小**
>
> **7.减少驱动表不必要的查询字段（字段越少，join buffer所缓存的数据就越少）**



### 5.2.2 Hash Join（新特性）

> ​	**`从MySQL8.0.20版本开始废弃BNLJ，使用Hash Join进行替代`**

- **`Nested Loop:`对于数据量较小的情况是一个较好的选择**
- **`Hash Join:`针对于`大数据集连接`时的常用方式，优化器使用两个表中较小的表利用`Join Key`在内存中建立`散列表`，然后扫描较大的表并探测散列表，找到与`Hash`表匹配的行**
  		- **较小的表可以直接放入内存当中，总成本就是访问两个表的成本之和**
  		- **当表很大的时候，优化器会将其分割成`若干个不同的分区`，不能放入内存的部分就改吧分区写入磁盘的临时段，此时要求有较大的临时段从而提高I/O性能**
  		- **`它能够很好的用于没有索引的大表并行查询的环境中，并提供最好的性能`,==Hash Join只能用于等值连接==**

| **类别**     | Nested Loop                                                  | Hash Join                                                    |
| :----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **使用条件** | **任何条件**                                                 | **等值连接**                                                 |
| **相关资源** | **CPU、磁盘I/O**                                             | **内存、临时空间**                                           |
| **特点**     | **当有高选择性索引或进行限制性搜索时，能够快速返回第一次的搜索结果** | **当缺乏索引或者索引条件模糊时，Hash Join 比 Nested Loop 更加有效，如果表的数据量很大，效率则相对更高** |
| **缺点**     | **当索引丢失或者查询条件限制不够时，效率很低，表的记录多时效率也很低** | **建立哈希表需要大量内存，第一次返回结果时间很长**           |



## 5.3 子查询优化

> ​	**子查询是 MySQL 的一项重要的功能，可以帮助我们通过一个 SQL 语句实现比较复杂的查询。但是，子查询的执行效率不高**
>
> - **执行子查询时，MySQL需要为内层查询语句的查询结果 建立一个临时表 ，然后外层查询语句从临时表中查询记录。查询完毕后，再 撤销这些临时表 。这样会消耗过多的CPU和IO资源，产生大量的慢查询**
>
> - **子查询的结果集存储的临时表，不论是内存临时表还是磁盘临时表都 不会存在索引 ，所以查询性能会受到一定的影响**
>
> - **对于返回结果集比较大的子查询，其对查询性能的影响也就越大**
>
> **结论：`尽量不要使用NOT IN 或者 NOT EXISTS，用LEFT JOIN xxx ON xx WHERE xx IS NULL替代`**



## 5.4 排序优化

> **在MySQL中，支持两种排序方式，分别是`FileSort`和`Index`排序**
>
> - **Index排序中，索引可以保证数据的有序性，不需要再进行排序，`效率很高`**
> - **FileSort排序一般再`内存中`进行，占用`CPU较多`，如果文件过大，会产生临时文件I/O到磁盘中进行排序，效率较低**



**`优化建议：`**

> 1. **SQL 中，可以在 WHERE 子句和 ORDER BY 子句中使用索引，目的是在 WHERE 子句中 避免全表扫 描 ，在 ORDER BY 子句 避免使用 FileSort 排序 。当然，某些情况下全表扫描，或者 FileSort 排序不一定比索引慢。但总的来说，我们还是要避免，以提高查询效率。**
> 2. **`尽量使用 Index 完成 ORDER BY 排序。如果 WHERE 和 ORDER BY 后面是相同的列就使用单索引列；如果不同就使用联合索引`。**
> 3. **无法使用 Index 时，需要对 FileSort 方式进行调优。**



### 5.4.1 order by 未使用limit条件，索引失效

```sql
# order by 时不使用 limit 索引失效
CREATE INDEX IDX_AGE_CLASSID_NAME ON STUDENT (AGE,CLASSID,NAME)

EXPLAIN SELECT * FROM STUDENT ORDER BY AGE,CLASSID  #不加以限制，索引失效（回表操作消耗资源巨大，MySQL优化器选择全表查询）

EXPLAIN SELECT * FROM STUDENT ORDER BY AGE,CLASSID LIMIT 50 #增加limit过滤条件，使用上了索引

EXPLAIN SELECT AGE,CLASSID FROM STUDENT ORDER BY AGE,CLASSID #没有回表操作的话，使用上了索引
```



### 5.4.2 order by 顺序错误，索引失效  



### 5.4.3 filesort算法：双路排序和单路排序

**`双路排序（慢）：`**

> - **MySQL 4.1之前是使用双路排序 ，字面意思就是两次扫描磁盘，最终得到数据， 读取行指针和order by列 ，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出**
> - **从磁盘取排序字段，在buffer进行排序，再从 磁盘取其他字段 。**

 

**`单路排序（快）：`**

> ​	**从磁盘读取查询需要的 `所有列` ，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出， 它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是它会使用更多的空间， 因为它把每一行都保存在内存中了。**
>
> ​	**在sort_buffer中，单路要比多路`占用更多的空间`，所以有可能去除的数据总大小超出了`sort_buffer`的容量，`导致每次只取`sort_buffer`容量大小的数据进行排序（创建tmp文件，多路合并）`，从而多次I/O,这样反而得不偿失**
>
> 
>
> **`优化策略：`**
>
> - **`尝试提高sort_buffer_size：无论那种算法，提高这个参数都会提高效率,这个参数是针对每个进程connection的 1M-8M之间调整，InnoDB存储引擎默认是1048576字节，也就是1MB`**
>
> ```sql
> SHOW VARIABLES LIKE '%SORT_BUFFER_SIZE%'
> ```
>
> | Variable_name           | Value   |
> | ----------------------- | ------- |
> | innodb_sort_buffer_size | 1048576 |
> | myisam_sort_buffer_size | 8388608 |
> | sort_buffer_size        | 262144  |
>
> - **`尝试提高 max_length_for_sort_data`:提高这个参数会增加用改进算法的改率。如果需要返回列的总长度大于`max_length_for_sort_data`，使用双路算法，否则使用单路算法，1024-8192字节之间调整。`（如果设置的过高，数据总容量容易超过sort_buffer_size，容易产生高的磁盘I/O活动和低的处理器使用率）`**
>
> ```sql
> SHOW VARIABLES LIKE '%MAX_LENGTH_FOR_SORT_DATA%' #默认1024字节
> ```
>
> 



## 5.5 GROUP BY优化

> - **group by 使用索引的原则几乎跟order by一致 ，group by 即使没有过滤条件用到索引，也可以直接使用索引。**
> - **group by 先排序再分组，遵照索引建的最佳左前缀法则**
> - **当无法使用索引列，增大 `max_length_for_sort_data` 和 `sort_buffer_size` 参数的设置**
> - **where效率高于having，能写在where限定的条件就不要写在having中了**
> - **减少使用order by，和业务沟通能不排序就不排序，或将排序放到程序端去做。Order by、group by、distinct这些语句较为耗费CPU，数据库的CPU资源是极其宝贵的**
> - **包含了order by、group by、distinct这些查询的语句，where条件过滤出来的结果集请保持在1000行以内，否则SQL会很慢。**



## 5.6 优化分页查询





## 5.7 优先使用覆盖索引

> **`一个索引包含了满足查询结果的数据就叫做覆盖索引`**



**好处：**

> - **避免Innodb表进行索引的二次查询（回表）**
> -  **避免Innodb表进行索引的二次查询（回表）**



## 5.8 索引下推（ICP）



### 5.8.1 ICP的使用条件

> - **只能用于二级索引(secondary index)**
> - **explain显示的执行计划中type值（join 类型）为 range 、 ref 、 eq_ref 或者 ref_or_null 。**
> - **并非全部where条件都可以用ICP筛选，如果where条件的字段不在索引列中，还是要读取整表的记录到server端做where过滤。**
> - **ICP可以用于MyISAM和InnnoDB存储引擎**
> - **MySQL 5.6版本的不支持分区表的ICP功能，5.7版本的开始支持**
> - **当SQL使用覆盖索引时，不支持ICP优化方法**



## 5.9 其他优化策略

### 5.9.1  EXISTS 和 IN 区分

```sql
SELECT * FROM A WHERE CC IN (SELECT CC FROM B)#不相关子查询
SELECT * FROM A WHERE EXISTS (SELECT CC FROM B WHERE B.CC = A.CC) #相关子查询
```

>- **`当A小于B时用EXISTS，因为EXISTS的实现，相当于外表循环`**
>
>```python
>for i in A
>	for j in B
>    	if j.cc == i.cc then... 
>```
>
>- **`当B小于A用IN`**
>
>```python
>for i in B
>	for j in A
>    	if j.cc == i.cc then...
>```
>
>**结论：==那个表小就用哪个表驱动，A表小就用 EXISTS ，B表小就用 IN==**



### 5.9.2 COUNT(*)、COUNT(1)、COUNT(具体字段)效率

> - **`COUNT(*)`与`COUNT(1)`本质上没有区别**
> - **如果时`MyISAM`存储引擎，`统计数据表的行数只需要O(1)的复杂度`，因为每张`MyISAM`的数据表都有一个meta信息存储了`row_count`值，而一致性则由表级锁来保证**
> - **如果时`InnoDB`存储引擎，因为`InnoDB`支持事务，采用`行级锁和MVCC机制`，无法像`MyISAM`一样去维护一个`row_count`变量，`因此需要扫描全表，是O(n)的复杂度，采用循环计数来完成统计`**
> - **在`InnoDB`中，如果采用COUNT(具体字段)来统计，要尽量采用二级索引，因为主键采用的是聚簇索引，聚簇索引包含的信息很多，明显会大于二级索引。对于`COUNT(1)`和`COUNT(*)`来说，系统会自动选择占用空间更小的二级索引来统计。如果有多个二级索引，会使用`key_len`小的二级索引进行扫描，当没有二级索引的时候，才会采用主键索引来进行统计**





### 5.9.3 多使用`COMMIT`

> **在程序中多使用COMMIT，这样程序性能得到提高，需求也会因为COMMIT所释放的资源而减少**





# 六、数据库的设计规范



# 七、数据库其他调优策略



# 八、事务



# 九、锁







