---
layout: post
title:  "执行数据库'INSERT'操作时事务无法回滚"
date:   2016-01-16 16:56:17 +0800
categories: database
tags: transaction mysql
---
在使用Spring声明式事务的时候，发现在做"update"时，出现异常事务可以回滚，但是在执行"insert"的时候，后台日志虽然显示回滚了，但是数据却已经保存到了数据库中，本来以为是哪里配置出错了，后来却发现是Mysql存储引擎的的问题。我们用的Mysql版本是"5.1.73"，默认存储引擎是"MyISAM"。

查看mysql版本：

<pre>
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.1.73    |
+-----------+
1 row in set (0.00 sec)
</pre>

查看mysql存储引擎

<pre>
mysql> SHOW ENGINES;
+------------+---------+------------------------------------------------------------+--------------+------+------------+
| Engine     | Support | Comment                                                    | Transactions | XA   | Savepoints |
+------------+---------+------------------------------------------------------------+--------------+------+------------+
| MRG_MYISAM | YES     | Collection of identical MyISAM tables                      | NO           | NO   | NO         |
| CSV        | YES     | CSV storage engine                                         | NO           | NO   | NO         |
| MyISAM     | DEFAULT | Default engine as of MySQL 3.23 with great performance     | NO           | NO   | NO         |
| InnoDB     | YES     | Supports transactions, row-level locking, and foreign keys | YES          | YES  | YES        |
| MEMORY     | YES     | Hash based, stored in memory, useful for temporary tables  | NO           | NO   | NO         |
+------------+---------+------------------------------------------------------------+--------------+------+------------+
5 rows in set (0.00 sec)
</pre>

到了这里原因就了然了，创建表时默认为"MyISAM"，关于"MyISAM"和"InnoDB"及其他的各种存储引擎的区别这里就不展开了，我们只需要将我们的表的存储引擎改为"InnoDB"就可以了。

<pre>
ALTER TABLE table_name ENGINE = InnoDB;
</pre>

<pre>
mysql> ALTER TABLE table_name ENGINE = InnoDB;
Query OK, 8 rows affected (0.03 sec)
Records: 8  Duplicates: 0  Warnings: 0
</pre>

如果是主库的话，最好是把默认的存储引擎改为 "InnoDB"。
