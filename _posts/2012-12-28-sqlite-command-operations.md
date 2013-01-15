---
layout: post
title: "SQLite命令操作"
description: ""
category: Programming/database
tags: [SQLite, commands]
---
{% include JB/setup %}

开源、跨平台、简洁、可嵌入，SQLite是一个非常给力的轻量级的数据库，在移动应用及桌面应用上使用广泛。

在开发的时候，我们时常要创建些测试的数据库并插入模拟数据，这时，我们就需要一个CONSOLE来连接我们的db（其实，就是个软件来访问数据库文件）。标配的`sqlite3`命令行就很给力，以下，介绍这个工具。

一个DB里的层次是这样的，最外层是个db文件，一个db里有1到多个表，1个表有1到多个字段（域），1个表里的一行叫记录，是一个元组。每个字段都对应一种数据类型，在SQLite里出于兼容性考虑，数据类型不那么具体，而是用[`type affinity`][1]的形式来组织。

## 1. 启动

如果要访问磁盘上的测试数据库，输入`sqlite3`后跟文件名作参数，如下所示：

	sqlite3 test.db

如果所跟的文件名不存在，在`sqlite3`里做了写操作，那么退出`sqlite3`后db文件会被创建。

如果不带参数，也会起来sqlite3的工具，但只会在内存里做操作，可以用来临时验证一些SQL语句是否可行。

## 2. sqlite3命令格式

`sqlite3`启动后会有这样的界面：

	SQLite version 3.7.12 2012-04-03 19:43:07
	Enter ".help" for instructions
	Enter SQL statements terminated with a ";"
	sqlite> 
	
很茫然是不是，不知道要干什么？可以打`.help`来显示帮助，是的，没错，前面有个`.`！

在`sqlite3`里，命令都是以`.`开关的，因为没以`.`开头的命令会被当成SQL语句。

## 3. 表级管理

##### 3.1 常用命令

把一个db文件想象成一个文件夹，该db文件夹下的表就是其文件。在UNIX环境下，我们用`ls`来查看文件夹里的文件，在`sqlite3`里则用`.table(s)`命令来查看db里的所有表：

	sqlite> .table
	student

可以看到`test.db`里有一个表，表名"student"。

单单一个表名，信息量比较小，有时，我们更关心表结构，这时，用`.schema`命令；

	sqlite> .schema student
	CREATE TABLE student (id INTEGER, name TEXT);

##### 3.2 表的CRUD

CRUD(create, read, update, delete)一般是指在数据库表里对数据的插入、读取、更新、删除操作。这里对表的CRUD是指，创建表、列出表、更新表、删除表操作，这些操作都可以用SQL语句来：

新建一个表：

	CREATE TABLE course (id INTEGER, name TEXT)
	
列出表：

	.table -- 列出db里的表
	.schema student -- 列出student表的结构

删除表：

	DROP TABLE IF EXISTS course
	
更新表主要是指更新表结构，就是对表字段的操作。目前SQLITE版本中ALTER TABLE不支持DROP COLUMN，只有RENAME和ADD，所以SQLite对删除表的属性支持有限，先说怎么加字段：

	sqlite> ALTER TABLE student ADD COLUMN age INTEGER; -- 添加age属性
	sqlite> .schema student
	CREATE TABLE student (id INTEGER, name TEXT, age INTEGER);
	
减字段比较复杂，不像其他数据库里直接ALTER一个table，DROP其column。在SQLite里你要这么捣腾：先建一个tmp表，包含要删除后剩下的字段，同时把原表的数据都导入，再删除原表，最后把tmp表改名为原表名。最好在一个事务里来做这些事：

	sqlite> BEGIN TRANSACTION;
	sqlite> CREATE TEMPORARY TABLE tmp_student(id, name);
	sqlite> INSERT INTO tmp_student SELECT id, name FROM student;
	sqlite> DROP TABLE student;
	sqlite> CREATE TABLE student(id INTEGER PRIMARY KEY, name TEXT NOT NULL);
	sqlite> INSERT INTO student SELECT id, name FROM tmp_student;
	sqlite> DROP TABLE tmp_student;
	sqlite> COMMIT;
	sqlite> .schema student
	CREATE TABLE student(id INTEGER PRIMARY KEY, name TEXT NOT NULL); -- 结果删除了新加的age字段
	
其实，这里说的对表的CRUD，有另外一种解读。在SQLite中，数据库表的结构也是存在一个表中的`sqlite_master`，但作为一个用户，没有权限对这个表CRUD，只是你用SQL语句来ADD、ALTER、DROP表结构时，会把最终操作的结果带到`sqlite_master`中。另外，就像上面的命令里涉及到的临时表，也是有表来存的`sqlite_temp_master`，对临时表的CRUD最终会体现在这个系统表，但是不会影响到`sqlite_master`。最后，`sqlite_temp_master`在本次`sqlite3`的SESSION内有效，重启`sqlite3`会把所有状态初始化。[REF][2]
	
## 4. 行级操作

所谓的行级操作，就是通常意义上的对数据的CRUD，SQLite支持标准的SQL，这里就不赘述了。

## 5. 输出格式

这节介绍怎么把查询语句输出弄得更pretty。首先是要不要输出表头（字段名）：

	.header(s) ON|OFF 
	
其次，输出模式也可以选择：

	.mode MODE -- MODE值可以是csv, column, html, insert, line, list, tabs和tcl中任一个

这些格式从字面上都好理解，常用的有`column`用来以列的形式输出，数据是按列对齐的；模式`insert`可以把数据记录的插入语句都列出来；模式`csv`和`html`可以用来导出数据用。

然后，如果在上面把`.mode`选为`list`，还可以用`.separator`来控制分割符。同时，它还决定了从文件往数据库里导数据时的分割符：

	.separator STRING      Change separator used by output mode and .import
	
对了，默认输出是`sqlite3`的交互界面，也可以改成输出到文件：

	.output FILENAME       Send output to FILENAME -- 输出文件全路径，确保您对文件具有w属性
	
最后，很多数据库都带有显示一个操作花费时间的功能，SQLite麻雀虽小，五脏俱全，这个当然也得有。

	.timer ON|OFF          Turn the CPU timer measurement on or off

## 6. 批处理

如果一行一行的输入命令，迟早会抓狂，尤其是在造模拟数据的时候。这时候所谓的批处理或者说脚本就大有作为了。
	
对于`sqlite3`有2种方式来运行脚本：

进入`sqlite3`交互界面，读取脚本文件来执行：

	$>sqlite3 test.db
	SQLite version 3.7.12 2012-04-03 19:43:07
	Enter ".help" for instructions
	Enter SQL statements terminated with a ";"
	sqlite> .read '/Users/path/to/your/file/script.sql'
	
注意，要用全路径来输入脚本文件（`~`这样的无法解析出来）。

另一种方式，用shell把脚本文件重定向给`sqlite3`命令：

	$> sqlite3 back_up.db < back_up.sql
	
如果想临时给数据库发个命令，不用进入交互式界面，也不用创建脚本，只需把命令以字符串形式传给`sqlite3`：

	$> sqlite3 test.db ".dump"

顺便说说这个`.dump`命令，可以把数据库表的创建和插入数据的SQL语句都转存下来。



[1]: http://www.sqlite.org/datatype3.html

[2]: http://jianlee.ylinux.org/Computer/%E6%9C%8D%E5%8A%A1%E5%99%A8/sqlite.html

-- EOF