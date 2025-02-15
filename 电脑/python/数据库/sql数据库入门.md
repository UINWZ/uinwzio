<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline3">1. 前言</a>
<ul>
<li><a href="#orgheadline1">1.1. 基本术语</a></li>
<li><a href="#orgheadline2">1.2. 安装sqlite3</a></li>
</ul>
</li>
<li><a href="#orgheadline21">2. 第一个例子</a>
<ul>
<li><a href="#orgheadline4">2.1. 新建一个表格</a></li>
<li><a href="#orgheadline5">2.2. 字段数据类型</a></li>
<li><a href="#orgheadline9">2.3. 字段约束词</a>
<ul>
<li><a href="#orgheadline6">2.3.1. PRIMARY KEY</a></li>
<li><a href="#orgheadline7">2.3.2. NOT NULL</a></li>
<li><a href="#orgheadline8">2.3.3. DEFAULT</a></li>
</ul>
</li>
<li><a href="#orgheadline10">2.4. 更改表格属性</a></li>
<li><a href="#orgheadline11">2.5. 删除表格</a></li>
<li><a href="#orgheadline12">2.6. 插入记录</a></li>
<li><a href="#orgheadline13">2.7. 删除记录</a></li>
<li><a href="#orgheadline14">2.8. 查看当前日期</a></li>
<li><a href="#orgheadline15">2.9. 检索单个列</a></li>
<li><a href="#orgheadline16">2.10. 检索多个列</a></li>
<li><a href="#orgheadline17">2.11. 排序</a></li>
<li><a href="#orgheadline18">2.12. 按多个列排序</a></li>
<li><a href="#orgheadline19">2.13. 降序排序</a></li>
<li><a href="#orgheadline20">2.14. 过滤数据</a></li>
</ul>
</li>
<li><a href="#orgheadline29">3. 第二个例子</a>
<ul>
<li><a href="#orgheadline22">3.1. 创建表格</a></li>
<li><a href="#orgheadline23">3.2. 外键引用</a></li>
<li><a href="#orgheadline24">3.3. 主键声明的另一方法</a></li>
<li><a href="#orgheadline25">3.4. 其他sql语句类型声明有时不修改也可</a></li>
<li><a href="#orgheadline26">3.5. check约束模拟mysql的enum类型</a></li>
<li><a href="#orgheadline27">3.6. 插入数据</a></li>
<li><a href="#orgheadline28">3.7. 演示例子的补充信息</a></li>
</ul>
</li>
<li><a href="#orgheadline30">4. 简单的SQL查询</a></li>
<li><a href="#orgheadline32">5. 联接</a>
<ul>
<li><a href="#orgheadline31">5.1. 内部联接</a></li>
</ul>
</li>
<li><a href="#orgheadline36">6. 视图</a>
<ul>
<li><a href="#orgheadline33">6.1. 新建视图</a></li>
<li><a href="#orgheadline34">6.2. 删除视图</a></li>
<li><a href="#orgheadline35">6.3. 更新视图</a></li>
</ul>
</li>
<li><a href="#orgheadline38">7. 附录</a>
<ul>
<li><a href="#orgheadline37">7.1. 参考资料</a></li>
</ul>
</li>
</ul>
</div>
</nav>


# 前言<a id="orgheadline3"></a>

本文主要介绍SQL数据库的最核心的基础知识，用sqlite3作为实验对象。postgresql和mysql则另外再撰文分析。

## 基本术语<a id="orgheadline1"></a>

数据库（database）简单的理解就是一个文件柜。然后DBMS（database management system）是数据库管理系统。而所谓的 **table** 表你可以将其看作文件柜里的一个结构化了的文件。在表格里面的数据都是 <span class="underline">按行存储</span> 的，这个要在头脑中牢记。然后一行数据称之为一个 **记录** 。SQL（Structured Query Language）的意思是结构化查询语言。

## 安装sqlite3<a id="orgheadline2"></a>

在ubuntu下可以简单的用apt-get命令来安装：

```sh
sudo apt-get install sqlite3
```

sqlite3的配置没什么好说的，如果你在终端输入 `sqlite3` ，应该就能够进去了。

# 第一个例子<a id="orgheadline21"></a>

sqlite3和postgresql不同，其没有客户端/服务器的概念，就是直接的文件管理，sqlite3的数据库就是一个文件，然后如果连接到某个不存在的文件，就会自动创建对应的数据库文件。删除数据库当然就是删除数据库文件即可，这一块sqlite3很简单，比如说你如下运行：

```sh
sqlite3 mydb
```

现在我们往这个数据库里面加入一点东西:

    wanze@wanze-ubuntu64:~/桌面$ sqlite3 mydb
    SQLite version 3.8.2 2013-12-06 14:53:30
    Enter ".help" for instructions
    Enter SQL statements terminated with a ";"
    sqlite> create table mytable(id integer primary key, name text);
    sqlite> insert into mytable(id,name) values(1,"Micheal");
    sqlite> select * from mytable;
    1|Micheal
    sqlite> .table
    mytable
    sqlite> .database
    seq  name             file                                                      
    ---  ---------------  ----------------------------------------------------------
    0    main             /home/wanze/桌面/mydb                                   
    sqlite> .header on 
    sqlite> select * from mytable;
    id|name
    1|Micheal

这个例子首先连接之前创建的mydb数据库，然后通过CREATE TABLE语句来创建database数据库里面的一个表，接下来用INSERT INTO语句来给某个表插入一些数据，然后用SELECT语句来查看这些数据。

然后 `.database` 列出当前连接的数据库信息， `.table` 列出当前表格的信息， `.header on` 显示SQL表格头， `.quit` 退出sqlite3命令行， 你还可以通过 `.help` 来查看更多相关信息。

## 新建一个表格<a id="orgheadline4"></a>

如下所示就是创建表格的命令格式:

    CREATE TABLE mytable(
      id     INTEGER PRIMARY KEY, 
      name   TEXT);

这里的mytable就是具体创建的表格名字，然后接下来每一行定义了具体表格的某一个字段或者说某一列。第一个是字段的名字，第二个是字段的数据类型定义，后面可选的还可以跟上其他一些约束词。下面先就字段的数据类型做出一些说明。

## 字段数据类型<a id="orgheadline5"></a>

按照sqlite3官方文档的 [介绍](https://www.sqlite.org/datatype3.html) ，其就支持五种数据类型：

-   **NULL:** 空值
-   **INTEGER:** 整型
-   **REAL:** 浮点型
-   **TEXT:** 字符串型
-   **BLOB:** 字节流型

然后sqlite3关于你的类型声明字符串还建立了一套语法糖规则，具体语法糖规则如下所示:

1.  如果没有类型声明，则视为none affinity;
2.  如果在声明字符串中看到"int"（不区分大小写）这个子字符串，那么就视为integer affinity；
3.  继续，接下来如果找到"char"或者"clob"或者"text"，则视为text affinity；所以varchar(80)会被简单视为text affinity;
4.  继续，接下来如果找到"blob"，则视为blob，如果没数据类型声明，则视为none affinity;
5.  继续，接下来如果找到"real"或者"floa"或者"doub"，则视为float affinity;
6.  然后其他的都视为numeric affinity。

这里什么affinity是sqlite特有的概念，比如text affinity和前面的内置TEXT字符串型是不同的，其可能对应的是NULL，TEXT或BLOB。然后如果输入的数值则会自动将其转换成为字符串型。然后NUMERIC会自动分配成为INTEGER或REAL型，如果输入的字符串还负责转化。等等，总之我们在心里知道sqlite3在类型声明上是很灵活的就行了，具体使用还是严格按照自己喜欢的一套类型声明即可，比如这五个: **null int float text blob** 或者 **null integer float varchar blob** 等等。

## 字段约束词<a id="orgheadline9"></a>

请看下面这个例子:

    sqlite> CREATE TABLE products(
       ...>   id  int PRIMARY KEY,
       ...>   name text NOT NULL,
       ...>   quantity  int NOT NULL DEFAULT 1,
       ...>   price real NOT NULL);

### PRIMARY KEY<a id="orgheadline6"></a>

一个表格只能有一个PRIMARY KEY，被PRIMARY KEY约束的字段值必须唯一且不为空，从而使其能够成为本表格中各个记录的唯一标识。表格中可以有一个列或者几个列被选定为primary key。值得一提的是 `integer primary key` 自动有了自动分配的属性，也就是大家清楚的id那一列，即使不赋值，也会自动添加。

### NOT NULL<a id="orgheadline7"></a>

约束该字段不可取空值，也就是该字段必须赋值的意思。

### DEFAULT<a id="orgheadline8"></a>

指定该字段的默认值。

## 更改表格属性<a id="orgheadline10"></a>

sqlite3更改表格属性的功能是很有限的，就只有两个，一个是更改表格名字，还一个是新建一个字段。所以sqlite3主要还是靠CREATE TABLE的时候就把各个字段属性设置好。具体使用就是使用 `ALTER TABLE` 语句，如下所示:

    ALTER TABLE tablename RENAME TO new_tablename;
    ALTER TABLE tablename ADD COLUMN column_name column_datatype;

## 删除表格<a id="orgheadline11"></a>

删除前请慎重。

    DROP TABLE tablename;

## 插入记录<a id="orgheadline12"></a>

插入一条记录如前所示使用 `INSERT INTO` 语句，具体如下所示：

    INSERT INTO products (product_no, name, price) VALUES (1, ’Cheese’, 9.99);

这是推荐的风格，后面圆括号跟着的列名不一定是按照顺序来，只是和后面的值一一对应，而且一条记录里面的所有列也不需要都列出来，不写的会按照默认值来处理。INSERT INTO语句是通用的。

如果只是需要简单插入新的一行的数据那么可以直接使用之前的insert into语句。

    sqlite> insert into mytable(age) values(6);
    sqlite> select * from mytable;
    id          name        age
    ----------  ----------  ----------
    1           Alice
    2           Betty
    3           Cassie
    4           Doris
    5           Emily
    6           Abby
    7           Bella
    8                       6

不过这并不满足这里的要求，下面的语句用于更新某个特定的表格的数据：

    sqlite> update mytable set age=18 where id =1;
    sqlite> select * from mytable;
    id          name        age
    ----------  ----------  ----------
    1           Alice       18
    2           Betty
    3           Cassie
    4           Doris
    5           Emily
    6           Abby
    7           Bella
    8                       6

## 删除记录<a id="orgheadline13"></a>

    DELETE FROM products WHERE price = 10;
    DELETE FROM products;

注意：第二个语句表内所有记录都将被删除！ DELETE FROM 语句是通用的。

## 查看当前日期<a id="orgheadline14"></a>

    sqlite> SELECT current_date;
    2015-06-16

## 检索单个列<a id="orgheadline15"></a>

为了后面讲解方便，这里根据前面的第一个例子简单创建一个数据库，具体代码如下：

    ~$ sqlite3 test.db
    
    sqlite> CREATE TABLE mytable(id INTEGER PRIMARY KEY, name TEXT);
    sqlite> INSERT INTO mytable(id,name) values(1,'Alice');
    sqlite> INSERT INTO mytable(id,name) values(2,'Betty');
    sqlite> INSERT INTO mytable(id,name) values(3,'Cassie');
    sqlite> INSERT INTO mytable(id,name) values(4,'Doris');
    sqlite> INSERT INTO mytable(id,name) values(5,'Emily');
    sqlite> .header on
    sqlite> .mode column
    sqlite> SELECT * FROM mytable;
    id          name
    ----------  ----------
    1           Alice
    2           Betty
    3           Cassie
    4           Doris
    5           Emily

检索单个列的语法如下：

    sqlite> SELECT id FROM mytable;
    id
    ----------
    1
    2
    3
    4
    5

## 检索多个列<a id="orgheadline16"></a>

多个列就是写上多个列名，列名之间用逗号隔开。检索所有列就是列名用通配符"\*"来匹配所有列。

## 排序<a id="orgheadline17"></a>

用SELECT语句默认情况是没有排序的，如果需要排序则需要使用ORDER BY字句。现在又插入几个新名字：

    sqlite> INSERT INTO mytable(id,name) values(6,'Abby');
    sqlite> INSERT INTO mytable(id,name) values(7,'Bella');

如果我们需要输入结果按照name来排序则：

    sqlite> SELECT id,name FROM mytable ORDER BY name;
    id          name
    ----------  ----------
    6           Abby
    1           Alice
    7           Bella
    2           Betty
    3           Cassie
    4           Doris
    5           Emily

## 按多个列排序<a id="orgheadline18"></a>

首先更新之前的表格的一些数据。

    sqlite> ALTER TABLE mytable ADD COLUMN age int;
    sqlite> UPDATE mytable set age=18 WHERE id=1;
    sqlite> update mytable set age=20 where id =2;
    sqlite> update mytable set age=6 where id =3;
    sqlite> update mytable set age=25 where id =4;
    sqlite> update mytable set age=30 where id =5;
    sqlite> update mytable set age=66 where id =6;
    sqlite> update mytable set age=20 where id =7;
    sqlite> INSERT INTO mytable(id,name,age) values(8,'Alice',20);
    sqlite> SELECT * FROM mytable;
    id          name        age
    ----------  ----------  ----------
    1           Alice       18
    2           Betty       20
    3           Cassie      6
    4           Doris       25
    5           Emily       30
    6           Abby        66
    7           Bella       20
    8           Alice       20

现在开始按多个列排序：

    sqlite> SELECT * FROM mytable ORDER BY name , age;
    id          name        age
    ----------  ----------  ----------
    6           Abby        66
    1           Alice       18
    8           Alice       20
    7           Bella       20
    2           Betty       20
    3           Cassie      6
    4           Doris       25
    5           Emily       30

我们看到首先按name排序，如果名字相同则按age排序。

## 降序排序<a id="orgheadline19"></a>

ORDER BY字句默认排序是升序，如果想要其为降序则使用DESC关键词，如下所示:

    sqlite> SELECT * FROM mytable ORDER BY name , age DESC;
    id          name        age
    ----------  ----------  ----------
    6           Abby        66
    8           Alice       20
    1           Alice       18
    7           Bella       20
    2           Betty       20
    3           Cassie      6
    4           Doris       25
    5           Emily       30

DESC关键词要放在想要降序排序的列的后面。如果想要多个列降序排序，则那些列 <span class="underline">都要加上DESC关键词</span> 。

## 过滤数据<a id="orgheadline20"></a>

SQL用WHERE字句来达到查询时过滤数据的功能，如果同时有WHERE字句和ORDER BY字句，则ORDER BY字句要放在后面。

一个简单的例子如下所示:

    sqlite> SELECT * FROM mytable WHERE age < 30;
    id          name        age       
    ----------  ----------  ----------
    1           Alice       18        
    2           Betty       20        
    3           Cassie      6         
    4           Doris       25        
    7           Bella       20        
    8           Alice       20

如果后面的值是字符串则需要加上单引号，如: 'string' ，这些比较符号的含义都是一目了然的，如: = , <> , != , < , <= , !< , > , >= ,!> 。此外还有 **BETWEEN** 在两个值之间， **IS NULL** 为NULL值。

BETWEEN的使用如下:

    SELECT prod_name, prod_price
    FROM Products
    WHERE prod_price BETWEEN 5 AND 10;

NULL 无值，它与字段包含0，空字符串等不同。 **IS NULL** 用法如下:

    SELECT prod_name
    FROM Products
    WHERE prod_price IS NULL;

# 第二个例子<a id="orgheadline29"></a>

第二个例子将更加大型和更具现实意义。我们将通过创建一个sql文件，刷入sqlite3命令的形式来创建这个数据库。

```bash
sqlite3 sqlite3_learning_example.db &lt; sqlite3_learning_example.sql
```

## 创建表格<a id="orgheadline22"></a>

```sqlite3
create table if not exists department
 (dept_id int primary key,
  name varchar not null
 );
```

sqlite3的 `int primary key` 一起是个约束词，这样写就不用写not null或者autoincrement了，然后sqlite3和mysql不一样，其约束词就是直接跟在列名后面的。

```sqlite3
create table if not exists branch
 (branch_id int  primary key,
  name varchar not null,
  address varchar,
  city varchar,
  state varchar,
  zip varchar
 );
```

## 外键引用<a id="orgheadline23"></a>

外键引用在sqlite3中必须加上这一行设置:

    PRAGMA foreign_keys = ON;

然后后面加入外键引用的语法和mysql有点类似，除了没有constraint pk\_product\_type这一描述外。然后sqlite3里面并没有 **date** 类型的，其内部会自动处理为text,int或real等类型。但我们在声明的时候还是可以这样写的，然后值得一提的是sqlite3里面有一些date相关函数支持。

关于外键引用更多信息请参看官方文档的 [这里](https://www.sqlite.org/foreignkeys.html) 。

```sqlite3
PRAGMA foreign_keys = ON;

create table if not exists employee
 (emp_id int  primary key,
  fname varchar not null,
  lname varchar not null,
  start_date date not null,
  end_date date,
  superior_emp_id int ,
  dept_id int ,
  title varchar,
  assigned_branch_id int ,
    foreign key (superior_emp_id) references employee (emp_id),
    foreign key (dept_id) references department (dept_id),
    foreign key (assigned_branch_id) references branch (branch_id)
 );
```

## 主键声明的另一方法<a id="orgheadline24"></a>

这里演示了主键声明的另一方法，和mysql语句很接近了。

```sqlite3
create table if not exists product_type
 (product_type_cd varchar not null,
  name varchar not null,
  primary key (product_type_cd)
 );
```

## 其他sql语句类型声明有时不修改也可<a id="orgheadline25"></a>

比如在下面语句中 `varchar(10)` 这种写法是mysql里面的，我们在这里没有将其修改为 `varchar` 或 `text` ，因为前面说过sqlite3有很灵活的语法糖规则，直接输入 `varchar(10)` 和改成 `varchar` 没有本质区别，所以我们在这里可以图省心就不修改了。

```sqlite3
create table if not exists product
(product_cd varchar(10) not null,
 name varchar(50) not null,
 product_type_cd varchar(10) not null,
 date_offered date,
 date_retired date,
 foreign key (product_type_cd)
   references product_type (product_type_cd),
  primary key (product_cd)
);
```

## check约束模拟mysql的enum类型<a id="orgheadline26"></a>

mysql提供了enum类型，在sqlite3中没有，要达到类似的效果只有如下用 **check** 约束。 参考了 [这个网页](http://stackoverflow.com/questions/5299267/how-to-create-enum-type-in-sqlite) 。然后这里mysql的integer类型声明也没改了，等同于int。

```sqlite3
create table if not exists customer
(cust_id integer  not null ,
 fed_id varchar(12) not null,
 cust_type_cd  not null check(cust_type_cd in ('I','B')),
 address varchar(30),
 city varchar(20),
 state varchar(20),
 postal_code varchar(10),
   primary key (cust_id)
);
```

然后我们继续:

```sqlite3
create table if not exists individual
(cust_id integer  not null,
 fname varchar(30) not null,
 lname varchar(30) not null,
 birth_date date,
  foreign key (cust_id)
   references customer (cust_id),
  primary key (cust_id)
);

create table if not exists business
(cust_id integer  not null,
 name varchar(40) not null,
 state_id varchar(10) not null,
 incorp_date date,
  foreign key (cust_id)
   references customer (cust_id),
  primary key (cust_id)
);


create table if not exists officer
(officer_id smallint  not null ,
 cust_id integer  not null,
 fname varchar(30) not null,
 lname varchar(30) not null,
 title varchar(20),
 start_date date not null,
 end_date date,
  foreign key (cust_id)
   references business (cust_id),
  primary key (officer_id)
);

create table if not exists account
(account_id integer  not null ,
 product_cd varchar(10) not null,
 cust_id integer  not null,
 open_date date not null,
 close_date date,
 last_activity_date date,
 status check(status in ('ACTIVE','CLOSED','FROZEN')),
 open_branch_id smallint ,
 open_emp_id smallint ,
 avail_balance float(10,2),
 pending_balance float(10,2),
  foreign key (product_cd)
   references product (product_cd),
  foreign key (cust_id)
   references customer (cust_id),
  foreign key (open_branch_id)
   references branch (branch_id),
  foreign key (open_emp_id)
   references employee (emp_id),
  primary key (account_id)
);

create table if not exists transaction_t
(txn_id integer not null ,
 txn_date datetime not null,
 account_id integer  not null,
 txn_type_cd check(txn_type_cd in ('DBT','CDT')),
 amount float not null,
 teller_emp_id smallint ,
 execution_branch_id smallint ,
 funds_avail_date datetime,
  foreign key (account_id)
   references account (account_id),
  foreign key (teller_emp_id)
   references employee (emp_id),
  foreign key (execution_branch_id)
   references branch (branch_id),
 primary key (txn_id)
);
```

这里唯一要讲的就是最后一个表格的名字，原来在mysql中为transaction，这和sqlite3里的关键词transaction冲突了，这里稍作修改了。

## 插入数据<a id="orgheadline27"></a>

在mysql中有如下语句:

```mysql
insert ignore into department (dept_id, name)
values (1, 'Operations');
insert ignore into department (dept_id, name)
values (2, 'Loans');
insert ignore into department (dept_id, name)
values (3, 'Administration');
```

其实现了插入，如果主键值重复则ignore的逻辑。

在sqlite3中有类似的这样的表达:

```sqlite3
insert or ignore into department (dept_id, name)
values (1, 'Operations');
insert or ignore into department (dept_id, name)
values (2, 'Loans');
insert or ignore into department (dept_id, name)
values (3, 'Administration');
```

也就是把ignore替换为 **or ignore** 即可。

然后mysql语句一样做如此修改之后，就没有什么额外要修改的了。因为SQL系insert，select和update这几个语句他们都是非常非常的接近的。如下所示这些都不用做任何修改:

```sqlite3
insert or ignore  into branch (branch_id, name, address, city, state, zip)
values (1, 'Headquarters', '3882 Main St.', 'Waltham', 'MA', '02451');
insert or ignore  into branch (branch_id, name, address, city, state, zip)
values (2, 'Woburn Branch', '422 Maple St.', 'Woburn', 'MA', '01801');
insert or ignore  into branch (branch_id, name, address, city, state, zip)
values (3, 'Quincy Branch', '125 Presidential Way', 'Quincy', 'MA', '02169');
insert or ignore  into branch (branch_id, name, address, city, state, zip)
values (4, 'So. NH Branch', '378 Maynard Ln.', 'Salem', 'NH', '03079');

/* employee data */
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (1, 'Michael', 'Smith', '2001-06-22',
  (select dept_id from department where name = 'Administration'),
  'President',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (2, 'Susan', 'Barker', '2002-09-12',
  (select dept_id from department where name = 'Administration'),
  'Vice President',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (3, 'Robert', 'Tyler', '2000-02-09',
  (select dept_id from department where name = 'Administration'),
  'Treasurer',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (4, 'Susan', 'Hawthorne', '2002-04-24',
  (select dept_id from department where name = 'Operations'),
  'Operations Manager',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (5, 'John', 'Gooding', '2003-11-14',
  (select dept_id from department where name = 'Loans'),
  'Loan Manager',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (6, 'Helen', 'Fleming', '2004-03-17',
  (select dept_id from department where name = 'Operations'),
  'Head Teller',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (7, 'Chris', 'Tucker', '2004-09-15',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (8, 'Sarah', 'Parker', '2002-12-02',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (9, 'Jane', 'Grossman', '2002-05-03',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'Headquarters'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (10, 'Paula', 'Roberts', '2002-07-27',
  (select dept_id from department where name = 'Operations'),
  'Head Teller',
  (select branch_id from branch where name = 'Woburn Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (11, 'Thomas', 'Ziegler', '2000-10-23',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'Woburn Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (12, 'Samantha', 'Jameson', '2003-01-08',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'Woburn Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (13, 'John', 'Blake', '2000-05-11',
  (select dept_id from department where name = 'Operations'),
  'Head Teller',
  (select branch_id from branch where name = 'Quincy Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (14, 'Cindy', 'Mason', '2002-08-09',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'Quincy Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (15, 'Frank', 'Portman', '2003-04-01',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'Quincy Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (16, 'Theresa', 'Markham', '2001-03-15',
  (select dept_id from department where name = 'Operations'),
  'Head Teller',
  (select branch_id from branch where name = 'So. NH Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (17, 'Beth', 'Fowler', '2002-06-29',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'So. NH Branch'));
insert or ignore  into employee (emp_id, fname, lname, start_date,
  dept_id, title, assigned_branch_id)
values (18, 'Rick', 'Tulman', '2002-12-12',
  (select dept_id from department where name = 'Operations'),
  'Teller',
  (select branch_id from branch where name = 'So. NH Branch'));


/* create data for self-referencing foreign key 'superior_emp_id' */
create temporary table emp_tmp as
select emp_id, fname, lname from employee;

update employee set superior_emp_id =
 (select emp_id from emp_tmp where lname = 'Smith' and fname = 'Michael')
where ((lname = 'Barker' and fname = 'Susan')
  or (lname = 'Tyler' and fname = 'Robert'));
update employee set superior_emp_id =
 (select emp_id from emp_tmp where lname = 'Tyler' and fname = 'Robert')
where lname = 'Hawthorne' and fname = 'Susan';
update employee set superior_emp_id =
 (select emp_id from emp_tmp where lname = 'Hawthorne' and fname = 'Susan')
where ((lname = 'Gooding' and fname = 'John')
  or (lname = 'Fleming' and fname = 'Helen')
  or (lname = 'Roberts' and fname = 'Paula')
  or (lname = 'Blake' and fname = 'John')
  or (lname = 'Markham' and fname = 'Theresa'));
update employee set superior_emp_id =
 (select emp_id from emp_tmp where lname = 'Fleming' and fname = 'Helen')
where ((lname = 'Tucker' and fname = 'Chris')
  or (lname = 'Parker' and fname = 'Sarah')
  or (lname = 'Grossman' and fname = 'Jane'));
update employee set superior_emp_id =
 (select emp_id from emp_tmp where lname = 'Roberts' and fname = 'Paula')
where ((lname = 'Ziegler' and fname = 'Thomas')
  or (lname = 'Jameson' and fname = 'Samantha'));
update employee set superior_emp_id =
 (select emp_id from emp_tmp where lname = 'Blake' and fname = 'John')
where ((lname = 'Mason' and fname = 'Cindy')
  or (lname = 'Portman' and fname = 'Frank'));
update employee set superior_emp_id =
 (select emp_id from emp_tmp where lname = 'Markham' and fname = 'Theresa')
where ((lname = 'Fowler' and fname = 'Beth')
  or (lname = 'Tulman' and fname = 'Rick'));

drop table emp_tmp;


/* product type data */
insert or ignore  into product_type (product_type_cd, name)
values ('ACCOUNT','Customer Accounts');
insert or ignore  into product_type (product_type_cd, name)
values ('LOAN','Individual and Business Loans');
insert or ignore  into product_type (product_type_cd, name)
values ('INSURANCE','Insurance Offerings');


/* product data */
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('CHK','checking account','ACCOUNT','2000-01-01');
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('SAV','savings account','ACCOUNT','2000-01-01');
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('MM','money market account','ACCOUNT','2000-01-01');
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('CD','certificate of deposit','ACCOUNT','2000-01-01');
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('MRT','home mortgage','LOAN','2000-01-01');
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('AUT','auto loan','LOAN','2000-01-01');
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('BUS','business line of credit','LOAN','2000-01-01');
insert or ignore  into product (product_cd, name, product_type_cd, date_offered)
values ('SBL','small business loan','LOAN','2000-01-01');



/* residential customer data */
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (1, '111-11-1111', 'I', '47 Mockingbird Ln', 'Lynnfield', 'MA', '01940');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (2, '222-22-2222', 'I', '372 Clearwater Blvd', 'Woburn', 'MA', '01801');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (3, '333-33-3333', 'I', '18 Jessup Rd', 'Quincy', 'MA', '02169');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (4, '444-44-4444', 'I', '12 Buchanan Ln', 'Waltham', 'MA', '02451');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (5, '555-55-5555', 'I', '2341 Main St', 'Salem', 'NH', '03079');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (6, '666-66-6666', 'I', '12 Blaylock Ln', 'Waltham', 'MA', '02451');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (7, '777-77-7777', 'I', '29 Admiral Ln', 'Wilmington', 'MA', '01887');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (8, '888-88-8888', 'I', '472 Freedom Rd', 'Salem', 'NH', '03079');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (9, '999-99-9999', 'I', '29 Maple St', 'Newton', 'MA', '02458');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (10, '04-1111111', 'B', '7 Industrial Way', 'Salem', 'NH', '03079');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (11, '04-2222222', 'B', '287A Corporate Ave', 'Wilmington', 'MA', '01887');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (12, '04-3333333', 'B', '789 Main St', 'Salem', 'NH', '03079');
insert or ignore  into customer (cust_id, fed_id, cust_type_cd,
  address, city, state, postal_code)
values (13, '04-4444444', 'B', '4772 Presidential Way', 'Quincy', 'MA', '02169');


insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'James', 'Hadley', '1972-04-22' from customer
where fed_id = '111-11-1111';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'Susan', 'Tingley', '1968-08-15' from customer
where fed_id = '222-22-2222';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'Frank', 'Tucker', '1958-02-06' from customer
where fed_id = '333-33-3333';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'John', 'Hayward', '1966-12-22' from customer
where fed_id = '444-44-4444';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'Charles', 'Frasier', '1971-08-25' from customer
where fed_id = '555-55-5555';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'John', 'Spencer', '1962-09-14' from customer
where fed_id = '666-66-6666';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'Margaret', 'Young', '1947-03-19' from customer
where fed_id = '777-77-7777';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'Louis', 'Blake', '1977-07-01' from customer
where fed_id = '888-88-8888';
insert or ignore  into individual (cust_id, fname, lname, birth_date)
select cust_id, 'Richard', 'Farley', '1968-06-16' from customer
where fed_id = '999-99-9999';


insert or ignore  into business (cust_id, name, state_id, incorp_date)
select cust_id, 'Chilton Engineering', '12-345-678', '1995-05-01' from customer
where fed_id = '04-1111111';
insert or ignore  into business (cust_id, name, state_id, incorp_date)
select cust_id, 'Northeast Cooling Inc.', '23-456-789', '2001-01-01' from customer
where fed_id = '04-2222222';
insert or ignore  into business (cust_id, name, state_id, incorp_date)
select cust_id, 'Superior Auto Body', '34-567-890', '2002-06-30' from customer
where fed_id = '04-3333333';
insert or ignore  into business (cust_id, name, state_id, incorp_date)
select cust_id, 'AAA Insurance Inc.', '45-678-901', '1999-05-01' from customer
where fed_id = '04-4444444';



insert or ignore  into officer (officer_id, cust_id, fname, lname,
  title, start_date)
select 1, cust_id, 'John', 'Chilton', 'President', '1995-05-01'
from customer
where fed_id = '04-1111111';
insert or ignore  into officer (officer_id, cust_id, fname, lname,
  title, start_date)
select 2, cust_id, 'Paul', 'Hardy', 'President', '2001-01-01'
from customer
where fed_id = '04-2222222';
insert or ignore  into officer (officer_id, cust_id, fname, lname,
  title, start_date)
select 3, cust_id, 'Carl', 'Lutz', 'President', '2002-06-30'
from customer
where fed_id = '04-3333333';
insert or ignore  into officer (officer_id, cust_id, fname, lname,
  title, start_date)
select 4, cust_id, 'Stanley', 'Cheswick', 'President', '1999-05-01'
from customer
where fed_id = '04-4444444';


insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 1, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Woburn' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2000-01-15' open_date, '2005-01-04' last_date,
    1057.75 avail, 1057.75 pend union all
  select 'SAV' prod_cd, '2000-01-15' open_date, '2004-12-19' last_date,
    500.00 avail, 500.00 pend union all
  select 'CD' prod_cd, '2004-06-30' open_date, '2004-06-30' last_date,
    3000.00 avail, 3000.00 pend) a
where c.fed_id = '111-11-1111';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 2, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Woburn' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2001-03-12' open_date, '2004-12-27' last_date,
    2258.02 avail, 2258.02 pend union all
  select 'SAV' prod_cd, '2001-03-12' open_date, '2004-12-11' last_date,
    200.00 avail, 200.00 pend) a
where c.fed_id = '222-22-2222';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 3, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Quincy' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2002-11-23' open_date, '2004-11-30' last_date,
    1057.75 avail, 1057.75 pend union all
  select 'MM' prod_cd, '2002-12-15' open_date, '2004-12-05' last_date,
    2212.50 avail, 2212.50 pend) a
where c.fed_id = '333-33-3333';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 4, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Waltham' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2003-09-12' open_date, '2005-01-03' last_date,
    534.12 avail, 534.12 pend union all
  select 'SAV' prod_cd, '2000-01-15' open_date, '2004-10-24' last_date,
    767.77 avail, 767.77 pend union all
  select 'MM' prod_cd, '2004-09-30' open_date, '2004-11-11' last_date,
    5487.09 avail, 5487.09 pend) a
where c.fed_id = '444-44-4444';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 5, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Salem' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2004-01-27' open_date, '2005-01-05' last_date,
    2237.97 avail, 2897.97 pend) a
where c.fed_id = '555-55-5555';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 6, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Waltham' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2002-08-24' open_date, '2004-11-29' last_date,
    122.37 avail, 122.37 pend union all
  select 'CD' prod_cd, '2004-12-28' open_date, '2004-12-28' last_date,
    10000.00 avail, 10000.00 pend) a
where c.fed_id = '666-66-6666';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 7, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Woburn' limit 1) e
  cross join
 (select 'CD' prod_cd, '2004-01-12' open_date, '2004-01-12' last_date,
    5000.00 avail, 5000.00 pend) a
where c.fed_id = '777-77-7777';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 8, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Salem' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2001-05-23' open_date, '2005-01-03' last_date,
    3487.19 avail, 3487.19 pend union all
  select 'SAV' prod_cd, '2001-05-23' open_date, '2004-10-12' last_date,
    387.99 avail, 387.99 pend) a
where c.fed_id = '888-88-8888';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 9, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Waltham' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2003-07-30' open_date, '2004-12-15' last_date,
    125.67 avail, 125.67 pend union all
  select 'MM' prod_cd, '2004-10-28' open_date, '2004-10-28' last_date,
    9345.55 avail, 9845.55 pend union all
  select 'CD' prod_cd, '2004-06-30' open_date, '2004-06-30' last_date,
    1500.00 avail, 1500.00 pend) a
where c.fed_id = '999-99-9999';


insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 10, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Salem' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2002-09-30' open_date, '2004-12-15' last_date,
    23575.12 avail, 23575.12 pend union all
  select 'BUS' prod_cd, '2002-10-01' open_date, '2004-08-28' last_date,
    0 avail, 0 pend) a
where c.fed_id = '04-1111111';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 11, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Woburn' limit 1) e
  cross join
 (select 'BUS' prod_cd, '2004-03-22' open_date, '2004-11-14' last_date,
    9345.55 avail, 9345.55 pend) a
where c.fed_id = '04-2222222';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 12, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Salem' limit 1) e
  cross join
 (select 'CHK' prod_cd, '2003-07-30' open_date, '2004-12-15' last_date,
    38552.05 avail, 38552.05 pend) a
where c.fed_id = '04-3333333';
insert or ignore  into account (account_id, product_cd, cust_id, open_date,
  last_activity_date, status, open_branch_id,
  open_emp_id, avail_balance, pending_balance)
select 13, a.prod_cd, c.cust_id, a.open_date, a.last_date, 'ACTIVE',
  e.branch_id, e.emp_id, a.avail, a.pend
from customer c cross join
 (select b.branch_id, e.emp_id
  from branch b inner join employee e on e.assigned_branch_id = b.branch_id
  where b.city = 'Quincy' limit 1) e
  cross join
 (select 'SBL' prod_cd, '2004-02-22' open_date, '2004-12-17' last_date,
    50000.00 avail, 50000.00 pend) a
where c.fed_id = '04-4444444';

/* put $100 in all checking/savings accounts on date account opened */
insert or ignore  into transaction_t (txn_id, txn_date, account_id, txn_type_cd,
  amount, funds_avail_date)
select 1, a.open_date, a.account_id, 'CDT', 100, a.open_date
from account a
where a.product_cd IN ('CHK','SAV','CD','MM');
```

## 演示例子的补充信息<a id="orgheadline28"></a>

该演示数据，按照《SQL学习指南》一书所说，是为了某一银行建立的数据模型，不过我们看到，其中涉及的顾客，部门，产品等概念在很多领域都是适用的。

下面列出关于这些表格的含义说明:

-   **account:** 为特定顾客开放的特定产品
-   **branch:** 开展银行交易业务的场所，就可理解为某分公司分银行。
-   **business:** 公司顾客
-   **customer:** 与银行有业务往来的个人或公司
-   **department:** 就可理解为银行内某部门
-   **employee:** 银行工作人员
-   **individual:** 个人顾客，我们看到和business对应。
-   **officer:** 允许为公司顾客发起交易的人
-   **product:** 向顾客提供的银行服务
-   **product\_type:** 服务类型
-   **transaction:** 改变账户余额的操作

# 简单的SQL查询<a id="orgheadline30"></a>

基于前面的第二个例子构建的数据库，下面我们对SQL最简单的一些查询过一遍。

# 联接<a id="orgheadline32"></a>

关于联接的抽象理论讨论还是很有必要的，这里我也谈论一些，但具体还是需要读者自己沉思。我们把一个SQL表格看作一个对应外部世界的模型，你可以将其看作柏拉图所谈论的理念。然后就作为外部世界各个模型来说，彼此之间有两种关系: 第一种是继承关系，第二种是组合关系。所谓继承关系是指对于每一个苹果实体来说，它有各种各样的属性，我们不可能将其装入一个数据库中，比如说本苹果实体的供应商信息，本苹果的重量颜色信息等等。更合理的做法是我们将描述苹果（或者所有产品）供应商信息单独放入一个模型，然后将描述苹果重量颜色等信息也单独放入一个模型，除此之外还有水果店信息等等。我们在实际处理的时候，常常各个衍生模型都有一个parent\_id类似这样的字段属性来描述本衍生模型的该记录所对应的实际母体parent是谁，从而将多个衍生模型在一个实体对象上统一起来，从而形成表的联接。你可以看作表的联接的最终成果就是你心目中所想的那个超大型SQL表格，对应的是苹果这个模型，里面存放着所有一切和该苹果相关的属性信息。然后你用SELECT语句查一下即可。

第二个组合关系就是苹果由苹果皮和苹果肉等部分组成，各子部分和母体一样具有某种实体性，这个时候我们应该在苹果这个模型上加入如同pingguopi\_id和pingguorou\_id这样的字段属性，然后约定其他以\_id结尾的都视作本苹果的组成部分，不是的都视作非其他组成部分独独为我苹果模型所有的属性。这个组合关系也很重要，在后面再谈及，下面主要谈论表格的联接，而这里表格的联接主要在描述模型之间的继承关系。

具体语法如下所示:

    SELECT name,price,weight
    FROM Supplier,Info
    WHERE Supplier.parent_id = Info.parent_id

这里我们约定Supplier和Info这两个衍生模型内存储的parent\_id具体就应该Pingguo这个模型中的记录号。然后记录号相等就代表着同一个苹果实体。然后将这同一个苹果实体的name, price, weight 等属性列出来。

如果不使用WHERE字句会执行笛卡尔乘积操作，各个字段属性随意组合，没有任何现实意义了。

## 内部联接<a id="orgheadline31"></a>

按照SQL规范，已经不推荐使用这样的WHERE字句表达了，上面的内部联接语句更推荐的是写成这样的形式:

    SELECT name,price,weight
    FROM Supplier INNER JOIN Info
    ON Supplier.parent_id = Info.parent_id

SQL语法里面有一个专门的术语来形容上面的这个SELECT语句，叫做内部联接。注意WHERE变成了ON，然后使用关键词INNER JOIN。SQL联接默认方式就是内部联接。

# 视图<a id="orgheadline36"></a>

视图是虚拟的表，其自身并不包含数据，更确切来说只是一种检索手段。视图可以嵌套视图，但需要注意的是视图因为不包含数据，依赖于检索，所以过多使用视图会很降低性能。

## 新建视图<a id="orgheadline33"></a>

    CREATE VIEW viewname AS 
    SELECT name id FROM Customers, Orders
    WHERE Customers.cust_id = Orders.cust_id

## 删除视图<a id="orgheadline34"></a>

    DROP VIEW viewname;

## 更新视图<a id="orgheadline35"></a>

只能通过先删除视图再新建视图的方式进行。

# 附录<a id="orgheadline38"></a>


## 参考资料<a id="orgheadline37"></a>

1.  SQL必知必会 [英] Ben Forta 著， 钟鸣 刘晓霞等译。