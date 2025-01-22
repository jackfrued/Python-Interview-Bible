## 数据分析师SQL面试宝典-2

### 题目2

MySQL 数据库中有如下所示的三张表，请用这三张表完成下面的查询。

订单明细表（`tb_order_detail`）：七个字段分别代表流水号（`id`）、订单编号（`order_no`）、用户ID（`user_id`）、下单日期（`order_date`）、店铺ID（`store`）、产品ID（`product`）和购买数量（`quantity`）。

|  id  | order_no |  user_id  | order_date | store  | product | quantity |
| :--: | :------: | :-------: | :--------: | :----: | :-----: | :------: |
|  1   |   001    | customera | 2022-01-01 | storea |  proda  |    1     |
|  2   |   001    | customera | 2022-01-01 | storea |  prodb  |    1     |
|  3   |   001    | customera | 2022-01-01 | storea |  prodc  |    1     |
|  4   |   002    | customerb | 2022-01-12 | storeb |  prodb  |    1     |
|  5   |   002    | customerb | 2022-01-12 | storeb |  prodd  |    1     |
|  6   |   003    | customerc | 2022-01-12 | storec |  prodb  |    1     |
|  7   |   003    | customerc | 2022-01-12 | storec |  prodc  |    1     |
|  8   |   003    | customerc | 2022-01-12 | storec |  prodd  |    1     |
|  9   |   004    | customera | 2022-01-01 | stored |  prodd  |    2     |
|  10  |   005    | customerb | 2022-01-23 | storeb |  proda  |    1     |

产品信息表（`tb_product`）：三个字段分别代表产品ID（`prod_id`）、产品类别（`category`）、产品单价（`price`）。

| prod_id | category | price |
| :-----: | :------: | :---: |
|  proda  |  catea   |  100  |
|  prodb  |  cateb   |  200  |
|  prodc  |  catec   |  300  |
|  prodd  |  cated   |  400  |

店铺信息表（`tb_store`）：两个字段分别代表店铺ID（`store_id`）和所在城市（`city`）。

| store_id | city  |
| :------: | :---: |
|  storea  | citya |
|  storeb  | citya |
|  storec  | cityb |
|  stored  | cityc |
|  storee  | cityd |
|  storef  | cityb |

> **说明**：订单明细表是事实表，产品信息表和店铺信息表示维度表，事实表跟两张维度表之间都存在多对一关系。

#### 1. 查出购买总金额不低于800的用户的总购买金额、总订单数和总购买商品数

> **要求**：订单号（`order_no`）相同的只算作一个订单。

参考答案：

```sql
SELECT user_id AS 用户ID
     , SUM(price * quantity) AS 购买总金额
     , COUNT(DISTINCT order_no) AS 总订单数
     , SUM(quantity) AS 购买商品总数
  FROM tb_order_detail 
       INNER JOIN tb_product
           ON product = prod_id
 GROUP BY user_id
HAVING SUM(price * quantity) >= 800;
```

> **提示**：分组后的筛选要使用`HAVING`子句。

#### 2. 查出所有城市（包含没有产生订单的城市）的总店铺数、总购买人数和总购买金额

参考答案：

```sql
SELECT city AS 
     , COUNT(distinct store_id) AS 总店铺数
     , COUNT(distinct user_id) AS 总购买人数
     , COALESCE(SUM(price * quantity), 0) AS 购买总金额
  FROM tb_store 
       LEFT JOIN tb_order_detail
           ON store_id = store
       LEFT JOIN tb_product
           ON prod_id = product
 GROUP BY city;
```

> **提示**：上面使用`LEFT JOIN`才能确保左表不满足连表条件的记录也能查到，所有城市的信息都能被取到。

#### 3. 查出购买过"catea"产品的用户的平均订单金额

> **要求**：订单号（`order_no`）相同的只算作一个订单。

参考答案：

```sql
WITH tmp AS (
    SELECT *
      FROM tb_order_detail
           INNER JOIN tb_product
               ON product = prod_id
)
SELECT user_id AS 用户ID
     , SUM(price * quantity) / COUNT(DISTINCT order_no) AS 平均订单金额
  FROM tmp
 WHERE user_id IN (SELECT user_id
                     FROM tmp
                    WHERE category = 'catea')
 GROUP BY user_id;
```

> **提示**：上面的 SQL 中使用了 MySQL 8 的 CTE（公共表表达式）语法，先将订单明细表和商品表进行内连接得到一个名为`tmp`的公共表，后面的查询直接基于公共表`tmp`来进行。如果你使用的 MySQL 版本较低，不支持这个语法，可以尝试创建一个跟`tmp`完全一样的视图（*view*），然后再基于视图写查询。当然，如果面试的时候不允许提前创建视图，那就只能做多次表连接操作了，SQL 语句会稍显臃肿。还有一点，题目要求的平均订单金额不能够直接用`AVG`函数来获取，因为订单号相同的只算作一个订单，所以用了求和再除以去重后的订单数来计算平均订单金额。

> **说明**：下面是为大家编写好的建库建表和插入数据的SQL语句，有需要的可以用它来完成建库建表操作。
>
> ```SQL
> -- 创建数据库
> CREATE DATABASE `Q2` DEFAULT CHARSET utf8mb4;
> 
> -- 切换数据库
> USE `Q2`;
> 
> -- 创建订单明细表
> CREATE TABLE `tb_order_detail`
> (
> `id`         int         NOT NULL COMMENT '流水号',
> `order_no`   varchar(20) NOT NULL COMMENT '订单编号',
> `user_id`    varchar(50) NOT NULL COMMENT '用户ID',
> `order_date` date        NOT NULL COMMENT '下单日期',
> `store`      varchar(50) NOT NULL COMMENT '店铺ID',
> `product`    varchar(50) NOT NULL COMMENT '商品ID',
> `quantity`   int         NOT NULL COMMENT '购买数量',
> PRIMARY KEY (`id`)
> );
> 
> -- 插入订单明细数据
> INSERT INTO `tb_order_detail`
> VALUES 
>     (1, '001', 'customera', '2022-01-01', 'storea', 'proda', 1),
>     (2, '001', 'customera', '2022-01-01', 'storea', 'prodb', 1),
>     (3, '001', 'customera', '2022-01-01', 'storea', 'prodc', 1),
>     (4, '002', 'customerb', '2022-01-12', 'storeb', 'prodb', 1),
>     (5, '002', 'customerb', '2022-01-12', 'storeb', 'prodd', 1),
>     (6, '003', 'customerc', '2022-01-12', 'storec', 'prodb', 1),
>     (7, '003', 'customerc', '2022-01-12', 'storec', 'prodc', 1),
>     (8, '003', 'customerc', '2022-01-12', 'storec', 'prodd', 1),
>     (9, '004', 'customera', '2022-01-01', 'stored', 'prodd', 2),
>     (10, '005', 'customerb', '2022-01-23', 'storeb', 'proda', 1);
> 
> -- 创建商品信息表
> CREATE TABLE `tb_product` 
> (
> `prod_id`  varchar(50) NOT NULL COMMENT '商品ID',
> `category` varchar(50) NOT NULL COMMENT '种类',
> `price`    int         NOT NULL COMMENT '价格',
> PRIMARY KEY (`prod_id`)
> );
> 
> -- 插入商品数据
> INSERT INTO `tb_product`
> VALUES 
>     ('proda', 'catea', 100),
>     ('prodb', 'cateb', 200),
>     ('prodc', 'catec', 300),
>     ('prodd', 'cated', 400);
> 
> -- 创建店铺信息表
> CREATE TABLE `tb_store`
> (
> `store_id` varchar(50) NOT NULL COMMENT '店铺ID',
> `city`     varchar(20) NOT NULL COMMENT '城市',
> PRIMARY KEY (`store_id`)
> );
> 
> -- 插入店铺数据
> INSERT INTO `tb_store`
> VALUES 
>     ('storea', 'citya'),
>     ('storeb', 'citya'),
>     ('storec', 'cityb'),
>     ('stored', 'cityc'),
>     ('storee', 'cityd'),
>     ('storef', 'cityb');
> 
> -- 添加外键约束
> ALTER TABLE `tb_order_detail` ADD CONSTRAINT FOREIGN KEY (`product`) REFERENCES `tb_product` (`prod_id`);
> ALTER TABLE `tb_order_detail` ADD CONSTRAINT FOREIGN KEY (`store`) REFERENCES `tb_store` (`store_id`);
> ```
