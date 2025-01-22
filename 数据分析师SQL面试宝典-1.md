## 数据分析师SQL面试宝典

### 题目1

MySQL 数据库中有如下所示的两张表部门表和员工表，请完成下面的查询。

部门表（`tb_dept`）：三个字段分别代表部门编号（`dno`）、部门名称（`dname`）和部门所在地（`dloc`）。

| dno  | dname  | dloc |
| :--: | :----: | :--: |
|  10  | 会计部 | 北京 |
|  20  | 研发部 | 成都 |
|  30  | 销售部 | 重庆 |
|  40  | 运维部 | 深圳 |

员工表（`tb_emp`）：六个字段分别代表员工编号（`eno`）、姓名（`ename`）、职位（`job`）、主管编号（`mgr`）、月薪（`sal`）、所在部门编号（`dno`），其中主管编号参照了员工表的员工编号，所在部门编号参照了部门表的部门编号。

| eno  | ename  |   job    | mgr  | sal  | dno  |
| :--: | :----: | :------: | :--: | :--: | :--: |
| 1359 | 胡一刀 |  销售员  | 3344 | 1800 |  30  |
| 2056 |  乔峰  |  分析师  | 7800 | 5000 |  20  |
| 3088 | 李莫愁 |  设计师  | 2056 | 3500 |  20  |
| 3211 | 张无忌 |  程序员  | 2056 | 3200 |  20  |
| 3233 | 丘处机 |  程序员  | 2056 | 3400 |  20  |
| 3244 | 欧阳锋 |  程序员  | 3088 | 3200 |  20  |
| 3251 | 张翠山 |  程序员  | 2056 | 4000 |  20  |
| 3344 |  黄蓉  | 销售主管 | 7800 | 3000 |  30  |
| 3577 |  杨过  |   会计   | 5566 | 2200 |  10  |
| 3588 | 朱九真 |   会计   | 5566 | 2500 |  10  |
| 4466 | 苗人凤 |  销售员  | 3344 | 2500 |  30  |
| 5234 |  郭靖  |   出纳   | 5566 | 2000 |  10  |
| 5566 | 宋远桥 |  会计师  | 7800 | 4000 |  10  |
| 7800 | 张三丰 |   总裁   | NULL | 9000 |  20  |

#### 1. 查询员工和他的主管的姓名

参考答案：

```sql
SELECT t1.ename AS 员工姓名
     , t2.ename AS 主管姓名
  FROM tb_emp AS t1
       LEFT JOIN tb_emp AS t2
           ON t1.mgr = t2.eno;
```

> **说明**：本题通过自连接就可以完成，使用左外连接是为了将没有主管的总裁张三丰也能查出来。

#### 2. 查询月薪最高的员工姓名和月薪

参考答案：

方法一：嵌套查询（先查出最高的月薪再用它当条件筛选出对应的员工）

```sql
SELECT ename
     , sal
  FROM tb_emp
 WHERE sal = (SELECT MAX(sal)
                FROM tb_emp);
```

方法二：ALL运算符（拿你的月薪和所有人比较，看看是否满足条件`>=`）

```sql
SELECT ename
     , sal
  FROM tb_emp
 WHERE sal >= ALL(SELECT sal
                    FROM tb_emp);
```

方法三：计数法（月薪比你更高的员工人数为`0`，你就是月薪最高的）

```sql
SELECT ename
     , sal
  FROM tb_emp AS t1
 WHERE (SELECT COUNT(*)
          FROM tb_emp AS t2
         WHERE t2.sal > t1.sal) = 0;
```

方法四：存在性判断（不存在有人比你月薪更高，那你就是月薪最高的）

```sql
SELECT ename
     , sal
  FROM tb_emp AS t1
 WHERE NOT EXISTS (SELECT 'x'
                     FROM tb_emp AS t2
                    WHERE t2.sal > t1.sal);
```

> **说明**：存在性判断并不需要投影某个具体的字段，所以通常直接写常量`'x'`或`1`来代替具体的字段。

#### 3. 查询月薪Top3的员工姓名和月薪

参考答案：

```sql
SELECT ename
     , sal
  FROM tb_emp AS t1
 WHERE (SELECT COUNT(*)
          FROM tb_emp AS t2
         WHERE t2.sal > t1.sal) < 3
 ORDER BY sal DESC;
```

> **说明**：这个查询跟上一个查询的方法三道理是一样的，如果月薪比我更高的人不会有3个，那么我们就是Top3。

#### 4. 查询部门人数超过5个人的部门的编号和人数

参考答案：

```sql
SELECT dno AS 部门编号
     , COUNT(*) AS 人数
  FROM tb_emp
 GROUP BY dno
HAVING COUNT(*) > 5;
```

> **说明**：分组之前的数据筛选使用`WHERE`子句，分组之后的数据筛选要使用`HAVING`子句。

#### 5. 查询所有部门的名称和人数

参考答案：

```sql
SELECT dname AS 部门名称
     , COALESCE(total, 0) AS 部门人数
  FROM tb_dept AS t1
       LEFT JOIN (SELECT dno
                       , COUNT(*) AS total
                    FROM tb_emp
                   GROUP BY dno) AS t2
           ON t1.dno = t2.dno;
```

> **说明**：先通过一个嵌套查询获取到部门编号和人数，再通过左外连接的方式连接部门表。

#### 4. 查询月薪超过其所在部门平均月薪的员工的姓名、部门编号和月薪

参考答案：

```sql
SELECT ename
     , dno
     , sal
  FROM tb_emp AS t1
       NATURAL JOIN (SELECT dno
                          , AVG(sal) AS avg_sal
                       FROM tb_emp
                      GROUP BY dno) AS t2
 WHERE sal > avg_sal;
```

#### 5. 查询部门中月薪最高的人姓名、月薪和所在部门名称

参考答案：

```sql
SELECT ename
     , sal
     , dname
  FROM tb_dept AS t1 
       NATURAL JOIN tb_emp AS t2
 WHERE (dno, sal) IN (SELECT dno
                           , max(sal) 
                        FROM tb_emp
                       GROUP BY dno);
```

#### 6. 查询主管和普通员工的平均月薪

参考答案：

```sql
SELECT tag AS 职级
     , ROUND(avg(sal), 2) AS 平均月薪
  FROM (SELECT sal
             , CASE WHEN EXISTS (SELECT 'x' 
                                   FROM tb_emp AS t2 
                                  WHERE t1.eno = t2.mgr) 
                    THEN '主管' 
                    ELSE '普通员工' 
               END AS tag
          FROM tb_emp AS t1) AS tmp
 GROUP BY tag;
```

> **说明**：先通过一个嵌套查询，给原来员工表的数据加上“主管”或“普通员工”的标签，然后根据标签分组数据再做聚合。

#### 7. 查询月薪排名4~6名的员工排名、姓名和月薪

参考答案：

方法一：不使用窗口函数

```sql
SELECT *
  FROM (SELECT @a := @a + 1 AS 排名
             , ename AS 姓名
             , sal AS 月薪
          FROM tb_emp, (SELECT @a := 0) AS tmp
         ORDER BY sal DESC) AS tmp
 WHERE 排名 BETWEEN 4 AND 6;
```

方法二：使用窗口函数

```sql
SELECT *
  FROM (SELECT ename AS 姓名
             , sal AS 月薪
             , RANK() OVER (ORDER BY sal DESC) AS 排名
          FROM tb_emp) AS tmp
 WHERE 排名 between 4 and 6;
```

> **说明**：上面方法一和方法二的查询结果并不相同，方法二的查询结果更符合我们对这个问题的认知，如果要获得跟方法一一样的结果，可以将窗口函数`RANK`修改为`ROW_NUMBER`即可。`RANK`、`DENSE_RANK`、`ROW_NUMBER`三个函数的差别大家可以自行了解。

#### 8. 查询每个部门月薪排前2名的员工姓名、月薪和部门编号

参考答案：

方法一：不使用窗口函数

```sql
SELECT ename
     , sal
     , t1.dno
  FROM tb_emp AS t1
 WHERE (SELECT COUNT(*)
          FROM tb_emp AS t2
         WHERE t2.dno = t1.dno 
               AND t2.sal > t1.sal) < 2
 ORDER BY dno ASC, sal DESC;
```

方法二：使用窗口函数

```sql
SELECT ename
     , sal
     , dno
  FROM (SELECT ename
             , sal
             , dno
             , RANK() OVER (PARTITION BY dno ORDER BY sal DESC) AS rn
          FROM tb_emp) AS tmp
 WHERE rn <= 2;
```

> **说明**：上面的题目总体难度不大，但是覆盖到的 SQL 知识点还是比较多的，大家可以用这些题来热热身，再去挑战后面的 SQL 面试题，这些面试题主要是针对数据分析师的 SQL 面试题。如果对 SQL 的知识不太熟悉，大家可以先移步到我在B站上发布的视频[《数据分析师的SQL入门课》](https://www.bilibili.com/video/BV13V4y1H7mT)进行学习。下面是为大家编写好的建库建表和插入数据的SQL语句，有需要的可以用它来完成建库建表操作。
>
> ```SQL
> -- 创建数据库
> CREATE DATABASE `Q1` DEFAULT CHARSET utf8mb4;
> 
> -- 切换数据库
> USE `Q1`;
> 
> -- 创建部门表
> CREATE TABLE `tb_dept`
> (
> `dno`   int         NOT NULL COMMENT '编号',
> `dname` varchar(10) NOT NULL COMMENT '名称',
> `dloc`  varchar(20) NOT NULL COMMENT '所在地',
> PRIMARY KEY (`dno`)
> );
> 
> -- 插入部门数据
> INSERT INTO `tb_dept`
> VALUES 
>     (10, '会计部', '北京'),
>     (20, '研发部', '成都'),
>     (30, '销售部', '重庆'),
>     (40, '运维部', '深圳');
> 
> -- 创建员工表
> CREATE TABLE `tb_emp`
> (
> `eno`   int         NOT NULL COMMENT '员工编号',
> `ename` varchar(20) NOT NULL COMMENT '员工姓名',
> `job`   varchar(20) NOT NULL COMMENT '员工职位',
> `mgr`   int                  COMMENT '主管编号',
> `sal`   int         NOT NULL COMMENT '员工月薪',
> `dno`   int         NOT NULL COMMENT '所在部门编号',
> PRIMARY KEY (`eno`),
> CONSTRAINT `fk_emp_mgr` FOREIGN KEY (`mgr`) REFERENCES tb_emp (`eno`),
> CONSTRAINT `fk_emp_dno` FOREIGN KEY (`dno`) REFERENCES tb_dept (`dno`)
> );
> 
> -- 插入员工数据
> INSERT INTO `tb_emp`
> VALUES 
>     (7800, '张三丰', '总裁', NULL, 9000, 20),
>     (2056, '乔峰', '分析师', 7800, 5000, 20),
>     (3088, '李莫愁', '设计师', 2056, 3500, 20),
>     (3211, '张无忌', '程序员', 2056, 3200, 20),
>     (3233, '丘处机', '程序员', 2056, 3400, 20),
>     (3251, '张翠山', '程序员', 2056, 4000, 20),
>     (5566, '宋远桥', '会计师', 7800, 4000, 10),
>     (5234, '郭靖', '出纳', 5566, 2000, 10),
>     (3344, '黄蓉', '销售主管', 7800, 3000, 30),
>     (1359, '胡一刀', '销售员', 3344, 1800, 30),
>     (4466, '苗人凤', '销售员', 3344, 2500, 30),
>     (3244, '欧阳锋', '程序员', 3088, 3200, 20),
>     (3577, '杨过', '会计', 5566, 2200, 10),
>     (3588, '朱九真', '会计', 5566, 2500, 10);
> ```

  

