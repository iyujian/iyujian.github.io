---
layout: post
title:  "MySQL中FIND_IN_SET()函数"
date:   2016-08-09 15:25:30 +0800
categories: database
tags: transaction mysql
---
语法： FIND_IN_SET(str,strlist)。

返回字符串在字符串列表中的位置（从1开始），字符串列表以英文逗号“,”分割，如果第一个参数是字符串常量，第二个是SET类型的列，FIND_IN_SET()方法会使用位运算优化。如果str在strlist中未找到，或者strlist为空，该方法返回0，如果任何一个参数是NULL，则返回NULL。如果第一个参数包含英文逗号“,”，那么该方法不会正确的执行。

文档地址：http://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_find-in-set

Returns a value in the range of 1 to N if the string str is in the string list strlist consisting of N substrings. A string list is a string composed of substrings separated by “,” characters. If the first argument is a constant string and the second is a column of type SET, the FIND_IN_SET() function is optimized to use bit arithmetic. Returns 0 if str is not in strlist or if strlist is the empty string. Returns NULL if either argument is NULL. This function does not work properly if the first argument contains a comma (“,”) character.

FIND_IN_SET()函数实例说明：

<pre>
mysql> SELECT FIND_IN_SET('b','a,b,c,d');
+----------------------------+
| FIND_IN_SET('b','a,b,c,d') |
+----------------------------+
|                          2 |
+----------------------------+
1 row in set (0.00 sec)
</pre>

查询选修了“数学”课程的学生：

<pre>
mysql> select * from student where FIND_IN_SET('数学', course);
+----+---------------------------+---------+
| id | name                      | course   |
+----+---------------------------+---------+
|  1 | 张三                      | 数学 |
|  2 | 李四                      | 语文,数学,历史 |
|  3 | 王五                      | 物理,化学,数学 |
</pre>
