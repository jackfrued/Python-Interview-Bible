## 数据分析师SQL面试宝典

### 题目3

MySQL 数据库中有如下所示的比赛记录表，请用这张表查询出如下所示的结果。

比赛结果记录表（`tb_result`）：两个字段分别代表比赛日期（`game_date`）和比赛结果（`game_rslt`）。

| game_date  | game_rslt |
| :--------: | :-------: |
| 2017-04-09 |    win    |
| 2017-04-09 |    win    |
| 2017-04-09 |   lose    |
| 2017-04-09 |   lose    |
| 2017-04-10 |    win    |
| 2017-04-10 |   lose    |
| 2017-04-10 |    win    |

#### 编写查出如下所示结果的SQL语句

| game_date  | win  | los  |
| :--------: | :--: | :--: |
| 2017-04-09 |  2   |  2   |
| 2017-04-10 |  2   |  1   |

参考答案：

```sql
SELECT game_date
     , SUM(CASE game_rslt WHEN 'win' THEN 1 ELSE 0 END) AS win
     , SUM(CASE game_rslt WHEN 'lose' THEN 1 ELSE 0 end) AS los
  FROM tb_match_result 
 GROUP BY game_date;
```

> **说明**：下面是为大家编写好的建库建表和插入数据的SQL语句，有需要的可以用它来完成建库建表操作。
>
> ```SQL
> -- 创建数据库
> CREATE DATABASE `Q3` DEFAULT CHARSET utf8mb4;
> 
> -- 切换数据库
> USE `Q3`;
> 
> -- 创建比赛结果记录表
> CREATE TABLE `tb_match_result`
> (
> `game_date` date        NOT NULL COMMENT '比赛日期',
> `game_rslt` varchar(10) NOT NULL COMMENT '比赛结果'
> );
> 
> -- 插入比赛结果数据
> INSERT INTO `tb_match_result`
> VALUES 
>  ('2017-04-09', 'win'),
>  ('2017-04-09', 'win'),
>  ('2017-04-09', 'lose'),
>  ('2017-04-09', 'lose'),
>  ('2017-04-10', 'win'),
>  ('2017-04-10', 'lose'),
>  ('2017-04-10', 'win');
> ```

### 题目4

MySQL 数据库中有如下所示的两张表，请用这两张表查询出如下所示的结果。

课程表（`tb_course`）：两个字段分别代表课程编号（`course_id`）和课程名称（`course_name`）。

| course_id | course_name |
| :-------: | :---------: |
|     1     |   Python    |
|     2     |    Java     |
|     3     |  Database   |
|     4     | JavaScript  |

开课日期表（`tb_date`）：两个

| open_date  | course_id |
| :--------: | :-------: |
| 2021-06-01 |     1     |
| 2021-06-02 |     3     |
| 2021-06-02 |     4     |
| 2021-07-04 |     4     |
| 2021-08-02 |     2     |
| 2021-08-02 |     4     |

#### 编写查出如下所示结果的SQL语句

|    Name    | June | July | August |
| :--------: | :--: | :--: | :----: |
|   Python   |  o   |  x   |   x    |
|    Java    |  x   |  x   |   o    |
|  Database  |  o   |  x   |   x    |
| JavaScript |  o   |  o   |   o    |

参考答案：

```sql
SELECT course_name AS Name
     , CASE WHEN EXISTS (SELECT 'x' 
                           FROM tb_date AS t2
                          WHERE t1.course_id = t2.course_id 
                                    AND MONTH(open_date) = 6) 
            THEN 'o' 
            ELSE 'x' 
       END AS June
     , CASE WHEN EXISTS (SELECT 'x' 
                           FROM tb_date AS t2
                          WHERE t1.course_id = t2.course_id 
                                    AND MONTH(open_date) = 7) 
            THEN 'o' 
            ELSE 'x' 
       END AS July
     , CASE WHEN EXISTS (SELECT 'x' 
                           FROM tb_date AS t2
                          WHERE t1.course_id = t2.course_id 
                                    AND MONTH(open_date) = 8) 
            THEN 'o' 
            ELSE 'x' 
       END AS August
  FROM tb_course AS t1;
```

> **说明**：下面是为大家编写好的建库建表和插入数据的SQL语句，有需要的可以用它来完成建库建表操作。
>
> ```SQL
> -- 创建数据库
> CREATE DATABASE `Q4` DEFAULT CHARSET utf8mb4;
> 
> -- 切换数据库
> USE `Q4`;
> 
> -- 创建课程表
> CREATE TABLE `tb_course`
> (
> `course_id`   int unsigned NOT NULL COMMENT '课程编号',
> `course_name` varchar(50)  NOT NULL COMMENT '课程名称',
> PRIMARY KEY (`course_id`)
> );
> 
> INSERT INTO `tb_course`
> VALUES 
>     (1, 'Python'),
>     (2, 'Java'),
>     (3, 'Database'),
>     (4, 'JavaScript');
> 
> -- 创建开课日期表
> CREATE TABLE `tb_date`
> (
> `open_date` date         NOT NULL COMMENT '开课日期',
> `course_id` int unsigned NOT NULL COMMENT '课程编号'
> );
> 
> INSERT INTO `tb_date`
> VALUES
>     ('2021-06-01', 1),
>     ('2021-06-02', 3),
>     ('2021-06-02', 4),
>     ('2021-07-04', 4),
>     ('2021-08-02', 2),
>     ('2021-08-02', 4);
> ```

### 题目5

用如下所示的 SQL 语句在 MySQL 完成建库建表后，通过毕业生收入表（`tb_graduate`）完成下面查询。

```sql
DROP DATABASE IF EXISTS `Q5`;

CREATE DATABASE `Q5` DEFAULT CHARSET utf8mb4;

USE `Q5`;

-- 创建毕业生收入表并插入数据
CREATE TABLE `tb_graduate`
(
`g_name`   varchar(50) NOT NULL COMMENT '名字',
`g_income` int         NOT NULL COMMENT '收入'
);

INSERT INTO `tb_graduate`
VALUES 
    ('Alice', 400000),
    ('Bob', 30000),
    ('Jack', 20000),
    ('Jerry', 20000),
    ('Smith', 20000),
    ('Tom', 15000),
    ('Lucy', 15000),
    ('Lily', 10000),
    ('Ana', 10000),
    ('White', 10000);
```

#### 1. 查出毕业生收入的众数

参考答案：

方法一：

```sql
SELECT g_income
  FROM tb_graduate 
 GROUP BY g_income 
HAVING COUNT(*) >= ALL(SELECT COUNT(*) 
                         FROM tb_graduate 
                        GROUP BY g_income);
```

方法二：

```sql
SELECT g_income
  FROM tb_graduate 
 GROUP BY g_income 
HAVING COUNT(*) = (SELECT MAX(cnt) 
                     FROM (SELECT COUNT(*) AS cnt 
                             FROM tb_graduate 
                            GROUP BY g_income) AS tmp);
```

#### 2. 查出毕业生收入的中位数

参考答案：

方法一：

```sql
SELECT AVG(g_income)
  FROM (SELECT t1.g_income 
          FROM tb_graduate AS t1, tb_graduate AS t2 
         GROUP BY t1.g_income 
        HAVING SUM(CASE WHEN t2.g_income <= t1.g_income 
                        THEN 1 
                        ELSE 0 
                    END) >= COUNT(*) / 2
               AND 
               SUM(CASE WHEN t2.g_income >= t1.g_income 
                        THEN 1 
                        ELSE 0 
                    END) >= COUNT(*) / 2) AS tmp;
```

方法二：

```sql
SELECT AVG(g_income)
  FROM (SELECT g_income
             , ROW_NUMBER() OVER (ORDER BY g_income ASC) AS r
             , COUNT(*) OVER () AS n
          FROM tb_graduate) AS temp
 WHERE r BETWEEN n / 2 AND n / 2 + 1;
```
