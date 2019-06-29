---
layout:     post
title:      "mysql：查询排名"
subtitle:   " DataBase Mysql"
date:       2017-07-13 11:34:07
author:     "TyurinTsien"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - MySql
---


# mysql：查询排名
+ TyurinTsien
+ 2017年07月25日11:34:07

## sql语句查询排名
+ 思路：有点类似循环里面的自增一样，设置一个变量并赋予初始值，循环一次自增加1，从而实现排序；

+ mysql里则是需要先将数据查询出来并先行按照需要排序的字段做好降序desc，或则升序asc，设置好排序的变量（初始值为0）：
1. 将已经排序好的数据从第一条依次取出来，取一条就自增加一，实现从1到最后的一个排名
2. 当出现相同的数据时，排名保持不变，此时则需要再设置一个变量，用来记录上一条数据的值，跟当前数据的值进行对比，如果相同，则排名不变，不相同则排名自增加1
3. 当出现相同的数据时，排名保持不变，但是保持不变的排名依旧会占用一个位置，也就是类似于(1,2,2,2,5)这种排名就是属于中间的三个排名是一样的，但是第五个排名按照上面一种情况是(1,2,2,2,3)，现在则是排名相同也会占据排名的位置

## 准备数据（用户id，分数）：
```markdown
CREATE TABLE `sql_rank` (
`id` int(11) unsigned NOT NULL AUTO_INCREMENT,
`user_id` int(11) unsigned NOT NULL,
`score` tinyint(3) unsigned NOT NULL,
`add_time` date NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

## 插入数据：
```markdown
INSERT INTO sql_rank (user_id, score, add_time)
VALUES
(100, 50, '2016-05-01'),
(101, 30, '2016-05-01'),
(102, 20, '2016-05-01'),
(103, 60, '2016-05-01'),
(104, 80, '2016-05-01'),
(105, 50, '2016-05-01'),
(106, 70, '2016-05-01'),
(107, 85, '2016-05-01'),
(108, 60, '2016-05-01')
```

## 当前数据库数据：
```markdown
mysql> select * from sql_rank;
+----+---------+-------+------------+
| id | user_id | score | add_time   |
+----+---------+-------+------------+
|  1 |     100 |    50 | 2016-05-01 |
|  2 |     101 |    30 | 2016-05-01 |
|  3 |     102 |    20 | 2016-05-01 |
|  4 |     103 |    60 | 2016-05-01 |
|  5 |     104 |    80 | 2016-05-01 |
|  6 |     105 |    50 | 2016-05-01 |
|  7 |     106 |    70 | 2016-05-01 |
|  8 |     107 |    85 | 2016-05-01 |
|  9 |     108 |    60 | 2016-05-01 |
+----+---------+-------+------------+
9 rows in set (0.00 sec)
```

## 三种不同排名
### sql1｛不管数据相同与否，排名依次排序（1,2,3,4,5,6,7.....）
```markdown
SELECT
    obj.user_id,obj.score,@rownum := @rownum + 1 AS rownum
FROM
    (
        SELECT
            user_id,
            score
        FROM
            `sql_rank`
        ORDER BY
            score DESC
    ) AS obj,
    (SELECT @rownum := 0) r
```

+ 执行的结果如下：
```markdown
+---------+-------+--------+
| user_id | score | rownum |
+---------+-------+--------+
|     107 |    85 |      1 |
|     104 |    80 |      2 |
|     106 |    70 |      3 |
|     103 |    60 |      4 |
|     108 |    60 |      5 |
|     100 |    50 |      6 |
|     105 |    50 |      7 |
|     101 |    30 |      8 |
|     102 |    20 |      9 |
+---------+-------+--------+
9 rows in set (0.01 sec)
```
+ 可以看到，现在按照分数从1到9都排好序了，但是有些分数相同的用户排名却不一样，这就是接下来要说的第二种sql

### sql2｛只要数据有相同的排名就一样，排名依次排序（1,2,2,3,3,4,5.....）｝
```markdown
SELECT
    obj.user_id,
    obj.score,
    CASE
WHEN @rowtotal = obj.score THEN
    @rownum
WHEN @rowtotal := obj.score THEN
    @rownum :=@rownum + 1
WHEN @rowtotal = 0 THEN
    @rownum :=@rownum + 1
END AS rownum
FROM
    (
        SELECT
            user_id,
            score
        FROM
            `sql_rank`
        ORDER BY
            score DESC
    ) AS obj,
    (SELECT @rownum := 0 ,@rowtotal := NULL) r
```
+ 这时候就新增加了一个变量，用于记录上一条数据的分数了，只要当前数据分数跟上一条数据的分数比较，相同分数的排名就不变，不相同分数的排名就加一，并且更新变量的分数值为该条数据的分数，依次比较

+ 结果如下：
```markdown
+---------+-------+--------+
| user_id | score | rownum |
+---------+-------+--------+
|     107 |    85 |      1 |
|     104 |    80 |      2 |
|     106 |    70 |      3 |
|     103 |    60 |      4 |
|     108 |    60 |      4 |
|     100 |    50 |      5 |
|     105 |    50 |      5 |
|     101 |    30 |      6 |
|     102 |    20 |      7 |
+---------+-------+--------+
9 rows in set (0.00 sec)
```
+ 跟第一条sql的结果相对比你会发现，分数相同的排名也相同，并且最后一名的名次由第9名变成了第7名；

+ 如果你需要分数相同的排名也相同，但是后面的排名不能受到分数相同排名相同而不占位的影响，也就是哪怕你排名相同，你也占了这个位置（比如：1,2,2,4,5,5,7....这种形式的，虽然排名有相同，但是你占位了，后续的排名根据占位来排）

### sql3｛只要数据有相同的排名就一样，但是相同排名也占位，排名依次排序（1,2,2,4,5,5,7.....）｝
+ 此时需呀再增加一个变量，来记录排序的号码（自增）
```markdown
SELECT
    obj_new.user_id,
    obj_new.score,
    obj_new.rownum
FROM
    (
        SELECT
            obj.user_id,
            obj.score,
            @rownum := @rownum + 1 AS num_tmp,
            @incrnum := CASE
        WHEN @rowtotal = obj.score THEN
            @incrnum
        WHEN @rowtotal := obj.score THEN
            @rownum
        END AS rownum
        FROM
            (
                SELECT
                    user_id,
                    score
                FROM
                    `sql_rank`
                ORDER BY
                    score DESC
            ) AS obj,
            (
                SELECT
                    @rownum := 0 ,@rowtotal := NULL ,@incrnum := 0
            ) r
    ) AS obj_new
```
+ 上面sql执行的结果如下：
```markdown
+---------+-------+--------+
| user_id | score | rownum |
+---------+-------+--------+
|     107 |    85 | 1      |
|     104 |    80 | 2      |
|     106 |    70 | 3      |
|     103 |    60 | 4      |
|     108 |    60 | 4      |
|     100 |    50 | 6      |
|     105 |    50 | 6      |
|     101 |    30 | 8      |
|     102 |    20 | 9      |
+---------+-------+--------+
9 rows in set (0.00 sec)
```
结果集中分数相同的，排名相同，同时它也占据了那个位置