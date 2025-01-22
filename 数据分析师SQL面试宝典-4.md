## 数据分析师SQL面试宝典

### 题目6

用如下所示的 SQL 语句在 MySQL 完成建库建表后，按要求完成查询。

```sql
DROP DATABASE IF EXISTS `Q6`;

CREATE DATABASE `Q6` DEFAULT CHARSET utf8mb4;

USE `Q6`;

-- 创建用户登录日志表
CREATE TABLE `tb_user_login`
(
`user_id`    varchar(300) NOT NULL COMMENT '用户ID',
`login_date` date         NOT NULL COMMENT '登录日期'
);
 
INSERT INTO `tb_user_login`
VALUES
    ('A', '2019-9-2'),
    ('A', '2019-9-2'),
    ('A', '2019-9-3'),
    ('A', '2019-9-4'),
    ('B', '2019-9-2'),
    ('B', '2019-9-4'),
    ('B', '2019-9-10'),
    ('C', '2019-1-1'),
    ('C', '2019-1-2'),
    ('C', '2019-1-30'),
    ('C', '2019-9-3'),
    ('C', '2019-9-4'),
    ('C', '2019-9-5'),
    ('C', '2019-9-11'),
    ('C', '2019-9-12'),
    ('C', '2019-9-13');
```

#### 1. 查询有连续3天登录行为的用户ID

参考答案：

```sql
SELECT DISTINCT user_id
  FROM (SELECT user_id
             , login_date
             , DENSE_RANK() OVER (PARTITION BY user_id ORDER BY login_date ASC) AS rn
          FROM tb_user_login
		 GROUP BY user_id, login_date
       ) AS tmp
 GROUP BY user_id, subdate(login_date, rn)
HAVING COUNT(*) >= 3;
```

#### 2. 查询每天新增用户数以及他们次留和月留

参考答案：

方法一：

```sql
SELECT first_login AS 日期
     , COUNT(DISTINCT t1.user_id) AS 新增用户数
     , COUNT(DISTINCT t2.user_id) / COUNT(DISTINCT t1.user_id) AS 次留
     , COUNT(DISTINCT t3.user_id) / COUNT(DISTINCT t1.user_id) AS 月留
  FROM (SELECT user_id
             , login_date
             , MIN(login_date) OVER (PARTITION BY user_id ORDER BY login_date ASC) AS first_login
          FROM tb_user_login) AS t1 
               LEFT JOIN tb_user_login AS t2 
                   ON t1.user_id = t2.user_id 
                       AND DATEDIFF(first_login, t2.login_date) = -1 
               LEFT JOIN tb_user_login AS t3
                   ON t1.user_id = t3.user_id 
                       AND DATEDIFF(first_login, t3.login_date) = -29
 GROUP BY first_login;
```

方法二：

```sql
SELECT first_login AS 日期
     , COUNT(DISTINCT user_id) as 新增用户数
     , SUM(CASE DATEDIFF(login_date, first_login) WHEN  1 THEN 1 ELSE 0 END) / COUNT(DISTINCT user_id) AS 次留
     , SUM(CASE DATEDIFF(login_date, first_login) WHEN 29 THEN 1 ELSE 0 END) / COUNT(DISTINCT user_id) AS 月留
  FROM (SELECT user_id
             , login_date
             , MIN(login_date) OVER (PARTITION BY user_id ORDER BY login_date ASC) AS first_login
          FROM tb_user_login) AS tmp 
 GROUP BY first_login;
```

### 题目7

用如下所示的 SQL 语句在 MySQL 完成建库建表后，按要求完成查询。

```sql
DROP DATABASE IF EXISTS `Q7`;

CREATE DATABASE `Q7` DEFAULT CHARSET utf8mb4;

USE `Q7`;

-- 创建直播访问日志表
CREATE TABLE `tb_broadcast_log`
(
`user_id`   bigint   NOT NULL COMMENT '用户ID',
`entr_time` datetime NOT NULL COMMENT '进入时间',
`exit_time` datetime NOT NULL COMMENT '离开时间'
);

INSERT INTO `tb_broadcast_log`
VALUES
    (1, '2022-10-01 13:30:23', '2022-10-01 17:23:35'),
    (2, '2022-10-01 13:35:55', '2022-10-01 16:30:12'),
    (3, '2022-10-01 13:42:02', '2022-10-01 18:15:09'),
    (4, '2022-10-01 14:12:55', '2022-10-01 15:15:35'),
    (5, '2022-10-01 13:30:23', '2022-10-01 17:23:35'),
    (6, '2022-10-01 14:30:23', '2022-10-01 20:21:21'),
    (7, '2022-10-01 15:30:23', '2022-10-01 21:23:35'),
    (8, '2022-10-01 15:35:12', '2022-10-01 18:33:13'),
    (9, '2022-10-01 16:01:02', '2022-10-01 17:59:25'),
    (10, '2022-10-01 16:15:20', '2022-10-01 18:20:56'),
    (11, '2022-10-01 17:31:23', '2022-10-01 19:56:57'),
    (12, '2022-10-01 17:32:23', '2022-10-01 18:30:20'),
    (10, '2022-10-01 18:25:10', '2022-10-01 20:22:22'),
    (1, '2022-10-01 18:30:23', '2022-10-01 21:24:00'),
    (2, '2022-10-01 19:18:55', '2022-10-01 19:25:40'),
    (3, '2022-10-01 19:20:20', '2022-10-01 20:20:07');
```

#### 查询直播流量（在线人数）的峰值

参考答案：

```sql
SELECT MAX(cnt)
  FROM (SELECT SUM(ee_status) OVER (ORDER BY dt ASC) AS cnt
          FROM (SELECT user_id
                     , entr_time AS dt
                     , 1 AS ee_status
                  FROM tb_broadcast_log
                 UNION 
                SELECT user_id
                     , exit_time AS dt
                     , -1 AS ee_status
                  FROM tb_broadcast_log) AS tmp) AS tmp;
```

### 题目8

用如下所示的 SQL 语句在 MySQL 完成建库建表后，按要求完成查询。

```sql
DROP DATABASE IF EXISTS `Q8`;

CREATE DATABASE `Q8` DEFAULT CHARSET utf8mb4;

USE `Q8`;

-- 创建比赛记录表
CREATE TABLE `tb_match`
(
`player_id`  int         NOT NULL COMMENT '选手编号',
`match_date` date        NOT NULL COMMENT '比赛日期',
`result`     varchar(10) NOT NULL COMMENT '比赛结果'
);

INSERT INTO `tb_match`
VALUES
    (1, '2022-10-01', 'win'),
    (1, '2022-10-04', 'win'),
    (1, '2022-10-08', 'win'),
    (1, '2022-10-12', 'draw'),
    (1, '2022-11-15', 'win'),
    (2, '2022-10-02', 'lose'),
    (2, '2022-10-07', 'lose'),
    (3, '2022-10-01', 'win'),
    (3, '2022-10-03', 'lose'),
    (3, '2022-10-08', 'win'),
    (3, '2022-10-11', 'win');
```

#### 查询如下所示的每个运动员最大连胜场次

参考答案：

```sql
SELECT player_id
     , MAX(win_cnt) AS max_win_cnt
  FROM (SELECT player_id
             , SUM(is_win) AS win_cnt
          FROM (SELECT player_id
                     , CASE result WHEN 'win' THEN 1 ELSE 0 END AS is_win
                     , ROW_NUMBER() OVER (PARTITION BY player_id ORDER BY match_date ASC) AS seq1
                     , ROW_NUMBER() OVER (PARTITION BY player_id, result ORDER BY match_date ASC) AS seq2
                  FROM tb_match) AS tmp
         GROUP BY player_id, seq1 - seq2) AS tmp
 GROUP BY player_id;
```

