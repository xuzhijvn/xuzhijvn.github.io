+++
title = "MySQL实验"
description = ""
date = 2021-09-22T14:10:51+08:00
featured = false
comment = true
toc = true
reward = true
categories = [
  "编程思想"
]
tags = [
  "MySQL"
]
series = ["Manual"]
images = []

+++

实验前提：MySQL默认隔离级别 = `REPEATABLE-READ`

<!--more-->

Session1

初始化商品数量=1

```sql
mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set (0.02 sec)

mysql> select connection_id();
+-----------------+
| connection_id() |
+-----------------+
|         6667466 |
+-----------------+
1 row in set (0.01 sec)
 
mysql> update test.good set good_num = 1 where good_name = 'iphone13';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from test.good;
+----+-----------+----------+---------------------+---------------------+
| id | good_name | good_num | updated_at          | created_at          |
+----+-----------+----------+---------------------+---------------------+
|  1 | iphone13  |        1 | 2021-09-22 14:06:47 | 2021-09-22 10:30:20 |
+----+-----------+----------+---------------------+---------------------+
1 row in set (0.11 sec)
 
mysql> SELECT TRX_ID FROM INFORMATION_SCHEMA.INNODB_TRX  WHERE TRX_MYSQL_THREAD_ID = CONNECTION_ID();
Empty set
```



Session2

1. 关闭自动提交 `set autocommit=0;`
2. 更新商品数量

```sql
mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set (0.02 sec)

mysql> select connection_id();
+-----------------+
| connection_id() |
+-----------------+
|         6667088 |
+-----------------+
1 row in set (0.02 sec)
 
mysql> select * from test.good;
+----+-----------+----------+---------------------+---------------------+
| id | good_name | good_num | updated_at          | created_at          |
+----+-----------+----------+---------------------+---------------------+
|  1 | iphone13  |        1 | 2021-09-22 14:06:47 | 2021-09-22 10:30:20 |
+----+-----------+----------+---------------------+---------------------+
1 row in set (0.01 sec)
 
mysql> SELECT TRX_ID FROM INFORMATION_SCHEMA.INNODB_TRX  WHERE TRX_MYSQL_THREAD_ID = CONNECTION_ID();
+-----------------+
| TRX_ID          |
+-----------------+
| 421350996972592 |
+-----------------+
1 row in set (0.02 sec)
 
mysql> update test.good set good_num = good_num-1 where good_name = 'iphone13';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
 
mysql> select * from test.good;
+----+-----------+----------+---------------------+---------------------+
| id | good_name | good_num | updated_at          | created_at          |
+----+-----------+----------+---------------------+---------------------+
|  1 | iphone13  |        0 | 2021-09-22 14:07:55 | 2021-09-22 10:30:20 |
+----+-----------+----------+---------------------+---------------------+
1 row in set (0.02 sec)
```

Session3

1. 关闭自动提交 `set autocommit=0;`
2. 更新商品数量

```sql
mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set (0.01 sec)

mysql> select connection_id();
+-----------------+
| connection_id() |
+-----------------+
|         6667089 |
+-----------------+
1 row in set (0.02 sec)
 
mysql> select * from test.good;
+----+-----------+----------+---------------------+---------------------+
| id | good_name | good_num | updated_at          | created_at          |
+----+-----------+----------+---------------------+---------------------+
|  1 | iphone13  |        1 | 2021-09-22 14:06:47 | 2021-09-22 10:30:20 |
+----+-----------+----------+---------------------+---------------------+
1 row in set (0.01 sec)
 
mysql> SELECT TRX_ID FROM INFORMATION_SCHEMA.INNODB_TRX  WHERE TRX_MYSQL_THREAD_ID = CONNECTION_ID();
+-----------------+
| TRX_ID          |
+-----------------+
| 421350996977312 |
+-----------------+
1 row in set (0.02 sec)

#会一直阻塞到Session2执行完commit
mysql> update test.good set good_num = good_num-1 where good_name = 'iphone13';
```

执行update的时候会被阻塞，因为Session2事务在update数据的瞬间对数据加了行级排他锁，直到Session2事务结束才释放。



Session2

提交 commit

```sql
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```



Session3

被Session2中事务阻塞的update语句被执行了

```sql
#会一直阻塞到Session2执行完commit
mysql> update test.good set good_num = good_num-1 where good_name = 'iphone13';
Query OK, 1 row affected (8.22 sec)
Rows matched: 1  Changed: 1  Warnings: 0
 
mysql> select * from test.good;
+----+-----------+----------+---------------------+---------------------+
| id | good_name | good_num | updated_at          | created_at          |
+----+-----------+----------+---------------------+---------------------+
|  1 | iphone13  |       -1 | 2021-09-22 14:08:42 | 2021-09-22 10:30:20 |
+----+-----------+----------+---------------------+---------------------+
1 row in set (0.02 sec)
```



Session1

查看商品数量

```sql
mysql> select * from test.good;
+----+-----------+----------+---------------------+---------------------+
| id | good_name | good_num | updated_at          | created_at          |
+----+-----------+----------+---------------------+---------------------+
|  1 | iphone13  |        0 | 2021-09-22 14:07:55 | 2021-09-22 10:30:20 |
+----+-----------+----------+---------------------+---------------------+
1 row in set (0.02 sec)
```



Session3

提交 commit

```sql
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```



Session1

查看商品数量

```sql
mysql> select * from test.good;
+----+-----------+----------+---------------------+---------------------+
| id | good_name | good_num | updated_at          | created_at          |
+----+-----------+----------+---------------------+---------------------+
|  1 | iphone13  |       -1 | 2021-09-22 14:08:42 | 2021-09-22 10:30:20 |
+----+-----------+----------+---------------------+---------------------+
1 row in set (0.01 sec)
```

