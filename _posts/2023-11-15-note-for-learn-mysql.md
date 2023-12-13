---
title: 数据库学习笔记
date: 2023-11-15 02:52:57 +0800
categories: [笔记, 数据库]
tags: [Database, mysql]
---

## MySQL 基本语法

### SELECT

```sql
SELECT *
FROM MOCK_DATA
WHERE points>100
ORDER BY points desc
LIMIT 100
```

#### JOIN

`INNER JOIN` 根据两个表的关联字段查询, 得到一张包含全部字段的表, 也可以指定要查询的字段. `INNER` 关键字可以省略, 直接写成`JOIN`

连接多个表使用多个 `join ... on ...` 语句即可.

```sql
SELECT *
FROM mock_orders o
JOIN mock_user_profile u
ON u.id = o.user_id
```

`self joinn`, 通常用在一个表中包含一个字段是表中另外一条记录的主键时使用. 比如一个人员信息表, 每个人有独立的id, 同时有一个字段记录父亲的id等等.

```sql
SELECT 
    t1.id as user_id, 
    t1.name as user_name, 
    t2.name as faher
FROM user_info t1
JOIN user_info t2
ON t1.fater_id = t2.id
```

#### OUTER JOIN

外联查询, 包含左外联`LEFT OUTER JOIN`和右外联`RIGHT OUTER JOIN`两种, `OUTER`关键字可以省略

```sql
SELECT *
FROM mock_user_profile u
LEFT JOIN mock_orders o
    on u.id = o.user_id
```

左外联的意思是, 无论关联条件是否满足, 左边的表, 即主表, `FROM`关键字后边的表, 上例中的`u`, 数据全部显示, 不符合条件的数据字段空置
右外联则是链接表, `JOIN`关键字后边的表, 数据全部显示.

#### USING

如果两个表包含同名字段, 可以使用`USING`关键字进行链接, 省略`ON`子句, 相当于指定两个表的某个字段相等

```sql
SELECT * 
FROM users
JOIN user_profile USING (id)
```

#### CROSS JOIN

把两个表的所有数据进行交叉匹配, 比较少用, 在类似于创建组合的时候会用到, 比如用所有的尺码型号匹配所有的颜色.

当有5种尺码, 3种颜色, 就会有15条插叙年数据.

```sql
SELECT *
FROM sizes
CROSS JOIN colors
```

#### UNION

将两个查询的数据上下合并到同一个表里, 要求字段数量必须相同, 类型可以不同.

```sql
SELECT u.first_name, u.points
FROM mock_user_profile u
UNION
SELECT c.id, c.title
FROM store_collection c
```

### INSERT INTO

在不指定插入列的情况下, 数据的数量必须与表定义相同.

`DEFAULT`: 代表默认值, 如对自增主键来说, 默认值为 +1
`NULL`: 字段空值

```sql
INSERT INTO store_collection
VALUE (DEFAULT,'new collection', NULL)
```

或者指定要插入数据的字段名, 省略的字段必须存在默认值, 同时`VALUES`关键字可以一次加入多条数据.

```sql
INSERT INTO store_collection (title)
VALUES  ('new collection title1'),
        ('new collection title2'),
        ('new collection title3')
```

也可以将某个表中的数据整体插入到另一个表中, 常用来进行过期数据归档

```sql
INSERT INTO store_collection_archived
SELECT * FROM store_collection WHERE store_collection.id < 10
```

### CREATE TABLE AS

通过子查询创建新表, 可以用于数据表的备份, 归档, 也可以将查询的结果集独立保存等.

```sql
CREATE TABLE collection_archived AS 
    SELECT * from store_collection
```

### UPDATE

`SET`子句中, 多个值之间用`,`隔开
`WHERE`子句中, 多个条件之间用`AND`隔开

```sql
UPDATE store_product
SET title='update title', unit_price=299.99
WHERE id=1000;
```

也可以利用查询子句动态判断查询条件, 一个稍微复杂的更新例子

订单表保存订单的基本信息, 订单项表保存了每个订单包含的产品和购买数量.

根据订单号, 查询到具体的产品, 并根据订单数量更新产品的库存数据.

```sql
UPDATE store_product
SET inventory = inventory - (
        SELECT oi.quantity
        FROM store_order o
        JOIN store_orderitem oi
        WHERE o.id=oi.order_id and o.id=357
    )
WHERE id in
    (SELECT oi.product_id
    FROM store_order o
    JOIN store_orderitem oi
    WHERE o.id=oi.order_id and o.id=357
    );
```

> 作为假想需求和练习代码, 只是在逻辑上可以执行, 并非真实解决方案.
>
{: .prompt-info}

> 实际运行中可能会出错, 因为订单包含的产品可能不止一个, 但是MySQL默认执行`SafeUpdate`, 一次仅能更新一条数据. 可以根据不同的IDE进行设置.
>
{: .prompt-tip}

### DELETE

```sql
DELETE FROM store_product
WHERE id > 2000
```

## 统计函数

MySQL提供了一下函数用来方便的进行数据统计

```sql
SELECT
    MAX(unit_price),                --最大值
    MIN(unit_price),                --最小值
    AVG(unit_price),                --平均值
    SUM(unit_price * inventory),    --总和
    COUNT(description),             --总数, 仅统计非空值
    COUNT(*)                        --总数, 统计总行书
FROM store_product
WHERE unit_price > 50;
```

### GROUP BY

配合统计函数, 根据特定字段进行分组

```sql
SELECT collection_id, SUM(unit_price * inventory), COUNT(id)
FROM store_product
GROUP BY collection_id
```

> 计算每个分类的产品总数和总值
>
{: .prompt-info}

### HAVING

作用于WHERE子句类似, 区别在于`HAVING`子句用来在分组结束之后进行条件判断

```sql
SELECT collection_id, SUM(unit_price * inventory), COUNT(id) as product_count
FROM store_product
WHERE unit_price > 20
GROUP BY collection_id
HAVING product_count > 200
```

> 人话: 在单价大于20的产品里, 统计各个分类的产品总值和产品数量, 看看有没有数量大于200的
>
{: .prompt-info}

## 子查询

可以将一个查询语句的结果作为参数传递给另外一个查询的表达式

```sql
SELECT
    id, title, unit_price
FROM store_product
WHERE unit_price > (
        SELECT MAX(unit_price)
        FROM store_product
        WHERE title REGEXP 'coffe'
    );
```

> 人话: 列出比价格最高的咖啡还贵的产品的id,名字和价格.
>
{: .prompt-info}

```sql
SELECT *
FROM store_product
WHERE id in (
        SELECT DISTINCT product_id
        FROM store_orderitem
    )
```

> 人话: 找到所有被下过订单的产品
>
{: .prompt-info}

```sql
SELECT *
FROM store_customer
WHERE id in (
        SELECT o.customer_id
        FROM store_order o
        JOIN store_orderitem oi
        ON o.id = oi.order_id
        WHERE oi.product_id = 616
    )
```

> 人话: 所有曾经买过id为 `616` 的产品的用户信息
>
{: .prompt-info}
