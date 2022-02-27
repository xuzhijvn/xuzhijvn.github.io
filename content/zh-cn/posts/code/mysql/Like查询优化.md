+++
title = "Like查询优化"
date = 2022-02-16T10:04:31+08:00
featured = false
comment = true
toc = true
reward = true
pinned = false
categories = [
  "编程思想"
]
tags = [
  "MySQL"
]
series = [
  "Manual"
]
images = []
+++

本文总结了左前缀匹配（ `LIKE '张三%'`）、右后缀匹配（ `LIKE '%张三'`）和模糊查询（ `LIKE '%张三%'`）常用优化方式。

<!--more-->

测试数据生成 👉  [MySQL快速生成大量测试数据（100万、1000万、1亿）](https://blog.csdn.net/mysqltop/article/details/105230327)

## 左前缀匹配

这种建普通索引就能优化：

```mysql
create index idx_name on t(name);
```

优化效果显著。

## 右后缀匹配

原理：<font color = green>转化成左前缀匹配</font>

实现：1. 新增字符倒置列  2. 生成列 

### 1. 新增字符倒置列 

具体做法是：在该表中增加 `“name_back”` 列，将 `“name”` 列的值倒置后，填入 `“name_back”` 列中，最后为 `“name_back” `列增加索引。

```mysql
ALTER TABLE t ADD name_back nvarchar(1000);-- 增加数据列
UPDATE t SET name_back = reverse(name); -- 填充 name_back 的值
CREATE INDEX t_like_name_back_idx ON t(name_back);-- 为 name_back 列增加索引
```

数据表调整之后，我们的 SQL 语句也需要调整：

```mysql
SELECT * FROM t WHERE name_back LIKE '三张%'
```

### 2. 生成列 

简介 👉 [MySQL生成列](https://www.yiibai.com/mysql/generated-columns.html)

```mysql
ALTER TABLE t ADD COLUMN reverse_name VARCHAR(200) GENERATED ALWAYS AS (REVERSE(name)); -- 生成列
ALTER TABLE t add INDEX idx_reverse_name(reverse_name); -- 建索引
SELECT * FROM t WHERE reverse_name LIKE '三张%'; -- 查询
```

## 模糊查询

原理：<font color = green>分词 + 倒排索引</font>

实现方式：1. 手动分词 + 创建物理倒排索引表   2. 使用MySQL的fulltext全文索引   3. 上es

### 1. 手动分词 + 创建倒排索引表 

👉  [单表千万行数据库 LIKE 搜索优化手记](https://www.coderbusy.com/archives/662.html)

### 2. 使用MySQL的FULLTEXT

当涉及汉语，日语或韩语等表意语言语言时，这些语言不使用分词符。为了解决这个问题，MySQL提供了ngram全文解析器。自MySQL5.7.6版起，MySQL将ngram全文解析器作为内置的服务器插件。👉  [MySQL ngram全文解析器](https://www.yiibai.com/mysql/ngram-full-text-parser.html)

```mysql
create fulltext index fulltext_idx_name on t(name); -- 建全文索引
SELECT * FROM t WHERE MATCH(name) AGAINST('张三'); -- 全文索引查询
```

### 3. 上es

底层基础lucene，专业做模糊查询。
