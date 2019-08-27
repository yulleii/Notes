# 数据库操作

- 查询当前数据库 `show DATABASE；`
- 创建库 `CREATE DATABASE [IF NOT EXISTS] 数据库名 数据库选项`
  - 数据库选项：CHARACTER SET charset_name       COLLATE collation_name
- 删除库` DROP DATABASE[IF EXISTS]`
- 修改数据库信息：`ALTER DATABASE 库名 选项信息`

# 表的操作

- 创建表 

  - ` CREATE [TEMPORARY] TABLE[ IF NOT EXISTS] [库名.]表名 ( 表的结构定义 )[ 表选项]`

    -  每个字段必须有数据类型

    - 最后一个字段后不能有逗号

    - TEMPORARY 临时表，会话结束时表自动消失

    - 对于字段的定义：

      字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']

- 查看数据表结构

  - `SHOW COLUMNS FROM table_name`
  - `SHOW CREATE TABLE table_name`

- 修改表

  - 修改表本身

    - ALTER TABLE 表名 表的选项  eg：ALTER TABLE table_name ENGINE=MYISAM；

  - 对表进行重命名

    - RENAME TABLE 原表名 TO 新表名

  - 修改表的字段结构

    ~~~mysql
      ALTER TABLE 表名 操作名
            -- 操作名
                ADD[ COLUMN] 字段定义       -- 增加字段
                    AFTER 字段名          -- 表示增加在该字段名后面
                    FIRST               -- 表示增加在第一个
                ADD PRIMARY KEY(字段名)   -- 创建主键
                ADD UNIQUE [索引名] (字段名)-- 创建唯一索引
                ADD INDEX [索引名] (字段名) -- 创建普通索引
                DROP[ COLUMN] 字段名      -- 删除字段
                MODIFY[ COLUMN] 字段名 字段属性     -- 支持对字段属性进行修改，不能修改字段名(所有原有属性也需写上)
                CHANGE[ COLUMN] 原字段名 新字段名 字段属性      -- 支持对字段名修改
                DROP PRIMARY KEY    -- 删除主键(删除主键前需删除其AUTO_INCREMENT属性)
                DROP INDEX 索引名 -- 删除索引
                DROP FOREIGN KEY 外键    -- 删除外键
    ~~~

- 删除表

  - DROP TABLE table_name

- 清空表数据

  - TRUNCATE 表名



# 数据操作

~~~mysql
-- 增
    INSERT [INTO] 表名 [(字段列表)] VALUES (值列表)[, (值列表), ...]
        -- 如果要插入的值列表包含所有字段并且顺序一致，则可以省略字段列表。
        -- 可同时插入多条数据记录！
        REPLACE 与 INSERT 完全一样，可互换。
    INSERT [INTO] 表名 SET 字段名=值[, 字段名=值, ...]
-- 查
    SELECT 字段列表 FROM 表名[ 其他子句]WHERE where_condition
    								GROUP BY col_name [ASC|DESC]
    								Having where_condition
    								ORDER BY col_name ASC|DESC
    								LIMIT 
        -- 可来自多个表的多个字段
        -- 其他子句可以不使用
        -- 字段列表可以用*代替，表示所有字段
        LIMIT m,n : 表示从第m+1条开始，取n条数据；
		LIMIT n ： 表示从第0条开始，取n条数据，是limit(0,n)的缩写。
-- 删
    DELETE FROM 表名[ 删除条件子句]
        没有条件子句，则会删除全部
-- 改
    UPDATE 表名 SET 字段名=新值[, 字段名=新值] [更新条件]
~~~

~~~java

/* 连接查询(join) */ ------------------
    将多个表的字段进行连接，可以指定连接条件。
-- 内连接(inner join)
    - 默认就是内连接，可省略inner。
    - 只有数据存在时才能发送连接。即连接结果不能出现空行。
    on 表示连接条件。其条件表达式与where类似。也可以省略条件（表示条件永远为真）
    也可用where表示连接条件。
    还有 using, 但需字段名相同。 using(字段名)
    -- 交叉连接 cross join
        即，没有条件的内连接。
        select * from tb1 cross join tb2;
-- 外连接(outer join)
    - 如果数据不存在，也会出现在连接结果中。
    -- 左外连接 left join
        如果数据不存在，左表记录会出现，而右表为null填充
    -- 右外连接 right join
        如果数据不存在，右表记录会出现，而左表为null填充
-- 自然连接(natural join)
    自动判断连接条件完成连接。
    相当于省略了using，会自动查找相同字段名。
    natural join
    natural left join
    natural right join
~~~

# 注意

- 关于Distinct

  大表一般用distinct效率不高，大数据量的时候都禁止用distinct，建议用group by解决重复问题。

  select salary from salaries where to_date='9999-01-01' group by salary order by salary desc

- not in在实际使用中，因为not in会转化成多表连接，而且不使用索引，在这里，觉得还是用left_join代替会好一点

```mysql
`select e.emp_no from employees as e left join dept_manager as d on e.emp_no = d.emp_no``where d.dept_no is ``null`
```