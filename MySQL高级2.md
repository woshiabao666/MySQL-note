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

## 6.1 范式(Normal Form)

### 6.1.1 范式简介

> ​	**在关系型数据库中，关于数据表设计的基本原则、规则就称为范式。可以理解为，一张数据表的设计结构需要满足的某种设计标准的`级别`**
>
> ​	**目前关系型数据库有六种常见范式，按照范式级别，从低到高分别是：`第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴斯-科德范式（BCNF）、第四范式(4NF）和第五范式（5NF，又称完美范式）`**



![](E:\阶段性资料\笔记\pic\QQ截图20221007220019.png)



### 6.1.2 键和相关属性的概念

> - **`超键：`能唯一标识元组的属性集叫超键**
> - **`候选键：`如果超键不包括多余的属性，那么这个超键就是候选键**
> - **`主键：`用户可以从候选键中选择一个作为主键**
> - **`外键：`如果数据表R1中的某属性集不是R1的主键，而是另一个数据表R2的主键，那么这个属性集就是数据表R1的外键**
> - **`主属性：`包含在任一候选键中的属性称为主属性**
> - **`非主属性：`与主属性相对，指的是不包含任一候选键中的属性**

**举例：**

>**`球员表(player)` ：球员编号 | 姓名 | 身份证号 | 年龄 | 球队编号**
>
>**`球队表(team) `：球队编号 | 主教练 | 球队所在地**
>
>
>
>- **`超键：`对于球员表来说，超键就是包括球员编号或者身份证号的任意组合，比如（球员编号）（球员编号，姓名）（身份证号，年龄）等**
>- **`候选键：`就是最小的超键，对于球员表来说，候选键就是（球员编号）或者（身份证号）**
>- **`主键：`我们自己选定，也就是从候选键中选择一个，比如（球员编号）**
>- **`外键：`球员表中的球队编号**
>- **`主属性、非主属性：`在球员表中，主属性是（球员编号）（身份证号），其他的属性（姓名）（年龄）（球队编号）都是非主属性**



### 6.1.1 第一范式（1st NF）

> ​	**第一范式主要是确保数据表每个字段的值必须具有`原子性`，也就是说数据表的每个字段的值为`不可拆分`的最小数据单元**



### 6.1.2 第二范式（2nd NF）

> ​	**`满足数据库表里的每一条数据记录，都是可唯一标识的，而且所有的非主键字段都必须完全依赖主键，不能只依赖主键的一部分`**



**举例1：**

> ​	**成绩表 （学号，课程号，成绩）关系中，（学号，课程号）可以决定成绩，但是学号不能决定成绩，课程号也不能决定成绩，**
>
> ​	**所以“`（学号，课程号）→成绩`”就是`完全依赖关系`。**



**举例2：**

> ​	**比赛表 player_game ，里面包含球员编号、姓名、年龄、比赛编号、比赛时间和比赛场地等属性，这里候选键和主键都为（球员编号，比赛编号），我们可以通过候选键（或主键）来决定如下的关系：**
>
> ```markdown
> (球员编号, 比赛编号) → (姓名, 年龄, 比赛时间, 比赛场地，得分)
> ```
>
> ​	**但是这个数据表不满足第二范式，因为数据表中的字段之间还存在着如下的对应关系：**
>
> ```
> (球员编号) → (姓名，年龄)
> 
> (比赛编号) → (比赛时间, 比赛场地)
> ```
>
> - **对于非主属性来说，并非完全依赖于候选键，会产生以下问题：**
>
> >**`1.数据冗余`：如果一个球员可以参加 m 场比赛，那么球员的姓名和年龄就重复了 m-1 次。一个比赛也可能会有 n 个球员参加，比赛的时间和地点就重复了 n-1 次。**
> >
> >**`2.插入异常` ：如果我们想要添加一场新的比赛，但是这时还没有确定参加的球员都有谁，那么就没法插入。**
> >
> >**`3.删除异常` ：如果我要删除某个球员编号，如果没有单独保存比赛表的话，就会同时把比赛信息删除掉**
> >
> >**`4.更新异常` ：如果我们调整了某个比赛的时间，那么数据表中所有这个比赛的时间都需要进行调整，否则就会出现一场比赛时间不同的情况。**
>
> - **我们可以将`例2`优化为以下3张表，这样就符合第二范式了。**
> - **==1NF 告诉我们字段属性需要是原子性的，而 2NF 告诉我们一张表就是一个独立的对象，一张表只表达一个意思==**
>
> | 表名                             | 属性（字段）                           |
> | -------------------------------- | -------------------------------------- |
> | **球员 player 表**               | **球员编号、姓名和年龄等属性**         |
> | **比赛 game 表 **                | **比赛编号、比赛时间和比赛场地等属性** |
> | **球员比赛关系 player_game 表 ** | **球员编号、比赛编号和得分等属性**     |
>
> 



### 6.1.3 第三范式（3rd NF）

> ​	**第三范式是在第二范式基础上，确保数据表中的每一个非主键字段都和主键字段直接相关，也就是说：`数据表中的所有非主键字段不能依赖于其他非主键字段`==（即不能存在非主属性A依赖于非主属性B，非主属性B依赖于主键C的情况，即存在“A=>B=>C”）==。该规则的意思是所有`非主键属性`之间不能有依赖关系，必须相互独立**

**举例1：**

> **`部门信息表：`每个部门有部门编号（dept_id）、部门名称、部门简介等信息**
>
> **`员工信息表:`每个员工有员工编号、姓名、部门编号。`列出部门编号后就不能再将部门名称、部门简介等与部门有关的信息再加入员工信息表中`**



**结论：**

> **符合3NF后的数据模型通俗地讲，2NF和3NF通常以这句话概括：“`每个非键属性依赖于键，依赖于整个键，并且除了键别无他物`”**



### 6.1.4 范式小结

> **`范式的优点`：数据的标准化有助于消除数据库中的`数据冗余`,第三范式通常被认为在性能、扩展性和数据完整性方面达到了最好的平衡**
>
> **`范式的缺点`：范式的使用，可能`降低查询的效率`。因为范式等级越高，设计出来的数据表也就越多、越精细，数据冗余度就越低，进行查询的时候就要`关联多张表`，不仅代价昂贵，也可能使一些`索引策略无效`**



## 6.2 反范式化

> ​	**有些数据对业务十分的重要，这时候我们就需要遵循`业务优先`的原则，首先满足业务需求，再尽量减少冗余**





## 6.3 BCNF(巴斯范式)

> ​	**若一个关系达到了第三范式，并且它只有一个候选键，或者它的每个候选键都是单属性，则该关系自然达到BC范式。**

## 6.4 第四范式

## 6.5 第五范式、域键范式









# 七、数据库其他调优策略

## 7.1 数据库调优措施

> - **尽可能 节省系统资源 ，以便系统可以提供更大负荷的服务(`吞吐量更大`）**
> - **合理的结构设计和参数调整，以提高用户操作 响应的速度(`响应速度更快`）**
> - **减少系统的瓶颈，提高MySQL数据库整体的性能**



## 7.2 优化MySQL的参数

> - **==innodb_buffer_pool_size==：这个参数是Mysql数据库最重要的参数之一，表示InnoDB类型的 `表和索引的最大缓存 `。它不仅仅缓存 `索引数据` ，还会缓存 `表的数据` 。这个值越大，查询的速度就会越快。但是这个值太大会影响操作系统的性能。**
> - **==key_buffer_size== ：表示 `索引缓冲区的大小` 。索引缓冲区是所有的 `线程共享` 。增加索引缓冲区可以得到更好处理的索引（对所有读和多重写）。当然，这个值不是越大越好，它的大小取决于内存的大小。如果这个值太大，就会导致操作系统频繁换页，也会降低系统性能。对于内存在 `4GB` 左右的服务器该参数可设置为 `256M 或 384M `。**
> - **==table_cache==：表示 `同时打开的表的个数 `。这个值越大，能够同时打开的表的个数越多。物理内存越大，设置就越大。默认为2402，调到512-1024最佳。这个值不是越大越好，因为同时打开的表太多会影响操作系统的性能。**
> - **==query_cache_size== ：表示 `查询缓冲区的大小` 。可以通过在MySQL控制台观察，如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况，就要增加Query_cache_size的值；如果Qcache_hits的值非常大，则表明查询缓冲使用非常频繁，如果该值较小反而会影响效率，那么可以考虑不用查询缓存；Qcache_free_blocks，如果该值非常大，则表明缓冲区中碎片很多。MySQL8.0之后失效。该参数需要和query_cache_type配合使用。**
> - **==query_cache_type== 的值是0时，所有的查询都不使用查询缓存区。但是query_cache_type=0并不会导致MySQL释放query_cache_size所配置的缓存区内存。**
>   - **当query_cache_type=1时，所有的查询都将使用查询缓存区，除非在查询语句中指定SQL_NO_CACHE ，如SELECT SQL_NO_CACHE * FROM tbl_name。**
>   - **当query_cache_type=2时，只有在查询语句中使用 SQL_CACHE 关键字，查询才会使用查询缓存区。使用查询缓存区可以提高查询的速度，这种方式只适用于修改操作少且经常执行相同的查询操作的情况**。
> - **==sort_buffer_size== ：`表示每个需要进行排序的线程分配的缓冲区的大小 `。增加这个参数的值可以提高 ORDER BY 或 GROUP BY 操作的速度。默认数值是2 097 144字节（约2MB）。对于内存在4GB左右的服务器推荐设置为6-8M，如果有100个连接，那么实际分配的总共排序缓冲区大小为100 × 6 ＝ 600MB。**
> - **==join_buffer_size = 8M== ：表示 联合查询操作所能使用的缓冲区大小 ，和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。**
> - **==read_buffer_size==：`表示 每个线程连续扫描时为扫描的每个表分配的缓冲区的大小（字节）` 。当线程从表中连续读取记录时需要用到这个缓冲区。SET SESSION read_buffer_size=n可以临时设置该参数的值。默认为64K，可以设置为4M。**
> - **==innodb_flush_log_at_trx_commit== ：表示 `何时将缓冲区的数据写入日志文件` ，并且将日志文件写入磁盘中。该参数对于innoDB引擎非常重要。该参数有3个值，分别为0、1和2。该参数的默认值为1。**
>   - **`值为 0 时`，表示 每秒1次 的频率将数据写入日志文件并将日志文件写入磁盘。每个事务的commit并不会触发前面的任何操作。该模式速度最快，但不太安全，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失。**
>   - **`值为 1 时`，表示 每次提交事务时 将数据写入日志文件并将日志文件写入磁盘进行同步。该模式是最安全的，但也是最慢的一种方式。因为每次事务提交或事务外的指令都需要把日志写入（flush）硬盘。**
>   - **`值为 2 时`，表示 每次提交事务时 将数据写入日志文件， 每隔1秒 将日志文件写入磁盘。该模式速度较快，也比0安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。**
> - **==innodb_log_buffer_size== ：`这是 InnoDB 存储引擎的 事务日志所使用的缓冲区` 。为了提高性能，也是先将信息写入 Innodb Log Buffer 中，当满足 innodb_flush_log_trx_commit 参数所设置的相应条件（或者日志缓冲区写满）之后，才会将日志写到文件（或者同步到磁盘）中。**
> - **==max_connections== ：`表示 允许连接到MySQL数据库的最大数量` ，默认值是 `151` 。如果状态变量connection_errors_max_connections 不为零，并且一直增长，则说明不断有连接请求因数据库连接数已达到允许最大值而失败，这是可以考虑增大max_connections 的值。在Linux 平台下，性能好的服务器，支持 500-1000 个连接不是难事，需要根据服务器性能进行评估设定。这个连接数 `不是越大越好` ，因为这些连接会浪费内存的资源。过多的连接可能会导致MySQL服务器僵死。**
> - **==back_log== ：用于 `控制MySQL监听TCP端口时设置的积压请求栈大小 `。如果MySql的连接数达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源，将会报错。5.6.6 版本之前默认值为 50 ， 之后的版本默认为 50 + （max_connections / 5）， 对于Linux系统推荐设置为小于512的整数，但最大不超过900。**
>   **如果需要数据库在较短的时间内处理大量连接请求， 可以考虑适当增大back_log 的值。**
> - **==thread_cache_size==： 线程池缓存线程数量的大小 ，当客户端断开连接后将当前线程缓存起来，当在接到新的连接请求时快速响应无需创建新的线程 。这尤其对那些使用短连接的应用程序来说可以极大的提高创建连接的效率。那么为了提高性能可以增大该参数的值。默认为60，可以设置为120。可以通过如下几个MySQL状态值来适当调整线程池的大小：**
> - **==wait_timeout==：指定 `一个请求的最大连接时间` ，对于4GB左右内存的服务器可以设置为5-10**
> - **==interactive_timeout== ：`表示服务器在关闭连接前等待行动的秒数`**





## 7.3 优化数据库结构



### 7.3.1 拆分表：冷热数据分离





























# 八、事务

## 8.1 事务基本概念

### 8.1.1 事务的ACID特性

> - **`原子性（atomicity）`：原子性是指事务是一个不可分割的工作单位，要么全部提交，要么全部失败回滚**
> - **`一致性（consistency）`：根据定义，一致性是指事务执行前后，数据从一个 合法性状态 变换到另外一个 合法性状态 。这种状态是 语义上的而不是语法上的，跟具体的业务有关**
> - **`隔离型（isolation）`：事务的隔离性是指一个事务的执行 不能被其他事务干扰 ，即一个事务内部的操作及使用的数据对 并发 的其他事务是隔离的，并发执行的各个事务之间不能互相干扰**
> - **`持久性（durability）`：持久性是指一个事务一旦被提交，它对数据库中数据的改变就是 永久性的 ，接下来的其他操作和数据库故障不应该对其有任何影响。**
>   - **持久性是通过 `事务日志` 来保证的。日志包括了 `重做日志` 和 `回滚日志` 。当我们通过事务对数据进行修改的时候，首先会将数据库的变化信息记录到重做日志中，然后再对数据库中对应的行进行修改。这样做的好处是，即使数据库系统崩溃，数据库重启后也能找到没有更新到数据库系统中的重做日志，重新执行，从而使事务具有持久性**





### 8.1.2 事务的状态



![](E:\阶段性资料\笔记\pic\QQ截图20221009165617.png)



## 8.2 如何使用事务



### 8.2.1 显式事务

> ​	**使用关键字：`START TRANSACTION`或者`BEGIN`显式的开启一个事务**
>
> ​	**其中`START TRANSACTION`后面能添加修饰词：**
>
> - **`READ ONLY`:标识当前事务是一个 只读事务 ，也就是属于该事务的数据库操作只能读取数据，而不能修改数据**
> - **`READ WRITE`:标识当前事务是一个 读写事务 ，也就是属于该事务的数据库操作既可以读取数据，也可以修改数据**
> - **`WITH CONSISTENT SNAPSHOT` ：启动一致性读**

**`SAVEPOINT`：**

> - **在事务中创建保存点，方便后续针对保存点进行回滚，一个事务可以存在多个保存点**
>
> ```sql
> SAVEPOINT 保存点名称;
> 
> RELEASE SAVEPOINT 保存点名称;#删除保存点
> ```



### 8.2.2 隐式事务

> **`autocommit`:默认是ON**
>
> ```sql
> show variables like 'autocommit'
> ```
>
> | Variables_name | Value |
> | -------------- | ----- |
> | `autocommit`   | `ON`  |
>
> - **关闭自动提交：**
>
> ```SQL
> #方式一
> #针对于DML
> SET autocommit = FALSE 
> SET autocommit = 0;
> 
> #方式二
> 显式的的使用 START TRANSACTION 或者 BEGIN 语句开启一个事务。这样在本次事务提交或者回滚前会暂时关闭掉自动提交的功能
> ```

  



### 8.2.3 隐式提交数据的情况

> - **数据定义语言（Data definition language，缩写为：DDL）**
>
> 
>
> - **隐式使用或修改mysql数据库中的表**
>
> 
>
> - **事务控制或关于锁定的语句**
>
> > **1.当我们在一个事务还没提交或者回滚时就又使用 START TRANSACTION 或者 BEGIN 语句开启了另一个事务时，会 `隐式的提交上一个事务`**
> >
> > 
> >
> > **2.当前的 autocommit 系统变量的值为 OFF ，我们手动把它调为 ON 时，也会 `隐式的提交` 前边语句所属的事务**
> >
> > 
> >
> > **3.使用` LOCK TABLES 、 UNLOCK TABLES` 等关于锁定的语句也会` 隐式的提交 `前边语句所属的事务**
>
> 
>
> - **加载数据的语句**
>
> 
>
> - **关于MySQL复制的一些语句**
>
> 
>
> - **其它的一些语句**







## 8.3 事务的并发问题

> ​	**MySQL是一个 客户端／服务器 架构的软件，对于同一个服务器来说，可以有若干个客户端与之连接，每个客户端与服务器连接上之后，就可以称为一个会话（ Session ）。每个客户端都可以在自己的会话中向服务器发出请求语句，一个请求语句可能是某个事务的一部分，也就是对于服务器来说可能同时处理**
> **多个事务。事务有 隔离性 的特性，理论上在某个事务 对某个数据进行访问 时，其他事务应该进行 排 队 ，当该事务提交之后，其他事务才可以继续访问这个数据。但是这样对 性能影响太大 ，我们既想保持事务的隔离性，又想让服务器在处理访问同一数据的多个事务时 性能尽量高些 ，那就看二者如何权衡取舍了**





### 8.3.1 数据并发问题：脏写（ `Dirty Write`）

> **对于两个事务 Session A、Session B，如果事务Session A `修改了` 另一个 `未提交` 事务Session B `修改过` 的数据，那就意味着发生了 `脏写`**

| 发生时间编号 | Session A                                              | Session B                                              |
| ------------ | ------------------------------------------------------ | ------------------------------------------------------ |
| 1            | **BEGIN**                                              |                                                        |
| 2            |                                                        | **BEGIN**                                              |
| 3            |                                                        | `update student set name = '李四' where studentno = 1` |
| 4            | `update student set name = '张三' where studentno = 1` |                                                        |
| 5            | **COMMIT**                                             |                                                        |
| 6            |                                                        | **ROLLBACK**                                           |

### 8.3.2 数据并发问题：脏读（`Dirty Read`）

> ​	**对于两个事务 Session A、Session B，Session A `读取` 了已经被 Session B `更新` 但还 `没有被提交` 的字段。之后若 Session B `回滚` ，Session A `读取`的内容就是 `临时且无效` 的**

| 发生时间编号 | Session A                                                    | Session B                                              |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| 1            | **BEGIN**                                                    |                                                        |
| 2            |                                                              | **BEGIN**                                              |
| 3            |                                                              | `update student set name = '李四' where studentno = 1` |
| 4            | `select * from student where studentno = 1`（**如果读到name = '张三'则说明发生了脏读**） |                                                        |
| 5            | **COMMIT**                                                   |                                                        |
| 6            |                                                              | **ROLLBACK**                                           |

### 8.3.3 数据并发问题：不可重复读（`Non-Repeatable Read`）

> ​	**对于两个事务Session A、Session B，Session A `读取` 了一个字段，然后 Session B `更新` 了该字段。 之后Session A `再次读取` 同一个字段， `值就不同` 了。那就意味着发生了不可重复读。**

| 发生时间编号 | Session A                                                    | Session B                                              |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| 1            | **BEGIN**                                                    |                                                        |
| 2            | `select * from student where studentno = 1;`（**此时读到的列name的值为'王五'**） |                                                        |
| 3            |                                                              | `update student set name = '张三' where studentno = 1` |
| 4            | `select * from student where studentno = 1`（**如果读到name = '张三'则说明发生了不可重复读**） |                                                        |
| 5            |                                                              | `update student set name = '李四' where studentno = 1` |
| 6            | `select * from student where studentno = 1`(**如果读到列name的值为'李四'，则意味着发生了不可重复读**) |                                                        |

### 8.3.4 数据并发问题：幻读（`Phantom`）

> ​	**对于两个事务Session A、Session B, Session A 从一个表中 读取 了一个字段, 然后 Session B 在该表中 `插入` 了一些新的行。 之后, 如果 Session A `再次读取` 同一个表, 就会多出几行。那就意味着发生了幻读。**



| 发生时间编号 | Session A                                                    | Session B                                     |
| ------------ | ------------------------------------------------------------ | --------------------------------------------- |
| 1            | **BEGIN**                                                    |                                               |
| 2            | `select * from student where studentno > 1`（**此时读到的列name的值为`张三`**） |                                               |
| 3            |                                                              | `insert into student values (2,'赵六','2班')` |
| 4            | `select * from student where studentno > 1`（**如果读到name = '张三'、'赵六'的记录则说明发生了幻读**） |                                               |



## 8.4 SQL中的四种隔离级别

> **问题严重性：`脏写 > 脏读 > 不可重复读 > 幻读`**





- **`READ UNCOMMITTED:读未提交`**

> ​	**读未提交，在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。`不能避免脏读、不可重复读、幻读`**



- **`READ COMMITTED:读已提交`(`Oracle默认事务等级`)**

> ​	**它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。`可以避免脏读`，但不可重复读、幻读问题仍然存在**



- **`REPEATABLE READ:可重复读`（`MySQL默认事务等级`）**

> ​	**事务A在读到一条数据之后，此时事务B对该数据进行了修改并提交，那么事务A再读该数据，读到的还是原来的内容。`可以避免脏读、不可重复读`，但幻读问题仍然存在。这是MySQL的默认隔离级别**



- **`SERIALIZABLE:可串行化`**

> ​	**确保事务可以从一个表中读取相同的行。在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。`所有的并发问题都可以避免`，但性能十分低下。能避免脏读、不可重复读和幻读**



| 隔离级别                        | 脏读可能性 | 不可重复读可能性 | 幻读可能性 | 加锁读 |
| ------------------------------- | ---------- | ---------------- | ---------- | ------ |
| **`READ UNCOMMITTED:读未提交`** | YES        | YES              | YES        | NO     |
| **`READ COMMITTED:读已提交`**   | NO         | YES              | YES        | NO     |
| **`REPEATABLE READ:可重复读`**  | NO         | NO               | YES        | NO     |
| **`SERIALIZABLE:可串行化`**     | NO         | NO               | NO         | YES    |

## 8.4 MySQL支持的四种级别



> ​	**首先，Oracle只支持`READ COMMITTED:读已提交`和`SERIALIZABLE:可串行化`；MySQL虽然支持4种隔离级别，但是与SQL标准里中所规定的各级隔离级别允许发生的问题有所出入**



### 8.4.1 查看MySQL事务级别

```sql
# 查看隔离级别，MySQL 5.7.20的版本之前： 
SHOW VARIABLES LIKE 'tx_isolation';

# 查看隔离级别，MySQL 5.7.20的版本及之后： 
SHOW VARIABLES LIKE 'transaction_isolation';

# 或者不同MySQL版本中都可以使用的： 
SELECT @@transaction_isolation;
```



### 8.4.2 如何设置事务的隔离级别

```sql
SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL = '隔离级别'; 
#其中，隔离级别格式： 
> READ UNCOMMITTED 
> READ COMMITTED 
> REPEATABLE READ 
> SERIALIZABLE
```

**或者：**

```sql
SET [GLOBAL|SESSION] TRANSACTION_ISOLATION = '隔离级别' 
#其中，隔离级别格式： 
> READ-UNCOMMITTED 
> READ-COMMITTED 
> REPEATABLE-READ 
> SERIALIZABLE
```

**关于设置时使用`GLOBAL`或`SESSION`的影响：**

- **`使用 GLOBAL 关键字（在全局范围影响）：`**
  - **当前已经存在的会话无效**
  - **只对执行完该语句之后产生的会话起作用**

```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE; 
#或
SET GLOBAL TRANSACTION_ISOLATION = 'SERIALIZABLE';
```



- **使用 `SESSION` 关键字（在会话范围影响）：**
  - **对当前会话的所有后续的事务有效**
  - **如果在事务之间执行，则对后续的事务有效**
  - **该语句可以在已经开启的事务中间执行，但不会影响当前正在执行的事务**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE; 
#或
SET SESSION TRANSACTION_ISOLATION = 'SERIALIZABLE';
```







# 九、MYSQL事务日志

> **事务有4种特性：原子性、一致性、隔离性和持久性**
>
> - **事务的隔离性由 `锁机制` 实现**
> - **而事务的原子性、一致性和持久性由事务的 `redo 日志`和`undo 日志`来保证==REDO 与 UNDO 都可以视为一种恢复操作==**
>   - **`REDO LOG` 称为 `重做日志` ，==提供再写入操作，恢复提交事务修改的页操作，用来保证事务的持久性==**
>   - **`UNDO LOG` 称为 `回滚日志` ，==回滚行记录到某个特定版本，用来保证事务的原子性、一致性==**
>
> **`REDO LOG`:是存储引擎层(innodb)生成的日志，记录的是`物理级别`上页的修改记录，比如`页号`，`偏移量`**
>
> **`UNDO LOG`:是存储引擎层(innodb)生成的日志，记录的是`逻辑操作`日志，比如对某一行数据进行了`INSERT`操作，`undo log`就记录一条与之相反的`DELETE`操作，主要用于`事务的回滚`和`一致性非锁定读`**





![](E:\阶段性资料\笔记\pic\QQ截图20221013162535.png)

## 9.1 redo 日志

> ​	**InnoDB存储引擎是以`页为单位`管理存储空间的。在真正访问页面之前，需要把`磁盘`上的页缓存到内存中的`Buffer Pool`才能访问。所有的变更都必须`先更新缓冲池`中的数据，然后缓冲池中的`脏页`会以一定的频率被刷入磁盘（`checkPoint机制`），通过缓冲池来优化CPU与磁盘之间的鸿沟，这样就可以保证整体的性能不会下降太快**





### 9.1.1 redo日志的作用

> ​	**一方面，缓冲池可以帮助我们消除CPU和磁盘之间的鸿沟，`checkpoint`机制可以保证数据的最终落盘，然而由于`checkpoint `并不是每次变更的时候就触发 的，而是`master线程隔一段时间去处理`的。所以`最坏的情况就是事务提交后，刚写完缓冲池，数据库宕机了`，那么这段数据就是丢失的，无法恢复**
>
> ​	**`解决的思路` ：我们只是想让已经提交了的事务对数据库中数据所做的修改永久生效，即使后来系统崩溃，在重启后也能把这种修改恢复出来。所以我们其实没有必要在每次事务提交时就把该事务在内存中修改过的全部页面刷新到磁盘，只需要把 修改 了哪些东西 记录一下 就好。比如，某个事务将系统表空间中 第10号 页面中偏移量为 100 处的那个字节的值 1 改成 2 。我们只需要记录一下：将第0号表空间的10号页面的偏移量为100处的值更新为 2**
>
> ​	**InnoDB引擎的事务采用了WAL技术（`Write-Ahead-Logging`）这种技术的思想就是`先写日志再写磁盘`，只有日志写入成功才算事务提交成功，这里的日志就是`redo 日志`，当发生宕机且数据未刷新到磁盘的时候，可以通过`redo log`来恢复**





### 9.1.2 redo 日志的优点与特点

**`优点：`**

> - **redo日志降低了刷盘频率**
> - **redo日志占用的空间非常小**



**`特点：`**

> - **redo日志是顺序写入磁盘的**
> - **事务执行过程中，redo log不断记录**





### 9.1.3 redo日志的组成

**Redo log可以简单分为以下两个部分：**



> - **==重做日志的缓冲 (redo log buffer)==，保存在内存中，是易失的**

> ​	**在服务器启动时就向操作系统申请了一大片成为`redo log buffer`的`连续空间`，称为`redo日志缓冲区`。这片内存空间被划分为若干个连续的`redo log block`。一个`redo log block`占用`512字节`大小**



![](E:\阶段性资料\笔记\pic\QQ截图20221013164455.png)



**参数设置**：**==redo log buffer 大小，默认 16M ，最大值是4096M，最小值为1M==**

```sql
SHOW VARIABLES LIKE '%INNODB_LOG_BUFFER_SIZE%'
```

| Variable_name              | Value        |
| -------------------------- | ------------ |
| **innodb_log_buffer_size** | **16777216** |



>- **==重做日志文件 (redo log file)== ，保存在硬盘中，是持久的。**
>
>**`ib_logfile0与ib_logfile1即为redo日志`**



### 9.1.4 redo日志的运作流程

**以更新事务为例，redo log运转过程：**

![](E:\阶段性资料\笔记\pic\QQ截图20221013210303.png)

> **第1步：`先将原始数据从磁盘中读入内存中来，修改数据的内存拷贝`** 
>
> **第2步：`生成一条重做日志并写入redo log buffer，记录的是数据被修改后的值`** 
>
> **第3步：`当事务commit时，将redo log buffer中的内容刷新到 redo log file，对 redo log file采用追加写的方式`** 
>
> **第4步：`定期将内存中修改的数据刷新到磁盘中`**
>
> 
>
> **==Write-Ahead Log(预先日志持久化)==:在持久化一个数据页之前，先将内存中相应的日志页持久化**



### 9.1.5 redo日志的刷盘策略

> **redo log的写入并不是直接写入磁盘的，InnoDB引擎会在写redo log的时候先写redo log buffer，之后以`一定的频率`刷入到真正的redo log file中**

![](E:\阶段性资料\笔记\pic\QQ截图20221013210954.png)



> ​	**注意，redo log buffer刷盘到redo log file的过程并不是真正的刷到磁盘中去，只是刷入到 `文件系统缓存 （page cache）`中去（这是现代操作系统为了提高文件写入效率做的一个优化），`真正的写入会交给系统自己来决定（比如page cache足够大了）`。那么对于InnoDB来说就存在一个问题，如果交给系统来同**
> **步，同样如果系统宕机，那么数据也丢失了（虽然整个系统宕机的概率还是比较小的）。针对这种情况，InnoDB给出 `innodb_flush_log_at_trx_commit` 参数，该参数控制 commit提交事务时，如何将 redo log buffer 中的日志刷新到 redo log file 中。它支持三种策略：**
>
> - **`设置为0` ：表示每次事务提交时不进行刷盘操作。（系统默认`master thread`每隔1s进行一次重做日志的同步）**
> - **`设置为1` ：表示每次事务提交时都将进行同步，刷盘操作（ ==默认值==）**
> - **`设置为2` ：表示每次事务提交时都只把 `redo log buffer` 内容写入 `page cache`，不进行同步。由os自己决定什么时候同步到磁盘文件。**

```sql
SHOW VARIABLES LIKE '%innodb_flush_log_at_trx_commit%'
```

| Variable_name                      | Value |
| :--------------------------------- | ----- |
| **innodb_flush_log_at_trx_commit** | **1** |

 ### 9.1.6 redo日志刷盘策略详细流程

- **`innodb_flush_log_at_trx_commit = 1`**

![](E:\阶段性资料\笔记\pic\QQ截图20221014145324.png)





- **`innodb_flush_log_at_trx_commit = 2`**

![](E:\阶段性资料\笔记\pic\QQ截图20221014145423.png)





- **`innodb_flush_log_at_trx_commit = 0`**



![](E:\阶段性资料\笔记\pic\QQ截图20221014145517.png)





### 9.1.7 写入redo log buffer



## 9.2 Undo日志

> **redo log是事务持久性的保证，undo log是事务原子性的保证。在事务中 `更新数据` 的 `前置操作` 其实是要`先`写入一个 `undo log`**



### 9.2.1 什么是Undo日志

> ​	**事务需要保证 原子性 ，也就是事务中的操作要么全部完成，要么什么也不做。但有时候事务执行到一半会出现一些情况，比如：**
>
> - **情况一：事务执行过程中可能遇到各种错误，比如 `服务器本身的错误` ， `操作系统错误` ，甚至是突然 `断电` 导致的错误**
> - **情况二：程序员可以在事务执行过程中手动输入 `ROLLBACK` 语句结束当前事务的执行**
>
> **以上情况出现，我们需要把数据改回原先的样子，这个过程称之为 `回滚` ，这样就可以造成一个假象：这个事务看起来什么都没做，所以符合 `原子性`要求。每当我们要对一条记录做改动时（`INSERT``UPDATE``DELETE`）,都需要把回滚时的记录记下来。比如：**
>
> 
>
> - **当`删除一条记录`时，至少要把这条数据的主键记录下来，之后回滚的时候`删除这条记录`**
> - **当`插入一条记录`时，至少要把这条记录的内容都记下来，之后回滚的时候再把这些记录组成`插入到表中`（==对于每个DELETE，InnoDB存储引擎都会执行一个INSERT==）**
> - **当`修改了一条记录`时，至少要把修改这条记录的旧值记录下来，这样之后回滚时再把这条记录`更新为旧值`（==对于每个UPDATE操作，InnoDB存储引擎会执行一个相反的UPDATE，将修改前的行放回去==）**
>
> **此外，undo log`会产生 redo log` 也就是`undo log`的产生会伴随着`redo log`的产生，这是因为`undo log`也需要持久性的保护**



### 9.2.2 Undo日志的作用

> - **作用1：回滚数据**
>
> > **undo日志是`逻辑日志`，因此只是将数据库逻辑恢复到原来的样子。所有修改都被逻辑地取消了，但是数据结构和页本身在回滚之后可能大不相同。**
>
> - **作用2：MVCC**
>
> > **当用户读取一行记录时，若该记录已经被其他事物占用，当前事务可以通过undo读取之前地行版本信息，以此实现非锁定读取**



### 9.2.3 undo的存储结构

> ​	**InnoDB对undo log的管理采用段的方式，也就是 `回滚段（rollback segment）` 。每个回滚段记录了`1024` 个 `undo log segment` ，而在每个`undo log segment`段中进行 `undo页` 的申请**
>
> - **在 `InnoDB1.1版本之前` （不包括1.1版本），只有一个rollback segment，因此支持同时在线的事务限制为 1024 。虽然对绝大多数的应用来说都已经够用。**
> - **从1.1版本开始InnoDB支持最大 `128个rollback segment` ，故其支持同时在线的事务限制提高到了 `128*1024`** 
>
> ```sql
> show variables like '%innodb_undo_logs%'
> ```







# 十、锁



## 10.1  MySQL并发事务访问相同记录



### 10.1.1 读-读情况

> **`读-读` 情况，即并发事务相继 `读取相同的记录` 。读取操作本身不会对记录有任何影响，并不会引起什么问题，所以允许这种情况的发生。**





### 10.1.2  写-写情况

> ​	**`写-写` 情况，即并发事务相继对相同的记录做出改动。**
>
> ​	**在这种情况下会发生 `脏写` 的问题，任何一种隔离级别都不允许这种问题的发生。所以在多个未提交事务相继对一条记录做改动时，需要让它们 `排队执行`，这个排队的过程其实是通过 `锁` 来实现的。这个所谓的锁其实是一个 内存中的结构 ，在事务执行前本来是没有锁的，也就是说一开始是没有 `锁结构` 和记录进行关联的**
>
> ​	**当一个事务想对这条记录做改动时，首先会看看内存中有没有与这条记录关联的 锁结构 ，当没有的时候就会在内存中生成一个 `锁结构` 与之关联。比如，事务 T1 要对这条记录做改动，就需要生成一个 `锁结构`,如下图所示:**

![](E:\阶段性资料\笔记\pic\QQ截图20221017101929.png)



- **`trx信息`:代表这个所结构是哪个事务生成的**
- **`is_waiting`:代表当前事务是否在等待**

![](E:\阶段性资料\笔记\pic\QQ截图20221017102424.png)

> **总结:**
>
> - **`不加锁`:意思就是不需要在内存中生成对应的 `锁结构` ，可以直接执行操作**
>
>  - **`加锁成功`:意思就是在内存中生成了对应的 `锁结构` ，而且锁结构的 `is_waiting` 属性为 `false` ，也就是事务可以继续执行操作**
>  - **`获取锁失败，或者加锁失败，或者没有获取到锁`:意思就是在内存中生成了对应的 `锁结构` ，不过锁结构的 `is_waiting` 属性为 `true` ，也就是事务
>    需要等待，不可以继续执行操作。**

### 10.1.3 读-写或写-读情况

> ​	**`读-写` 或 `写-读` ，即一个事务进行读取操作，另一个进行改动操作。这种情况下可能发生 `脏读 、 不可重复读 、 幻读 的问题`。**
>
> ​	**各个数据库厂商对 `SQL标准` 的支持都可能不一样。比如MySQL在 `REPEATABLE READ` 隔离级别上就已经解决了 幻读 问题**





### 10.1.4 并发问题的解决方案

- **==方案一==：读操作利用多版本并发控制（MVCC），写操作进行 加锁**

> **普通的SELECT语句在`READ COMMITTED`和`REPEATABLE READ`隔离级别下会使用到`MVCC`读取记录**
>
> - **在 `READ COMMITTED` 隔离级别下，`一个事务在执行过程中每次执行SELECT操作`时都会生成一个`ReadView`，`ReadView`的存在本身就保证了 事务不可以读取到未提交的事务所做的更改 ，也就是避免了脏读现象；**
> - **在 `REPEATABLE READ` 隔离级别下，一个事务在执行过程中只有 `第一次执行SELECT操作`才会生成一个`ReadView`，之后的SELECT操作都 复用 这个ReadView，这样也就避免了不可重复读和幻读的问题。**





- **==方案二==：读、写操作都采用 加锁 的方式**





## 10.2 锁的不同角度分类

![](E:\阶段性资料\笔记\pic\QQ截图20221017163837.png)



### 10.2.1 从数据操作类型划分



> ​	**对于`读-读`这种情况一般不会引起什么问题，对于`写-写`、`读-写`、`写-读`这些情况可能会出现一些问题，因此需要使用`MVCC`和`加锁`的方式来解决。MySQL实现一个由两种类型的锁组成的锁系统来解决，这两种锁通常被叫做==共享锁(Shared Lock 或 S lock)、排他锁(Exclusive Lock 或 X Lock)==也被称作为`读锁(read lock)`和`写锁(write lock)`**
>
> - **`读锁` ：也称为 `共享锁` 、英文用 S 表示。`针对同一份数据，多个事务的读操作可以同时进行而不会互相影响，相互不阻塞的`。**
> - **`写锁` ：也称为 `排他锁` 、英文用 X 表示。`当前写操作没有完成前，它会阻断其他写锁和读锁`。这样就能确保在给定的时间里，只有一个事务能执行写入，并防止其他用户读取正在写入的同一资源**
>
> **注意：对于 InnoDB 引擎来说，`读锁和写锁可以加在表上，也可以加在行上`**
>
> |         | X锁      | S锁      |
> | ------- | -------- | -------- |
> | **X锁** | `不兼容` | `不兼容` |
> | **S锁** | `不兼容` | `兼容`   |
>
> 





**1.锁定读：**

> - **对读取记录加`S锁`**
>
> ```sql
> select ... lock in share mode	
> #或
> select ... for share
> ```
>
> > ​	**在普通`select`语句后面加`lock in share mode`，如果当前事务执行了该语句，那么它会为读取到的记录加`S锁`，这样允许别的事务继续获取这些记录的`S锁`（比如别的事务也使用`select ... lock in share mode`来读取这些记录），但是不能获取这些记录的`X锁`（比如使用`select ... for update`语句来读取这些记录，或者直接修改这些记录）。如果别的事务想要获取这些记录的`X锁`。那么他们会阻塞，直到当前事务提交之后将这些记录上的`S锁`释放掉**
>
> 
>
> - **对读取的记录加`X锁`**
>
> > ​	**在普通`select`语句后面加`for update`，如果当前事务执行了该语句，那么它会为读取到的记录加`X锁`，这样既不许别的事务获取这些记录的`S锁`，也不许获取这些记录的`X锁`。如果别的事务想要获取这些记录的`X锁`或者`S锁`。那么他们会阻塞，直到当前事务提交之后将这些记录上的`X锁`释放掉**



**`MySQL8.0新特性`**

> ​	**在5.7之前的版本，`SELECT...FOR UPDATE`如果获取不到锁，会一直等待，直到`innodb_lock_wait_timeout`超时。在8.0版本中，`select...for update``select...for share`添加==NOWAIT==、==SKIP LOCKED==语法，跳过锁等待**
>
> - **`NOWAIT`会立即报错返回**
> - **`SKIP LOCKED`也会立即返回，`只是返回的结果中不包含被锁定的行`**
>
> ```sql
> SELECT * FROM STUDENT FOR SHARE NOWAIT
> SELECT * FROM STUDENT FOR SHARE SKIP LOCKED
> ```



**2.写操作**

> - **`DELETE`:现在`B+`树中定位到这条记录的位置，然后获取这条记录的`X锁`，再执行`delete mark`操作。可以把这个定位待删除记录在`B+`树中位置的过程看成是一个获取`X锁`的`锁定读`**
>
> - **`UPDATE`:分为三种情况**
>
>   - **`情况一`：未修改该记录的键值，并且被更新的列占用的存储空间在修改前后未发生变化。**
>
>     **现在`B+`树中找到这条记录的位置，然后再获取一下记录的`X锁`，在原记录的位置进行修改操作**
>
>   - **`情况二`：未修改该记录的键值，并且至少有一个被更新的列占用的存储空间再修改前后发生变化**
>
>     **现在`B+`树中定位到这条记录的位置，然后获取一下记录的`X锁`，将该条记录`完全删掉`（移入垃圾链表），最后在插入一条新纪录，这个定位待修改记录再`B+`树位置的过程看成一个获取`X锁`的`锁定读`,新插入的记录由`INSERT`操作提供的`隐式锁`进行保护**
>
>   - **`情况三`：修改了该记录的键值，则相当于在原纪录做`DELETE`操作之后再来一次`INSERT`操作。**
>
> - **`INSERT`:一般情况下不加锁，通过一种`隐式锁`的结构来保护这条新插入的记录在本事务提交前不被别的事务访问**



### 10.2.2 从数据操作的粒度划分：表级锁、页级锁、行锁

> ​	**为了尽可能地提高数据库地并发性能，`每次锁定地数据范围越小越好`，但是管理`锁`是很消耗资源地，因此，数据库需要在`高并发响应`和`系统性能`两方面进行平衡，这样就产生了`锁粒度（Lock granularity）`的概念**



**1.==表锁（Table Lock）==：**

> ​	**表锁会锁定整张表，它是MySQL最基本的锁策略，并`不依赖于存储引擎`,并且表锁是`开销最小`的策略。表锁会一次将整个表锁定，所以可以很好的避免`死锁问题`,但是锁的粒度大也会导致`并发性降低`**
>
> - **表级别的S锁、X锁**
>
> > ​	**在对某个表执行SELECT、INSERT、DELETE、UPDATE语句时，InnoDB存储引擎是不会为这个表添加表级别的 `S锁` 或者 `X锁` 的。在对某个表执行一些诸如 `ALTER TABLE` 、 `DROP TABLE` 这类的 `DDL` 语句时，其他事务对这个表并发执行诸如SELECT、INSERT、DELETE、UPDATE的语句会发生阻塞。同理，某个事务中对某个表执行SELECT、INSERT、DELETE、UPDATE语句时，在其他会话中对这个表执行 `DDL` 语句也会发生阻塞。这个过程其实是通过在 `server层` 使用一种称之为 `元数据锁` （英文名： `Metadata Locks` ，简称 `MDL` ）结构来实现的**
> >
> > ​	**一般情况下，不会使用InnoDB存储引擎提供的表级别的 S锁 和 X锁 。只会在一些特殊情况下，比方说 崩 溃恢复 过程中用到。比如，在系统变量 autocommit=0，innodb_table_locks = 1 时， 手动 获取InnoDB存储引擎提供的表t 的 S锁 或者 X锁 可以这么写：**
> >
> > - **`LOCK TABLES t READ` ：InnoDB存储引擎会对表 `t` 加表级别的 `S锁`**
> > - **`LOCK TABLES t WRITE` ：InnoDB存储引擎会对表 `t` 加表级别的 `X锁`**
> >
> > ```sql
> > show open tables; #查看表上加过的锁
> > #或者
> > show open tables where in_use > 0;
> > ```
> >
> > ```sql
> > # 给表结构加锁
> > lock table temp read;# s锁
> > lock table temp write;# x锁
> > ```
> >
> > ```sql
> > unlock tables;# 解锁
> > ```
> >
> > | 锁类型     | 自己可读 | 自己可写 | 自己可操作其他表 | 他人可读 | 他人可写 |
> > | ---------- | -------- | -------- | ---------------- | -------- | -------- |
> > | **`读锁`** | 是       | 否       | 否               | 是       | 否，等   |
> > | **`写锁`** | 是       | 是       | 否               | 否，等   | 否，等   |
> >
> > **`总结`：MyISAM在执行查询语句（SELECT）前，会给涉及的所有表加`读锁`，在执行增删改操作前，会给涉及的表加写锁，`InnoDB`存储引擎是不会为这个表添加`表级别`的`读锁`或者`写锁`的**



 

**2. ==意向锁 （intention lock）==**



> ​	**InnoDB 支持 `多粒度锁（multiple granularity locking）` ，它允许 `行级锁` 与 `表级锁` 共存，而`意向锁`就是其中的一种`表锁`。意向锁的存在是为了==协调行锁与表锁的关系，支持多粒度（表锁与行锁）的锁并存==。意向锁是一种==不与行级锁冲突的表级锁==，表明`某个事务正在某些行持有了锁或该事务准备去持有锁`**
>
> 
>
> - **`意向共享锁（intention shared lock, IS）`：事务有意向对表中的某些行加`共享锁`（S锁）**
>
> ```sql
> -- 事务要获取某些行的 S 锁，必须先获得表的 IS 锁。
> SELECT column FROM table ... LOCK IN SHARE MODE;
> ```
>
> - **`意向排他锁（intention exclusive lock, IX）`：事务有意向对表中的某些行加`排他锁`（X锁）**
>
> ```sql
> -- 事务要获取某些行的 X 锁，必须先获得表的 IX 锁。
> SELECT column FROM table ... FOR UPDATE;
> ```
>
> > ​	**意向锁是由`存储引擎自己维护的 `，用户`无法手动操作意向锁`，在为数据行加共享 / 排他锁之前，InooDB 会先获取该数据行 所在数据表的对应意向锁。如果某一行被加上了`排他锁`，数据库会自动给更大一级的空间，比如数据页或者数据表加上`意向锁`，表明这个数据页或者数据表已经上过排他锁了**
> >
> > - **如果事务想要获取数据表中某些记录的共享锁，就需要在数据表上`添加意向共享锁`**
> > - **如果事务想要获取数据表中某些记录的排他锁，就需要在数据表上`添加意向排他锁`**
>
> 
>
> **总结：**
>
> 	- **InnoDB 支持 `多粒度锁` ，特定场景下，行级锁可以与表级锁共存**
> 	- **`意向锁之间互不排斥`，但除了 IS 与 S 兼容外， `意向锁会与 共享锁 / 排他锁 互斥`**
> 	- **IX，IS是表级锁，不会和行级的X，S锁发生冲突。只会和表级的X，S发生冲突**
> 	- **意向锁在保证并发性的前提下，实现了 `行锁和表锁共存` 且 `满足事务隔离性` 的要求**





**3.==自增锁（AUTO-INC锁）==**

> 



**4.==元数据锁（MDL锁）==**



> ​	**MySQL5.5引入了meta data lock，简称MDL锁，属于表锁范畴。MDL 的作用是，保证读写的正确性。比如，如果一个查询正在遍历一个表中的数据，而执行期间另一个线程对这个 表结构做变更 ，增加了一列，那么查询线程拿到的结果跟表结构对不上，肯定是不行的。因此，`当对一个表做增删改查操作的时候，加 MDL读锁；当要对表做结构变更操作的时候，加 MDL 写锁`。**





### 10.2.3 InnoDB中的行锁

> ​	**行锁（Row Lock）：MySQL服务层没有实现行锁机制，`行级锁只在存储引擎层实现`**



**数据准备：**

```sql
CREATE TABLE `temp1` (
  `id` int NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  `class` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;
```

| id   | name | class |
| ---- | ---- | ----- |
| `1`  | 张三 | 一班  |
| `3`  | 李四 | 一班  |
| `8`  | 王五 | 一班  |
| `15` | 赵六 | 一班  |
| `20` | 钱七 | 一班  |



**1.==记录锁（Record Locks）==**



![](F:\mysql笔记\MySQL-note\pic\QQ截图20221018134020.png)



> - **当一个事务获取了一条记录的S型记录锁后，其他事务也可以继续获取该记录的S型记录锁，但不可以继续获取X型记录锁**
> - **当一个事务获取了一条记录的X型记录锁后，其他事务既不可以继续获取该记录的S型记录锁，也不可以继续获取X型记录锁**







**2.==间隙锁（Gap Locks）==：**

> ​	**MySQL 在 `REPEATABLE READ` 隔离级别下是可以解决幻读问题的，解决方案有两种，可以使用 `MVCC` 方案解决，也可以采用 `加锁 `方案解决。但是在使用加锁方案解决时有个大问题，就是事务在第一次执行读取操作时，那些幻影记录尚不存在，我们无法给这些 `幻影记录` 加上` 记录锁`。InnoDB提出了一种称之为`Gap Locks` 的锁，官方的类型名称为： LOCK_GAP ，我们可以简称为 `gap锁` 。**





![](F:\mysql笔记\MySQL-note\pic\QQ截图20221018135507.png)



> ​	**图中id值为8的记录加了`gap锁`，意味着 `不允许别的事务在id值为8的记录前边的间隙插入新记录` ，`其实就是id列的值(3, 8)这个区间的新记录是不允许立即插入的`。比如，有另外一个事务再想插入一条id值为4的新记录，它定位到该条新记录的下一条记录的id值为8，而这条记录上又有一个gap锁，所以就会阻塞插入操作，直到拥有这个gap锁的事务提交了之后，id列的值在区间(3, 8)中的新记录才可以被插入。**
>
> ​	**`gap锁`的提出仅仅是为了防止插入幻影记录而提出的`共享gap锁`与`独占gap锁`实质上作用是一样的，间隙锁可能会导致死锁问题**





**3.==临键锁（Next-Key Locks）==**

> ​	**有时候我们既想 `锁住某条记录` ，又想 `阻止` 其他事务在该记录前边的 `间隙插入新记录` ，所以InnoDB就提出了一种称之为 `Next-Key Locks` 的锁，官方的类型名称为：` LOCK_ORDINARY `，我们也可以简称为`next-key`锁 。Next-Key Locks是在存储引擎 `innodb` 、事务级别在 `可重复读` 的情况下使用的数据库锁，innodb默认的锁就是Next-Key locks**
>
> ​	**`实际上就是间隙锁与记录锁的合体将间隙改变成一个闭区间`**
>
> ```sql
> begin;
> select * from student where id <=8 and id > 3 for update;
> ```
>
> ![](F:\mysql笔记\MySQL-note\pic\QQ截图20221018155808.png)
>
> 





### 10.2.4 页锁

> ​	**页锁就是在 页的粒度 上进行锁定，锁定的数据资源比行锁要多，因为一个页中可以有多个行记录。当我们使用页锁的时候，会出现数据浪费的现象，但这样的浪费最多也就是一个页上的数据行。`页锁的开销介于表锁和行锁之间，会出现死锁。锁定粒度介于表锁和行锁之间，并发度一般。`**
>
> ​	**每个层级的锁数量是有限制的，因为锁会占用内存空间， `锁空间的大小是有限的` 。当某个层级的锁数量超过了这个层级的阈值时，就会进行 `锁升级` 。`锁升级就是用更大粒度的锁替代多个更小粒度的锁`，比如InnoDB 中行锁升级为表锁，这样做的好处是占用的锁空间降低了，但同时数据的并发度也下降了**

 



### 10.2.5 从对待锁的态度划分:乐观锁、悲观锁

> **从对待锁的态度来看锁的话，可以将锁分成乐观锁和悲观锁，从名字中也可以看出这两种锁是两种看待数据并发的思维方式 。需要注意的是，乐观锁和悲观锁并不是锁，而是锁的 `设计思想`**



**1.==悲观锁（Pessimistic Locking）==：**

> ​	**悲观锁是一种思想，顾名思义，就是很悲观，对数据被其他事务的修改持保守态度，会通过数据库自身的锁机制来实现，从而保证数据操作的排它性**
> 
>
> ​	**悲观锁总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会 `阻塞` 直到它拿到锁（`共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程`）。比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁，当其他线程想要访问数据时，都需要阻塞挂起。Java中 `synchronized` 和 `ReentrantLock` 等独占锁就是悲观锁思想的实现**



>​	**`select ... for update`就是MySQL中的悲观锁，注意：==select ... for update 语句执行的过程中所有扫描的行都会被锁上，因此在MySQL中使用悲观锁必须确定使用了索引，而不是全表扫描，否则将会把整个表锁住==。悲观锁大多数情况下依靠数据库的锁机制实现，以保证程序的并发性，同时这样对数据库性能开销影响很大，特别是`长事务`而言，这种开销会非常大。**





**2.==乐观锁（Optimistic Locking）==**

> ​	**乐观锁认为对同一数据的并发操作不会总发生，属于小概率事件，不用每次都对数据上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，也就是`不采用数据库自身的锁机制，而是通过程序来实现`。在程序上，我们可以采用 `版本号机制` 或者 `CAS机制` 实现。`乐观锁适用于多读的应用类型，这样可以提高吞吐量`。在Java中 `java.util.concurrent.atomic` 包下的原子变量类就是使用了乐观锁**



>- **乐观锁的`版本号机制`**
>
>> ​	**在表中设计一个 `版本字段 version` ，第一次读的时候，会获取 version 字段的取值。然后对数据进行更新或删除操作时，会执行 `UPDATE ... SET version=version+1 WHERE version=version` 。此时如果已经有事务对这条数据进行了更改，修改就不会成功。**
>
>
>
>- **乐观锁的`时间戳机制`**
>
>> ​	**时间戳和版本号机制一样，也是在更新提交的时候，将当前数据的时间戳和更新之前取得的时间戳进行比较，如果两者一致则更新成功，否则就是版本冲突**
>>
>> ​	**你能看到乐观锁就是程序员自己控制数据并发操作的权限，基本是通过给数据行增加一个戳（版本号或者时间戳），从而证明当前拿到的数据是否最新**



**总结：**

- **乐观锁 适合 `读操作多` 的场景，相对来说写的操作比较少。它的优点在于` 程序实现` ， `不存在死锁问题`，不过适用场景也会相对乐观，因为它阻止不了除了程序以外的数据库操作。**
- **悲观锁 适合 `写操作多` 的场景，因为写的操作具有 `排它性` 。采用悲观锁的方式，可以在数据库层面阻止其他事务对该数据的操作权限，防止 `读 - 写 `和 `写 - 写` 的冲突**





### 10.2.6 按加锁的方式划分：显式锁、隐式锁

**1.`隐式锁`：**

> ​	**一个事务在执行`INSERT`操作时，如果插入的间隙已经被其他事务加了`gap锁`，那么本次`INSERT`操作会阻塞，并且当前事务会在改间隙上加一个`插入意向锁`，否则一般情况下`INSERT`操作是不加锁的。如果一个事务首先插入了一条记录，然后另一个事务：**
>
> - **立即使用`SELECT ... LOCK IN SHARE MODE`语句获取`S锁`，或者使用`SELECT ... FROM UPDATE`语句获取`X锁`；此时如果允许这种情况发生，则会产生`脏读`问题**
> - **立即修改这条记录，也就是获取这条记录的`X锁`；如果允许这种情况发生，则会产生`脏写`问题**
>
> **此时，之前提到的`事务id`就起到作用了：**
>
> 
>
> - **`情景一:`**
>
> > ​	**对于`聚簇索引`记录来说，有一个 `trx_id 隐藏列`，该隐藏列记录着最后改动该记录的 `事务id `。那么如果在当前事务中新`插入一条聚簇索引记录`后，该记录的` trx_id` 隐藏列代表的的就是`当前事务的 事务id` ，如果其他事务此时想对该记录添加 `S锁 或者 X锁 `时，首先会看一下该记录的`trx_id` 隐藏列代表的事务`是否是当前的活跃事务`，如果是的话，那么就帮助当前事务创建一个 `X锁 `（也就是为当前事务创建一个锁结构， is_waiting 属性是 false ），然后`自己进入等待状态`（也就是为自己也创建一个锁结构， is_waiting 属性是 true ）。**
>
> - **`情景二：`**
>
> > ​	**对于二级索引记录来说，本身并没有 `trx_id` 隐藏列，但是在二级索引页面的 `PageHeader` 部分有一个 `PAGE_MAX_TRX_ID` 属性，该属性代表对该页面做改动的`最大的 事务id `，如果` PAGE_MAX_TRX_ID` 属性值`小于当前最小的活跃 事务id` ，那么说明对该页面做修改的事务都已经提交了，否则就需要在页面中定位到对应的二级索引记录，然后回表找到它对应的聚簇索引记录，然后再重复 `情景一` 的做法。**



> **总结：一个事务对新插入的记录可以不显示的加锁，但是由于`事务id`的存在，相当于加了一个隐式锁。别的事务在对这条记录加`S锁`和`X锁`时，由于`隐式锁`的存在，会先帮助当前事务生成一个锁结构，然后自己再生成一个锁结构后进入等待状态。隐式锁是一种`延迟加锁`的机制，从而减少加锁的数量，隐式锁再实际内存对象中`并不含有这种锁信息`==只有当产生锁等待时，隐式锁转化为显示锁==**
>
> ```sql
> SELECT * FROM performance_schema.data_lock_waits #查看锁等待
> ```



**隐式锁的逻辑如下：**



**`A` InnoDB的每条记录中都一个隐含的`trx_id`字段，这个字段存在于聚簇索引的`B+Tree`中。**



**`B` 在操作一条记录前，首先根据记录中的`trx_id`检查该事务`是否是活动的事务(未提交或回滚)`。如果是活动的事务，首先将 `隐式锁` 转换为 `显式锁` (就是为该事务添加一个锁)**



**`C` 检查是否有锁冲突，如果有冲突，创建锁，并设置为`waiting`状态。如果没有冲突不加锁，跳到E**



**`D` 等待加锁成功，被唤醒，或者超时**



**`E` 写数据，并将自己的`trx_id`写入`trx_id`字段**





**2.`显示锁`：**

```sql
select .... lock in share mode
select .... for update
```



### 10.2.7 全局锁

> ​	**全局锁就是对 `整个数据库实例` 加锁。当你需要让整个库处于 `只读状态` 的时候，可以使用这个命令，之后其他线程的以下语句会被阻塞：数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句。全局锁的典型使用 `场景` 是：做 `全库逻辑备份`**



```sql
Flush tables with read lock
```

### 10.2.8 死锁

> **死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环**



| 事务一                                                       | 事务二                                      |
| ------------------------------------------------------------ | ------------------------------------------- |
| **start transaction;<br />update account set money=10 where id=1;** | **start transaction;**                      |
|                                                              | **update account set money=10 where id=2;** |
| **update account set money=20 where id=2;**                  |                                             |
|                                                              | **update account set money=20 where id=1;** |

**1.`产生死锁条件`：**

> **1.两个及其以上的事务**
>
> **2.每个事务都已经持有锁并且申请新的锁**
>
> **3.锁资源同时只能被同一个事务持有或者不兼容**
>
> **4.事物之间因为持有锁和申请锁导致彼此循环等待**





**2.`如何解决死锁`：**



- **方式一：`innodb_lock_wait_timeout`**

> ​	**`等待直到超时`==innodb_lock_wait_timeout==**
>
> ​	**即当两个事务互相等待时，当一个事务等待时间超过设置的阈值时，就将其`回滚`，另外事务继续进行。在`InnoDB`中，参数`innodb_lock_wait_timeout`用来设置超时时间**



- **方式二：`wait-for graph`**

> ​	**`使用死锁检测进行死锁处理`**
>
> ​	**方式一过于被动，InnoDB还提供了`wait-for graph算法`来主动进行死锁检测，每当加锁请求无法立即满足需要并进入等待时，`wait-for graph算法`就会被触发**
>
> 
>
> ​	**这是一种较为主动的死锁检测机制，要求数据库保存`锁的信息链表`,`事务等待链表`两部分信息**	
> ​	![](F:\mysql笔记\MySQL-note\pic\QQ截图20221020160510.png)
>
> ​	**基于如上两个信息可以绘制`wait-for graph`等待图：**
>
> ![](E:\阶段性资料\笔记\pic\QQ截图20221020160943.png)
>
> 
>
> > **`死锁检测的原理是构建一个以事务为顶点，锁为边的有向图，判断有向图是否存在环，存在即有死锁`**
> >
> > **一旦检测到有死锁，InnoDB会选择`回滚undo量最小的事务`，让其他事务继续执行（`innodb_deadlock_detect`控制这个逻辑）**
> >
> > **缺点：时间复杂度为O(n)，如果100个线程同时更新一行意味着要执行1万次**
>
> 
>
>
> **如何避免死锁：**
>
> - **合理设计索引，使业务SQL尽可能通过索引定位更少的行，减少锁竞争**
> - **调整业务逻辑SQL执行顺序，避免`update/delete`长时间持有锁的SQL在事务前面**
> - **避免大事务，尽量将大事务拆成多个小事务，小事务可以缩短锁定资源的时间**
> - **在并发比较高的系统中，不要显式加锁，特别是在事务里显式加锁。比如`select ... for update`语句**
> - **业务允许的情况下可以调低隔离级别**



## 10.3 锁的内部结构

>  	**一条记录加锁的本质就是在内存中创建一个`锁结构`与之关联，理论上创建多个`锁结构`没有问题，但是如果一个事务获取上万条记录的锁资源消耗过大，所以在决定不同记录加锁时，如果符合下边这些条件的记录会放到`锁结构`中**
>
> - **在同一事务中进行加锁操作**
> - **被加锁的记录在同一页面中**
> - **加锁的类型是一样的**
> - **等待状态是一样的**



**`InnoDB`存储引擎的`锁结构`:**

![](E:\阶段性资料\笔记\pic\QQ截图20221021092844.png)



**结构解析：**

> 1. **`锁所在的事务信息` ：**
>
> > ​	**不论是 `表锁 `还是 `行锁` ，都是在事务执行过程中生成的，哪个事务生成了这个 `锁结构` ，这里就记录这个事务的信息**
> >
> > ​	**此 `锁所在的事务信息` 在内存结构中只是一个指针，通过指针可以找到内存中关于该事务的更多信息，比方说`事务id`等。**
>
> 2. **`索引信息` ：**
>
> > ​	**对于 `行锁` 来说，需要记录一下加锁的记录是属于哪个索引的。这里也是一个指针**
>
> 3. **表锁／行锁信息 ：**
>
> > ​	**`表锁结构` 和 `行锁结构` 在这个位置的内容是不同的：**
> >
> > - **表锁：记载着是对哪个表加的锁，还有其他的一些信息**
> > - **行锁：**
> >   - **`Space ID` ：记录所在表空间**
> >   - **`Page Number` ：记录所在页号**
> >   - **`n_bits` ：对于行锁来说，一条记录就对应着一个比特位，一个页面中包含很多记录，用不同的比特位来区分到底是哪一条记录加了锁。为此在行锁结构的末尾放置了一堆比特位，这个`n_bits` 属性代表使用了多少比特位。**
>
> 4. **type_mode ：**
>
> > ​	**这是一个32位的数，被分成了 `lock_mode` 、 `lock_type` 和 `rec_lock_type` 三个部分**
> >
> > ![](E:\阶段性资料\笔记\pic\QQ截图20221021095146.png)
> >
> > 
> >
> > - **锁的模式（ `lock_mode` ），占用低4位，可选的值如下：**
> >
> >   - **`LOCK_IS` （十进制的 0 ）：表示共享意向锁，也就是 IS锁**
> >   - **`LOCK_IX` （十进制的 1 ）：表示独占意向锁，也就是 IX锁**
> >   - **`LOCK_S `（十进制的 2 ）：表示共享锁，也就是 S锁**
> >   - **`LOCK_X` （十进制的 3 ）：表示独占锁，也就是 X锁**
> >   - **`LOCK_AUTO_INC` （十进制的 4 ）：表示 AUTO-INC锁**
> >
> >   **在InnoDB存储引擎中，`LOCK_IS，LOCK_IX，LOCK_AUTO_INC都算是表级锁`的模式，`LOCK_S和LOCK_X既可以算是表级锁的模式，也可以是行级锁的模式`**
> >
> > - **锁的类型（ `lock_type` ），占用第5～8位，不过现阶段只有第5位和第6位被使用**
> >
> >   - **`LOCK_TABLE `（十进制的 16 ），也就是当第5个比特位置为1时，表示表级锁**
> >   - **`LOCK_REC` （十进制的 32 ），也就是当第6个比特位置为1时，表示行级锁**
> >
> > - **行锁的具体类型（ `rec_lock_type` ），使用其余的位来表示。只有在 `lock_type` 的值为`LOCK_REC` 时，也就是只有在该锁为行级锁时，才会被细分为更多的类型：**
> >
> >   - **`LOCK_ORDINARY` （十进制的 0 ）：表示 `next-key`锁**
> >   - **`LOCK_GAP` （十进制的 512 ）：也就是当第10个比特位置为1时，表示 `gap锁`**
> >   - **`LOCK_REC_NOT_GAP` （十进制的 1024 ）：也就是当第11个比特位置为1时，表示正经 `记录锁`**
> >   - **`LOCK_INSERT_INTENTION` （十进制的 2048 ）：也就是当第12个比特位置为1时，表示`插入意向锁`**
> >
> > - **`is_waiting` 属性基于内存空间的节省，所以把 `is_waiting` 属性放到了 `type_mode` 这个32位的数字中**
> >
> >   - **`LOCK_WAIT` （十进制的 `256` ） ：当第9个比特位置为 `1` 时，表示 `is_waiting` 为 `true` ，也就是当前事务尚未获取到锁，处在等待状态；当这个比特位为 `0` 时，表示 `is_waiting` 为`false` ，也就是`当前事务获取锁成功`**
>
> 6. **一堆比特位:**
>
> > ​	**如果是 `行锁结构` 的话，在该结构末尾还放置了一堆比特位，比特位的数量是由上边提到的 n_bits 属性表示的。InnoDB数据页中的每条记录在 记录头信息 中都包含一个 heap_no 属性，伪记录 Infimum 的heap_no 值为 0 ， Supremum 的 heap_no 值为 1 ，之后每插入一条记录， heap_no 值就增1。 锁结构 最后的一堆比特位就对应着一个页面中的记录，一个比特位映射一个 heap_no ，即一个比特位映射到页内的一条记录**



## 10.4 锁监控

> **关于MySQL锁的监控，我们一般可以通过检查 `InnoDB_row_lock` 等状态变量来分析系统上的行锁的争夺情况**
>
> ```sql
> show status like 'innodb_row_lock%';
> ```
>
> | Variables_name                  | Value   |
> | ------------------------------- | ------- |
> | `Innodb_row_lock_current_waits` | `0`     |
> | `Innodb_row_lock_time`          | `84060` |
> | `Innodb_row_lock_time_avg`      | `5253`  |
> | `Innodb_row_lock_time_max`      | `18855` |
> | `Innodb_row_lock_waits`         | `16`    |
>
> - **`Innodb_row_lock_current_waits`：当前正在等待锁定的数量**
> - **`Innodb_row_lock_time` ：从系统启动到现在锁定总时间长度；（等待总时长）**
> - **`Innodb_row_lock_time_avg` ：每次等待所花平均时间；（等待平均时长）**
> - **`Innodb_row_lock_time_max`：从系统启动到现在等待最常的一次所花的时间；**
> - **`Innodb_row_lock_waits` ：系统启动后到现在总共等待的次数；（等待总次数）**



**其他监控方法：**

> ​	**MySQL把事务和锁的信息记录在了 `information_schema` 库中，涉及到的三张表分别是`INNODB_TRX` 、 `INNODB_LOCKS` 和 `INNODB_LOCK_WAITS`**
>
> ​	**`MySQL5.7及之前` ，可以通过`information_schema.INNODB_LOCKS`查看事务的锁情况，但只能看到阻塞事务的锁；如果事务并未被阻塞，则在该表中看不到该事务的锁情况**
>
> ​	**MySQL8.0删除了`information_schema.INNODB_LOCKS`，添加了 `performance_schema.data_locks` ，可以通过==performance_schema.data_locks==查看事务的锁情况，和MySQL5.7及之前不同，`performance_schema.data_locks`不但可以看到阻塞该事务的锁，还可以看到该事务所持有的锁**。
>
> ​	**同时，`information_schema.INNODB_LOCK_WAITS`也被 `performance_schema.data_lock_waits` 所代替**









# 十一、多版本控制并发



## 11.1 MVCC

> ​	**MVCC （`Multiversion Concurrency Control`），多版本`并发控制`。顾名思义，MVCC 是通过数据行的多个版本管理来实现数据库的 并发控制 。这项技术使得在InnoDB的事务隔离级别下执行 一致性读 操作有了保证。换言之，就是为了查询一些正在被另一个事务更新的行，并且可以看到它们被更新之前的值，这样在做查询的时候就不用等待另一个事务释放锁**
>
> ​	**MVCC没有正式的标准，在不同的DBMS中MVCC的实现方式可能是不同的，也不是普遍使用的，MySQL中只有InnoDB实现了MVCC机制。**





## 11.2 快照读与当前读

### 11.2.1 **快照读**

> ​	**快照读又叫一致性读，读取的是快照数据。`不加锁的简单的 SELECT 都属于快照读`，即不加锁的非阻塞读；比如:**
>
> ```sql
> SELECT * FROM player WHERE ...
> ```
>
> ​	**之所以出现快照读的情况，是基于提高并发性能的考虑，快照读的实现是基于MVCC，它在很多情况下，避免了加锁操作，降低了开销**
>
> ​	**既然是基于多版本，那么快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本，快照读的前提是隔离级别不是串行级别，串行级别下的快照读会退化成当前读**



### 11.2.2 当前读

> ​	**当前读读取的是记录的最新版本（最新数据，而不是历史版本的数据），读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。加锁的 SELECT，或者对数据进行增删改都会进行当前读。比如：**
>
> ```sql
> SELECT * FROM student LOCK IN SHARE MODE; # 共享锁
> SELECT * FROM student FOR UPDATE; # 排他锁
> INSERT INTO student values ... # 排他锁
> DELETE FROM student WHERE ... # 排他锁
> UPDATE student SET ... # 排他锁
> ```





## 11.3 隐藏字段、Undo Log版本链

> **对于InnoDB存储引擎而言，它的聚簇索引包含两个必要的隐藏列**
>
> - **`trx_id`:每次一个事务对某条聚簇索引记录进行改动时，都会把该事务的 `事务id` 赋值给`trx_id` 隐藏列**
> - **`roll_pointer`：每次对某条聚簇索引记录进行改动时，都会把旧的版本写入到 `undo`日志 中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息**



![](E:\阶段性资料\笔记\pic\QQ截图20221021135842.png)



> ​	**`insert undo`只在事务回滚时起作用，当事务提交后，该类型的`undo`日志就没用了，它占用的`UndoLog Segment`也会被系统回收（也就是该undo日志占用的Undo页面链表要么被重用，要么被释放）**



**有如下操作流程：**

| 发生时间顺序 |                   事务10                   |                   事务20                   |
| :----------: | :----------------------------------------: | :----------------------------------------: |
|      1       |                   BEGIN;                   |                                            |
|      2       |                                            |                   BEGIN;                   |
|      3       | UPDATE student SET name="李四" WHERE id=1; |                                            |
|      4       | UPDATE student SET name="王五" WHERE id=1; |                                            |
|      5       |                  COMMIT;                   |                                            |
|      6       |                                            | UPDATE student SET name="钱七" WHERE id=1; |
|      7       |                                            | UPDATE student SET name="宋八" WHERE id=1; |
|      8       |                                            |                  COMMIT;                   |



​	**每次对记录进行改动，都会记录一条`undo`日志，每条undo日志也都有一个 `roll_pointer` 属性（ `INSERT `操作对应的undo日志没有该属性，因为该记录并没有更早的版本），可以将这些 `undo日志`都连起来，串成一个链表：**

![](E:\阶段性资料\笔记\pic\QQ截图20221021141059.png)



## 11.4 MVCC实现原理之ReadView

> ​	**在MVCC机制中，多个事务对同一个记录进行更新会产生多个历史快照，这些历史快照保存在`Undo Log`里。如果一个事务想要查询这个行记录，需要读取哪个版本的行记录呢？这个时候就需要`ReadView`来解决行的可见性问题**
>
> ​	**`ReadView`就是`某一个事务`在使用MVCC机制进行`快照读操作`时产生的`读视图`。当事务启动时，会产生数据库系统当前的一个快照，InnoDB为每个事务构造了一个数组，用来记录并维护系统当前`活跃事务`的ID（`“活跃”是指启动了但是没有提交`）**





### 11.4.1 设计思路

> - **使用 `READ UNCOMMITTED` 隔离级别的事务，由于可以读到未提交事务修改过的记录，所以直接读取记录的最新版本就好了**
>
> - **使用 `SERIALIZABLE` 隔离级别的事务，InnoDB规定使用加锁的方式来访问记录**
>
> - **使用 `READ COMMITTED` 和 `REPEATABLE READ` 隔离级别的事务，都必须保证读到 `已经提交了的` 事务修改过的记录。假如另一个事务已经修改了记录但是尚未提交，是不能直接读取最新版本的记录的，`核心问题就是需要判断一下版本链中的哪个版本是当前事务可见的`，这是`ReadView`要解决的主要问题**





**ReadView中主要包含4个比较重要的内容:**



> **1.`creator_trx_id` :创建这个 Read View 的事务 ID**
>
> > ​	**说明：只有在对表中的记录做改动时（执行INSERT、DELETE、UPDATE这些语句时）才会为事务分配事务id，否则在一个`只读事务中的事务id值都默认为0`**
>
> 
>
> **2.`trx_ids` ：表示在生成ReadView时当前系统中活跃的读写事务的 `事务id列表`**
>
> **3.`up_limit_id` :活跃的事务中最小的事务 ID**
>
> **4.`low_limit_id` :表示生成ReadView时系统中应该分配给下一个事务的 `id` 值。low_limit_id 是系统最大的事务`id`值，这里要注意是系统中的事务id，需要区别于正在活跃的事务ID**
>
> > ​	**注意：`low_limit_id`并不是`trx_ids`中的`最大值`，事务id是递增分配的。比如，现在有id为1，2，3这三个事务，之后id为3的事务提交了。那么一个新的读事务在生成`ReadView`时，trx_ids就包括1和2，`up_limit_id`的值就是1，`low_limit_id`的值就是4**



![](E:\阶段性资料\笔记\pic\QQ截图20221024181111.png)



### 11.4.2 ReadView的规则

- **如果被访问版本的`trx_id`属性值与ReadView中的 `creator_trx_id` 值`相同`，意味着当前事务在访问它自己修改过的记录，所以该版本`可以`被当前事务访问**

- **如果被访问版本的`trx_id`属性值`小于`ReadView中的 `up_limit_id` 值，表明生成该版本的事务在当前事务生成`ReadView`前已经提交，所以该版本`可以`被当前事务访问**

- **如果被访问版本的`trx_id`属性值`大于`或等于`ReadView`中的 `low_limit_id` 值，表明生成该版本的事务在当前事务生成`ReadView`后才开启，所以该版本`不可以`被当前事务访问**

- **如果被访问版本的`trx_id`属性值在ReadView的 `up_limit_id` 和 `low_limit_id` `之间`，那就需要判断一下`trx_id`属性值是不是在 `trx_ids` 列表中**

  - **`如果在`，说明创建ReadView时生成该版本的事务还是活跃的，该版本`不可以`被访问**
  - **`如果不在`，说明创建ReadView时生成该版本的事务已经被提交，该版本`可以`被访问**
  
  
  
  

### 11.4.3 MVCC整体操作流程

**1.`首先获取事务自己的版本号，也就是事务 ID`**

**2.`获取 ReadView`**

**3.`查询得到的数据，然后与 ReadView 中的事务版本号进行比较`**

**4.`如果不符合 ReadView 规则，就需要从 Undo Log 中获取历史快照`**

**5.`最后返回符合规则的数据`**

> ​	**如果某个版本的数据对当前事务不可见的话，那就顺着版本链找到下一个版本的数据，继续按照上边的步骤判断可见性，依此类推，知道找到版本链中的最后一个版本。如果最后一个版本也不可见的话，那么就意味着该条记录对该事务完全不可见，查询结果就不包含该记录。**
>
> ​	**InnoDB中，MVCC是通过Undo Log + Read View 进行数据读取的，`Undo Log保存了历史快照`，而`Read View规则帮我们判断当前版本数据是否可见。`**





### 11.4.4 举例

> ​	**假如现在student表中只有一条`事务id`为`8`的事务插入一条记录**
>
> ![](F:\mysql笔记\MySQL-note\pic\QQ截图20221026155928.png)



**MVCC只能在 `READ COMMITTED` 与 `REPEATABLE READ`两个隔离级别下工作:**





**==1.READ COMMITTED隔离级别下：==**

> **==READ COMMITTED ：每次读取数据前都生成一个ReadView==**
>
> **现在有两个 `事务id` 分别为 `10` 、 `20` 的事务在执行：**
>
> ```sql
> # Transaction 10
> BEGIN;
> UPDATE student SET name="李四" WHERE id=1;
> UPDATE student SET name="王五" WHERE id=1;
> # Transaction 20
> BEGIN;
> # 更新了一些别的表的记录
> ...
> ```
>
> - **此刻，表student 中 `id` 为 `1` 的记录得到的版本链表如下所示：**
>
> ![](F:\mysql笔记\MySQL-note\pic\QQ截图20221026160548.png)
>
> 
>
> - **假设现在有一个使用 `READ COMMITTED` 隔离级别的事务开始执行：**
>   - **==步骤1==：在执行`SELECT 1`的时候会生成一个`READ VIEW`这个`READ VIEW`的`trx_ids`列表的内容就是`[10,20]`，`up_limit_id`为`10`,`low_limit_id`为`21`，`creator_trx_id`为`0`**
>   - **==步骤2==：从版本链中挑选可见的记录，从图中看出，最新版本的列`name`的内容是`王五`，该版本的`trx_id`值为`10`，在`trx_ids`列表内，所以不符合可见性要求，根据`roll_pointer`跳到下一个版本**
>   - **==步骤3==：下一个版本的列`name`是`李四`，该版本的`trx_id`的值也为`10`，也在`trx_ids`列表内，所以也不符合要求，继续跳到下一版本**
>   - **==步骤4==：下一个版本的`name`是张三，该版本的`trx_id`值为`8`，小于`ReadView`的`up_limit_id`，所以这个版本是符合要求的，最后返回的就是这个版本`name`为`张三`的记录**
>
> ```sql
> # 使用READ COMMITTED隔离级别的事务
> BEGIN;
> # SELECT1：Transaction 10、20未提交
> SELECT * FROM student WHERE id = 1; # 得到的列name的值为'张三'
> ```
>
> 
>
> - **之后，我们把 `事务id` 为 `10` 的事务提交一下：**
>
> ```sql
> # Transaction 10
> BEGIN;
> UPDATE student SET name="李四" WHERE id=1;
> UPDATE student SET name="王五" WHERE id=1;
> COMMIT;
> ```
>
> - **然后再到 `事务id` 为 `20` 的事务中更新一下表 student 中` id` 为 `1` 的记录**
>
> ```sql
> # Transaction 20
> BEGIN;
> # 更新了一些别的表的记录
> ...
> UPDATE student SET name="钱七" WHERE id=1;
> UPDATE student SET name="宋八" WHERE id=1;
> ```
>
> - **此刻，表student中 `id` 为 `1` 的记录的版本链如图所示：**
>
> ![](F:\mysql笔记\MySQL-note\pic\QQ截图20221026161558.png)
>
> 
>
> - **然后再到刚才使用 `READ COMMITTED` 隔离级别的事务中继续查找这个 `id` 为 `1` 的记录**
>
> 
>
> ```sql
> # 使用READ COMMITTED隔离级别的事务
> 
> BEGIN;
> 
> # SELECT1：Transaction 10、20均未提交
> 
> SELECT * FROM student WHERE id = 1; # 得到的列name的值为'张三'
> 
> # SELECT2：Transaction 10提交，Transaction 20未提交
> 
> SELECT * FROM student WHERE id = 1; # 得到的列name的值为'王五'
> ```
>
> - **==步骤1：==在执行`SELECT`语句时又会单独生成一个`ReadView`，该`ReadView`的`trx_ids`列表的内容就是`[20]`，`up_limit_id`为`20`，`low_limit_id`为`21`，`creator_trx_id`为`0`**
>
> - **==步骤2==：从版本链中挑选可见的记录，从图中看出，最新版本的列`name`的内容是`宋八`，该版本的`trx_id`值为`20`，在`trx_ids`列表内，所以`不符合可见性要求`，根据`roll_pointer`跳到下一版本**
>
> - **==步骤3==：该版本的`name`内容是`钱七`，该版本的`trx_id`值为`20`，也在`trx_ids`列表内，所以也不符合，继续跳到下一个版本**
>
> - **==步骤4==：该版本的`name`内容是`王五`，`trx_id`值为`10`，小于`ReadView`中的`up_limit_id`，所以这个版本是`符合要求`的**
>
>   **以此类推，如果`事务id`为`20`的记录也`提交了`，再次使用`READ COMMITED`隔离级别的食物中查询表student中`id`为`1`的记录，得到的结果就是`宋八`**







------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------







**==2.REPEATABLE READ隔离级别下==**

> ​	**使用 `REPEATABLE READ` 隔离级别的事务来说，==只会在第一次执行查询语句时生成一个 `ReadView`== ，之后的查询就不会重复生成了**
>
> 
>
> - **现在有两个 `事务id` 分别为 `10` 、 `20` 的事务在执行**
>
> ```sql
> # Transaction 10
> BEGIN;
> UPDATE student SET name="李四" WHERE id=1;
> UPDATE student SET name="王五" WHERE id=1;
> # Transaction 20
> BEGIN;
> # 更新了一些别的表的记录
> ...
> ```
>
> - **此刻，表student 中 `id 为 `1` 的记录得到的版本链表如下所示**
>
> ![](F:\mysql笔记\MySQL-note\pic\QQ截图20221026162303.png)
>
> 
>
> - **假设现在有一个使用 `REPEATABLE READ` 隔离级别的事务开始执行：**
>   - **==步骤1==：在执行`SELECT 1`的时候会生成一个`READ VIEW`这个`READ VIEW`的`trx_ids`列表的内容就是`[10,20]`，`up_limit_id`为`10`,`low_limit_id`为`21`，`creator_trx_id`为`0`**
>   - **==步骤2==：从版本链中挑选可见的记录，从图中看出，最新版本的列`name`的内容是`王五`，该版本的`trx_id`值为`10`，在`trx_ids`列表内，所以不符合可见性要求，根据`roll_pointer`跳到下一个版本**
>   - **==步骤3==：下一个版本的列`name`是`李四`，该版本的`trx_id`的值也为`10`，也在`trx_ids`列表内，所以也不符合要求，继续跳到下一版本**
>   - **==步骤4==：下一个版本的`name`是张三，该版本的`trx_id`值为`8`，小于`ReadView`的`up_limit_id`，所以这个版本是符合要求的，最后返回的就是这个版本`name`为`张三`的记录**
>
> ```sql
> # 使用REPEATABLE READ隔离级别的事务
> BEGIN;
> # SELECT1：Transaction 10、20未提交
> SELECT * FROM student WHERE id = 1; # 得到的列name的值为'张三'
> ```
>
> - **我们把 `事务id` 为 `10` 的事务`提交`一下：**
>
> ```sql
> # Transaction 10
> BEGIN;
> UPDATE student SET name="李四" WHERE id=1;
> UPDATE student SET name="王五" WHERE id=1;
> COMMIT;
> ```
>
> - **然后再到 事务id 为 `20` 的事务中`更新`一下表 student 中 id 为 `1` 的记录**
>
> ```sql
> # Transaction 20
> BEGIN;
> # 更新了一些别的表的记录
> ...
> UPDATE student SET name="钱七" WHERE id=1;
> UPDATE student SET name="宋八" WHERE id=1;
> ```
>
> - **此刻，表student中 id 为 `1` 的记录的版本链如图所示：**
>
> ![](F:\mysql笔记\MySQL-note\pic\QQ截图20221026162601.png)
>
> 
>
> - **然后再到刚才使用 `REPEATABLE READ` 隔离级别的事务中继续查找这个 `id `为 `1 `的记录**
>   - **==步骤1==：当前事务隔离级别是`REPEATABLE READ`，而之前执行`SELECT1`的时候已经产生`ReadView`了，所以现在会复用之前的`ReadView`，之前的`ReadView`的`trx_ids`列表内容就是`[10,20]`，`up_limit_id`为`10`，`low_limit_id`为`21`，`creator_trx_id`为`0`**
>
> ```sql
> # 使用REPEATABLE READ隔离级别的事务
> BEGIN;
> # SELECT1：Transaction 10、20均未提交
> SELECT * FROM student WHERE id = 1; # 得到的列name的值为'张三'
> # SELECT2：Transaction 10提交，Transaction 20未提交
> SELECT * FROM student WHERE id = 1; # 得到的列name的值仍为'张三
> ```
>
> 
> 















































