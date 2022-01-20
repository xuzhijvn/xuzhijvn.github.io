---
title: "MySQL Integer类型与INT(11)"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"MySQL"
]
series : [
"Manual"
]
images : [

]
---

[comment]: <> "# MySQL Integer类型与INT(11)"

## 1.介绍

Integer类型，即整数类型，MySQL支持的整数类型有TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT。

 

### 1.1 空间和范围

每种整数类型所需的存储空间和范围如下：

| `类型`      | 字节 | 最小值(有符号)                  | 最大值(有符号)                  | 最小值(无符号) | 最大值(无符号)                   |
| ----------- | ---- | ------------------------------- | ------------------------------- | -------------- | -------------------------------- |
| `TINYINT`   | 1    | `-128`                          | `127`                           | `0`            | `255`                            |
| `SMALLINT`  | 2    | `-32768`                        | `32767`                         | `0`            | `65535`                          |
| `MEDIUMINT` | 3    | `-8388608`                      | `8388607`                       | `0`            | `16777215`                       |
| `INT`       | 4    | `-2147483648`                   | `2147483647`                    | `0`            | `4294967295`                     |
| `BIGINT`    | 8    | $-2^{63}$(-9223372036854775808) | $2^{63}-1$(9223372036854775807) | `0`            | $2^{64}-1$(18446744073709551615) |



## 2. INT(11)

### 2.1 数字是否限制长度？

```sql
id INT(11) NOT NULL AUTO_INCREMENT,
```

在一些建表语句会出现上面 int(11) 的类型，那么其代表什么意思呢？

对于Integer类型括号中的数字称为字段的**显示宽度**。这与其他类型字段的含义不同。对于DECIMAL类型，表示数字的总数。对于字符字段，这是可以存储的最大字符数，例如VARCHAR（20）可以存储20个字符。

显示宽度并不影响可以存储在该列中的最大值。INT(5) 和 INT(11)可以存储相同的最大值。哪怕设置成 INT(20) 并不意味着将能够存储20位数字(BIGINT)，该列还是只能存储INT的最大值。

#### 示例

创建一个临时表：

```sql
CREATE TEMPORARY TABLE demo_a (
    id INT(11) NOT NULL AUTO_INCREMENT,
    a INT(1) NOT NULL,
    b INT(5) NOT NULL,
    PRIMARY KEY (`id`)
)
```

插入超过"长度"的数字：

```sql
INSERT INTO demo_a(a,b) VALUES(255, 88888888);
```

查看结果：发现数字并不是设置长度

```sql
mysql> SELECT * FROM demo_a;
+----+-----+----------+
| id | a   | b        |
+----+-----+----------+
|  1 | 255 | 88888888 |
+----+-----+----------+
1 row in set (0.03 sec)
```

 

### 2.2 数字表达什么意思

当列设置为**UNSIGNED ZEROFILL**时，INT(11)才有意义，其表示的意思为如果要存储的数字少于11个字符，则这些数字将在左侧补零。

**注意**：ZEROFILL默认的列为无符号，因此不能存储负数。

#### 示例

创建一个临时表：b列设置为UNSIGNED ZEROFILL

```sql
CREATE TEMPORARY TABLE demo_a (
    id INT(11) NOT NULL AUTO_INCREMENT,
    a INT(11) NOT NULL,
    b INT(11) UNSIGNED ZEROFILL NOT NULL,
    PRIMARY KEY (`id`)
);
```

 插入数值：

```sql
INSERT INTO demo_a(a,b) VALUES(1, 1);
```

 结果：b列的左侧使用了0填充长度

```sql
mysql> SELECT * FROM demo_a;
+----+---+-------------+
| id | a | b           |
+----+---+-------------+
|  1 | 1 | 00000000001 |
+----+---+-------------+
1 row in set (0.18 sec)
```

## 参考

[MySQL Integer类型与INT(11)](https://www.cnblogs.com/polk6/p/11595107.html)
