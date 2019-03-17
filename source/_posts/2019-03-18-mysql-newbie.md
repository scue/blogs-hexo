[TOC]

---

# MySQL  
  
## 提示符  
  
### 修改方法  
  
* `mysql -uroot -proot --prompt`  
* `mysql>prompt 提示符`  
  
### 参数  
  
* `\D`：完整日期  
    `\d`：当前数据库  
    `\h`：服务器名称  
    `\u`：当前用户名  
  
* 如：`\u@\h (\d)`  
  
## 存储引擎  
  
### 配置文件  
  
* `default-storage-engine=INNODB`  
  
## 数据类型  
  
### 整数  
  
* TINYINT  
    * 1字节  
* SMALLINT  
    * 2字节  
* MEDIUMINT  
    * 3字节  
* INT  
    * 4字节  
* BIGINT  
    * 8字节  
  
### 浮点数  
  
* FLOAT[(M,D)]  
    * `-3.402823466E+38 ~ -1.175494351E-38`、  
        0、  
        `1.175494351E-38 ~ 3.402823466E+38`  
  
* DOUBLE[(M,D)]  
    * `-1.7976931348623157E+308 ~ -2.2250738585072014E-308`、  
        0、  
        `2.2250738585072014E-308 ~ 1.7976931348623157E+308`  
  
* M表示数字总位数  
    D表示小数点之后的位数  
      
    M≥D总是成立  
  
### 日期时间  
  
* YEAR  
* TIME  
* DATE  
* DATETIME  
* TIMESTAMP  
  
### 字符型  
  
* CHAR(M)  
    * 0 ≤ M ≤ 255  
    * 定长类型，不足以空格补充  
* VARCHAR(M)  
    * 0 ≤ M ≤ 65535  
    * 可变长类型  
* TINYTEXT  
    * 2 ⁸  
* TEXT  
    * 2 ¹⁶  
* MEDIUMTEXT  
    * 2 ²⁴  
* LONGTEXT  
    * 2³²  
* ENUM(‘value1’, ‘value2’, ...)  
    * 1或2字节  
    * 枚举值个数最多65535  
* SET(‘value1’, ‘value2’, ...)  
    * 1、2、3、4或8字节  
    * 取决于set成员的数目（最多64个成员）  
    * …  
  
## 数据库  
  
### 查看数据库  
  
* SHOW DATABASES;  
  
### 切换数据库  
  
* USE 数据库名  
  
### 查看当前数据库  
  
* SELECT DATABASE();  
  
## 数据表  
  
### 创建数据表  
  
* ```  
    CREATE TABLE [IF NOT EXISTS] table_name (  
      column_name data_type,  
      ....  
    )  
    ```  
  
### 查看数据表  
  
* SHOW TABLES;  
* DESC table_name;  
* SHOW COLUMNS FROM tbl_name;  
* SHOW CREATE TABLE tbl_name;  
  
### 插入记录  
  
* INSERT [INTO] tbl_name [(col_name,...)] VALUES(val, ...)  
  
### 查看记录  
  
* SELECT expr,... FROM tbl_name  
* SELECT * FROM tb1;  
    * 思考：*表示筛选所有记录，还是表示筛选列？  
  
### 空值与非空  
  
* NULL，字段可以为空  
* NOT NULL，字段值禁止为空  
    * 非空约束¹  
* 默认允许为空  
  
### 自动编号  
  
* AUTO_INCREMENT  
* 必须与主键组合使用  
* 默认起始值为1，递增量为1  
  
### 查看索引  
  
* `SHOW INDEXES FROM users\G`  
  
### 主键约束²  
  
* PRIMARY KEY  
* 每张数据表只存在一个主键  
* 主键保证记录的唯一性  
* 主键自动为NOT NULL  
* 可以不与 AUTO_INCREMENT 一起使用  
  
### 唯一约束³  
  
* UNIQUE KEY  
* 唯一约束保证记录的唯一性  
* 唯一约束的字段可以为空值（NULL)  
* 一张数据表可以存在多个唯一约束  
  
### 默认约束⁴  
  
* DEFAULT  
* 默认值  
* 插入记录时，若无明确为字段赋值，则赋予默认值  
  
### 外键约束⁵  
  
* FOREIGN KEY  
    * 一般不使用它，通常使用逻辑外键约束  
* 保持数据一致性，完整性  
* 实现一对一、或一对多关系  
* 创建外键结束的要求  
    * 父表和子表必须使用相同的存储引擎  
    * 存储引擎只能为InnoDB  
    * 外键列和参照列必须具有相似的数据类型  
          
        - 其中数字的长度或是否有符号位必须相同  
        - 而字符的长度则可以不同  
  
    * 外键列和参照列必须创建索引  
          
        - 若外键列不存在索引的，将自动创建索引  
  
* 示例  
    * 省份  
        * ```sql  
            mysql> CREATE TABLE provinces (  
                -> id SMALLINT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,  
                -> pname VARCHAR(20) NOT NULL  
                -> );  
            ```  
  
    * 用户  
        * ```sql  
            mysql> CREATE TABLE users(  
                -> id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
                -> username VARCHAR(20) NOT NULL,  
                -> pid SMALLINT UNSIGNED,  
                -> FOREIGN KEY (pid) REFERENCES provinces (id)  
                -> );  
            ```  
              
            - 1）`pid SMALLINT UNSIGNED` 必须与provinces的id字段类型一致  
            - 2）`FOREIGN KEY (pid) REFERENCES provinces (id)`指明外键列、参照列  
            - 3）pid是外键列  
            - 4）provinces的id为参照列  
            - 5）pid外键列将自动创建索引  
  
        * 查看索引`SHOW INDEXES FROM users\G`  
            * ```sql  
                mysql> SHOW INDEXES FROM users\G  
                *************************** 1. row ***************************  
                        Table: users  
                   Non_unique: 0  
                     Key_name: PRIMARY  
                 Seq_in_index: 1  
                  Column_name: id  
                    Collation: A  
                  Cardinality: 0  
                     Sub_part: NULL  
                       Packed: NULL  
                         Null:  
                   Index_type: BTREE  
                      Comment:  
                Index_comment:  
                *************************** 2. row ***************************  
                        Table: users  
                   Non_unique: 1  
                     Key_name: pid  
                 Seq_in_index: 1  
                  Column_name: pid  
                    Collation: A  
                  Cardinality: 0  
                     Sub_part: NULL  
                       Packed: NULL  
                         Null: YES  
                   Index_type: BTREE  
                      Comment:  
                Index_comment:  
                2 rows in set (0.00 sec)  
                ```  
  
            * pid外键列已自动创建索引  
* 参照操作  
    * 使用方法  
        * - FOREIGN KEY (pid) REFERENCES provinces (id)  
            - FOREIGN KEY (pid) REFERENCES provinces (id) **ON DELETE CASCADE**  
  
    * CASCADE  
        * 从父表删除或更新，会自动删除或更新子表中匹配的行  
    * SET NULL  
        * 从父表删除或更新，会自动将子表的外键列为NULL（必须保证子表列没有指定NOT NULL）  
    * RESTRICT  
        * 拒绝对父表的删除或更新操作  
    * NO ACTION  
        * 标准SQL的关键字，与RESTRICT相同  
  
### 约束分类  
  
* 表级约束  
    * 对一个数据列建立的约束  
    * 只可在列定义后声明  
* 列级约束  
    * 对多个数据列建立的约束  
    * 可在列定义时声明，亦可在列定义后声明  
    * 举例  
        * NOT NULL  
        * DEFAULT  
  
### 列的操作  
  
* 添加列  
    * **ALTER TABLE** tbl_name **ADD** [COLUMN] col_name column_definition [FIRST | AFTER col_name]  
    * `ALTER TABLE users ADD password VARCHAR(20) NOT NULL DEFAULT "***" AFTER username;`  
* 删除列  
    * ALTER TABLE users DROP truename;  
* 添加约束  
    * 主键约束  
        * ALTER TABLE tbl_name ADD [CONSTRAINT [symbol]]  
            **PRIMARY KEY** [index_type] (index_col_name...)  
  
        * 示例：ALTER TABLE user2 ADD CONSTRAINT PK_user2_id PRIMARY KEY (id);  
    * 唯一约束  
        * ALTER TABLE tbl_name ADD [CONSTRAINT [symbol]]  
            **UNIQUE** [INDEX|KEY] [index_name] [index_type]  
            (index_col_name,..)  
  
        * 示例：ALTER TABLE user2 ADD UNIQUE (username);  
    * 外键约束  
        * ALTER TABLE tbl_name ADD [CONSTRAINT [symbol]]  
            **FOREIGN KEY** [index_name] (index_col_name,…)  
            reference_definition  
  
        * 示例：ALTER TABLE user2 ADD FOREIGN KEY (pid) REFERENCES provinces (id);  
    * 添加/删除默认约束  
        * ALTER TABLE tbl_name ALTER [COLUMN] col_name  
            [**SET DEFAULT** literal | **DROP DEFAULT**]  
  
        * 示例：ALTER TABLE user2 ALTER age SET DEFAULT 15;  
* 删除约束  
    * 主键约束  
        * ALTER TABLE tbl_name DROP PRIMARY KEY;  
        * 思考：为什么不需要指定名字？  
    * 唯一约束  
        * ALTER TABLE tbl_name DROP {INDEX|KEY} index_name;  
        * 示例： ALTER TABLE user2 DROP INDEX username;  
    * 外键约束  
        * ALTER TABLE tbl_name DROP FOREIGN KEY fk_symbol;  
        * 示例：ALTER TABLE user2 DROP FOREIGN KEY user2_ibfk_1;  
    * 默认约束  
        * ALTER TABLE tbl_name ALTER [COLUMN] col_name **DROP DEFAULT**  
        * 示例：ALTER TABLE user2 ALTER age DROP DEFAULT;  
  
### 修改列定义  
  
* ALTER TABLE tbl_name **MODIFY** [COLUMN] col_name  
    column_definition [FIRST | AFTER col_name]  
  
* 示例：  
      
    移动位置  
    ALTER TABLE user2 MODIFY id SMALLINT UNSIGNED NOT NULL FIRST;  
    数据类型  
    ALTER TABLE user2 MODIFY id TINYINT UNSIGNED NOT NULL;  
  
### 修改列名称  
  
* ALTER TABLE tbl_name **CHANGE** [COLUMN] old_col_name  
    new_col_name column_definition [FIRST | AFTER col_name]  
  
* 示例：ALTER TABLE user2 CHANGE pid p_id TINYINT UNSIGNED NOT NULL;  
  
### 修改表名称  
  
* ALTER TABLE tbl_name **RENAME** tbl_name_new;  
* **RENAME** TABLE tbl_name TO tbl_name_new;  
  
## 记录  
  
### 插入记录 INSERT  
  
* INSERT [INTO] tbl_name [(col_ name,...)] [VALUES|VALUE)  
    ({expr | DEFAULT},…),(…),…  
  
* 创建：  
      
    ```  
    mysql> CREATE TABLE users2(  
        -> id SMALLINT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT  
        -> username VARCHAR(20) NOT NULL,  
        -> password VARCHAR(20) NOT NULL,  
        -> age TINYINT UNSIGNED NOT NULL DEFAULT 10,  
        -> sex BOOLEAN);   
    ```  
  
* 示例1：INSERT users2 VALUES(NULL, 'Tom', '123', 25, 1);  
* 示例2：INSERT users2 VALUES(DEFAULT, 'Lily', '456', 25, 1);  
* 自增字段，可以使用NULL或DEFAULT来自动编号  
* 示例3：INSERT users2 VALUES(DEFAULT, 'Lucy', '456', 3*7+1, 1);  
* 字段值可以使用表达式  
* 示例4：INSERT users2 VALUES(DEFAULT, 'Lucy', '456', DEFAULT, 1);  
* 字段值可以DEFAULT以使用默认值  
* 示例5：INSERT users2 VALUES(DEFAULT, 'Lucy', '456', 3*7+1, 1),(NULL, 'Rose', md5('hello'), DEFAULT, 0);  
* 可以插入多个记录，可以使用函数  
  
### 插入记录 INSERT SET  
  
* INSERT [INTO] tbl_name SET col_name={expr | DEFAULT}, ...  
* 与第一种方式的区别  
    * 此方法可以使用子查询（SubQuery）  
    * 此方法只能一次性插入一条记录  
* 示例：INSERT users2 SET username='Ben', password='456';  
  
### 插入记录 INSERT SELECT  
  
* INSERT [INTO] tbl_name [(col_name,...)] SELECT ...  
* 此方法可以将查询结果插入到指定的数据表  
* 示例1：INSERT users2(username) SELECT username FROM users WHERE age >= 30;  
* 示例2  
    * ```  
        CREATE TABLE tdb_goods_cates (  
          cate_id smallint not null primary key auto_increment,  
          cate_name varchar(40) not null);  
        ```  
  
    * `INSERT tdb_goods_cates(cate_name) SELECT goods_cate FROM tdb_goods GROUP BY goods_cate;`  
  
### CREATE ... SELECT  
  
* 创建数据表同时将查询结果写入数据表  
* **CREATE TABLE [IF NOT EXISTS]** tbl_name  
    [(create_definition,...)]  
    select_statement  
  
* 示例1：  
    ```  
    CREATE TABLE tdb_goods_brands (  
      brand_id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,  
      brand_name VARCHAR(40) NOT NULL)  
      SELECT brand_name FROM tdb_goods GROUP BY brand_name;  
    ```  
  
### 更新记录 UPDATE（单表）  
  
* UPDATE [LOW_PRIORITY] [IGNORE] tbl_reference  
    SET col_name1={expr|DEFAULT} [, col_name2={expr|DEFAULT}] ...  
    [WHERE condition]  
  
* 省略where条件时，所有的记录都会更新  
* 示例1：UPDATE users2 SET age = age + 5;  
* 示例2：UPDATE users2 SET age = age - id, sex = 0;  
* 示例3：UPDATE users2 SET age = age + 10 WHERE id % 2 = 0;  
  
### 更新记录 UPDATE (多表)  
  
* **UPDATE** tbl_refs  
    **SET** col_name1={expr|DEFAULT}  
    [,col_name2={expr2|DEFAULT}] ...  
    [**WHERE** condition]  
  
* 指的是参照其他表更新自己的表  
* 示例1：  
    **UPDATE** tdb_goods **JOIN** tdb_goods_cates **ON** goods_cate = cate_name  
    **SET** goods_cate = cate_id;  
  
* 示例2：  
    **UPDATE** tdb_goods **AS** g **JOIN** tdb_goods_brands **AS** b  
    **ON** g.brand_name = b.brand_name  
    **SET** g.brand_name = b.brand_id;  
  
* ```  
    mysql> ALTER TABLE tdb_goods  
        -> CHANGE goods_cate cate_id SMALLINT UNSIGNED NOT NULL,  
        -> CHANGE brand_name brand_id SMALLINT UNSIGNED NOT NULL;  
    ```  
    接下来可以把 goods_cate和brand_name改为id的形式，并使用smallint来存储  
  
### 表的参照关系 (REFERENCE)  
  
* tbl_reference  
    {[INNER | CROSS] **JOIN** | {LEFT|RIGHT} [OUTER] JOIN}  
    tbl_reference  
    **ON** condition_expr  
  
* 连接类型  
    * INNER JOIN：内连接  
        * 最常用  
        * JOIN、CROSS JOIN和INNER JOIN是等价的  
        * 交集部分  
        * 示例：  
            **SELECT** goods_id,goods_name,cate_name **FROM** tdb_goods  
            **JOIN** tdb_goods_cates  
            **ON** tdb_goods.cate_id = tdb_goods_cates.cate_id;  
  
    * LEFT [OUTER] JOIN：左外连接  
        * 显示左表全部记录，右表符合条件的记录  
        * 左集  
        * 示例：  
            **SELECT** goods_id,goods_name,cate_name **FROM** tdb_goods  
            **LEFT JOIN** tdb_goods_cates  
            **ON** tdb_goods.cate_id = tdb_goods_cates.cate_id;  
  
        * 特点  
            * 数据表B的结果依赖于数据表A  
            * 左外连接条件决定如何检索B（未指定WHERE条件下）  
            * 若数据表A某条记录符合WHERE条件，而数据表B不存在符合连接条件的，将产生额外所有列为NULL的额外B行（即不存在的以NULL表示）  
                * 若恰巧此字段定义为NOT NULL，则停止继续搜索  
    * RIGHT [OUTER] JOIN: 右外连接  
        * 显示右表全部记录，左表符合条件的记录  
        * 右集  
* 多表连接  
    * 和两张表连接没区别  
    * 示例：  
        **SELECT** goods_id,goods_name,cate_name,brand_name **FROM** tdb_goods  
          **JOIN** tdb_goods_cates **ON** tdb_goods_cates.cate_id = tdb_goods.cate_id  
          **JOIN** tdb_goods_brands **ON** tdb_goods_brands.brand_id = tdb_goods.brand_id;  
  
* 自身连接  
    * 涉及到无限分类设计时，需要自身连接  
    * **SELECT** s.type_id,s.type_name,p.type_name  
        **FROM** tdb_goods_types **AS** s  
        **LEFT** **JOIN** tdb_goods_types **AS** p **ON** s.parent_id = p.type_id;  
  
    * …  
  
### 无限分类设计  
  
* ```  
    mysql> CREATE TABLE tdb_goods_types (  
        ->type_id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,  
        -> type_name VARCHAR(40) NOT NULL,  
        -> parent_id SMALLINT UNSIGNED NOT NULL DEFAULT 0);  
    ```  
    使用一个parent_id指向父级分类  
  
### 删除记录 DELETE (单表)  
  
* DELETE FROM tbl_name [WHERE condition]  
* 省略where条件时，所有的记录都会删除  
  
### 删除记录 DELETE (多表)  
  
* DELETE tbl_name[.*] [, tbl_name[.*]] ...  
    FROM tbl_refs  
    [WHERE condition]  
  
* 示例：（删除自身表重复记录）  
    ```  
    DELETE t1 FROM tdb_goods AS t1   
     LEFT JOIN (SELECT goods_id,goods_name FROM tdb_goods GROUP BY goods_name HAVING count(goods_name) > 1) AS t2  
     ON t1.goods_name = t2.goods_name   
     WHERE t1.goods_id > t2.goods_id;  
    ```  
    - 使用子查询结果作为数据表  
    - 连接条件是两个商品名字相同  
    - 删除id较大的记录  
  
### 查询记录 SELECT  
  
* ```  
    SELECT select_expr [, select_expr ...]  
    [  
      FROM tbl_references  
      [WHERE condition]  
      [GROUP BY {col_name|position} [ASC|DESC], ...]  
      [HAVING where_condition]  
      [ORDER BY {col_name|expr|position} [AEC|DESC], ...]  
      [LIMIT {[offset,] row_count | row_count OFFSET offset}]  
    ]  
    ```  
  
* 特殊用法  
    * SELECT VERSION();  
        * 查看版本  
    * SELECT NOW();  
        * 查看时间  
    * SELECT 3 + 5;  
        * 只运算  
* SELECT id,username FROM users;  
    - 可以只查询所需字段  
  
* SELECT username,id FROM users;  
    - 可以不按原字段的顺序  
  
* SELECT * FROM users;  
    - 查询所有字段  
  
* SELECT users.id,users.username FROM users;  
    - 可以加上数据表的名字  
  
* SELECT id AS userId,username AS uname FROM users;  
    - 字段可以设定别名  
  
    * AS可以省略  
    * 思考：为什么使用别名的建议使用AS关键词？  
    * 思考：`SELECT id username FROM users;`会输出什么？  
* 条件查询 WHERE  
    * 可以使用MySQL支持的函数和运算符  
* 结果分组 GROUP BY  
    * GROUP BY {col_name | position} [ASC | DESC], ...  
    * 按字段指定  
        * SELECT sex FROM users GROUP BY sex;  
    * 按位置指定  
        * SELECT sex FROM users GROUP BY 1;  
        * 1表示SELECT的第1个字段，即这里的sex  
* 分组条件 HAVING  
    * HAVING where_condition  
    * SELECT sex FROM users GROUP BY sex HAVING age > 10;   
        - 出错  
  
    * SELECT sex,age FROM users GROUP BY sex HAVING age > 10;   
        - age需要出现在SELECT的字段内  
  
    * SELECT sex FROM users GROUP BY sex HAVING count(id) > 2;   
        - id出现在聚合函数内  
  
* 结果排序 ORDER BY  
    * ORDER BY {col_name|expr|position} [ASC|DESC]  
    * SELECT * FROM users ORDER BY id DESC;  
        - 按id字段排序  
  
    * SELECT * FROM users ORDER BY age,id DESC;  
        - 按两字段进行排序  
  
    * SELECT * FROM users2 ORDER BY age DESC,id ASC;  
        - 按两字段进行排序  
  
* 限制数量 LIMIT  
    * LIMIT {[offset,] row_count | row_count OFFSET offset}  
    * SELECT * FROM users LIMIT 2,2;  
        - 从第3条记录开始，共2条数据被展示  
  
## 子查询  
  
### 指出现在其他SQL语句内的SELECT语句  
  
### 术语  
  
* 外层查询：Outer Query，Outer Statement  
* 子查询：SubQuery  
* 查询：所有SQL语言的统称，而非「仅查找」  
  
### 特点  
  
* 外层查询可以是SELECT、INSERT、UPDATE、SET或DO  
* 必须始终出现在圆括号内  
* 子查询可返回标量、一行、一列或**子查询**  
  
### 分类  
  
* 使用比较运算符的子查询  
    * =, >, <, >=, <=, <>, !=, <=>  
          
        - 1）`<>`和`!=`效果一样，历史遗留  
        - 2）`<=>`是等于的意思，通常用于判空`WHERE a <=> NULL`  
  
    * SELECT goods_id,goods_name,goods_price FROM tdb_goods   
        WHERE goods_price >=   
        _(SELECT ROUND(AVG(goods_price),2) FROM tdb_goods);_  
  
    * …  
    * SELECT goods_id,goods_name,goods_price FROM tdb_goods  
        WHERE goods_price >= **ANY**  
        _(SELECT goods_price FROM tdb_goods WHERE goods_cate = '超级本');_  
  
* 使用 [NOT] IN的子查询  
    * NOT IN, IN  
    * SELECT goods_id,goods_name,goods_price FROM tdb_goods  
        WHERE goods_price **IN**   
        _(SELECT goods_price FROM tdb_goods WHERE goods_cate = '超级本');_  
  
* 使用[NOT] EXISTS的子查询  
    * NOT EXISTS, EXISTS  
    * 若子查询返回任何行，EXISTS将返回TRUE，反之为FALSE  
    * 不常用  
  
## 运算符与函数  
  
### 字符函数  
  
* CONCAT(): 字符连接  
    * 示例1：SELECT CONCAT('imooc', 'mysql');  
    * 示例2：SELECT CONCAT(goods_id, goods_name) AS goods FROM tdb_goods;  
* CONCAT_WS(): 使用指定分隔符进行字符连接  
    * SELECT CONCAT_WS('|', goods_id, goods_name) AS goods  
        FROM tdb_goods;  
  
* FORMAT(): 数字格式化  
    * mysql> SELECT FORMAT(123456.7890, 2);  
        +------------------------+  
        | FORMAT(123456.7890, 2) |  
        +------------------------+  
        | 123,456.79             |  
        +------------------------+  
        1 row in set (0.01 sec)  
  
* LOWER(): 转换小写字母  
* UPPER(): 转换大写字母  
* LEFT(): 取左侧字符  
* RIGHT(): 取右侧字符  
* LENGTH(): 字符串长度（包含空格）  
* LTRIM(): 删除左侧空格  
* RTRIM(): 删除右侧空格  
* TRIM(): 删除左右侧空格  
    * 示例1：删除空格  
        * mysql> SELECT TRIM('  MYSQL  ');  
            +-------------------+  
            | TRIM('  MYSQL  ') |  
            +-------------------+  
            | MYSQL             |  
            +-------------------+  
            1 row in set (0.00 sec)  
  
    * 示例2：删除前导指定的’?’字符  
        * mysql> SELECT TRIM(LEADING '?' FROM '??MYSQL???');  
            +-------------------------------------+  
            | TRIM(LEADING '?' FROM '??MYSQL???') |  
            +-------------------------------------+  
            | MYSQL???                            |  
            +-------------------------------------+  
            1 row in set (0.00 sec)  
  
    * 示例3：删除后续指定的’?’字符  
        * mysql> SELECT TRIM(TRAILING '?' FROM '??MYSQL???');  
            +--------------------------------------+  
            | TRIM(TRAILING '?' FROM '??MYSQL???') |  
            +--------------------------------------+  
            | ??MYSQL                              |  
            +--------------------------------------+  
            1 row in set (0.00 sec)  
  
    * 示例4：删除两头指定的’?’字符  
        * mysql> SELECT TRIM(BOTH '?' FROM '??MYSQL???');  
            +----------------------------------+  
            | TRIM(BOTH '?' FROM '??MYSQL???') |  
            +----------------------------------+  
            | MYSQL                            |  
            +----------------------------------+  
            1 row in set (0.00 sec)  
  
* SUBSTRING(str, start, len): 截取指定长度子字符串  
    *  mysql> SELECT SUBSTRING('MYSQL', 1, 2);  
        +--------------------------+  
        | SUBSTRING('MYSQL', 1, 2) |  
        +--------------------------+  
        | MY                       |  
        +--------------------------+  
        1 row in set (0.00 sec)  
  
    * 值得注意的是一般编程是从0开始，而MySQL从1开始  
    * start起始可以是负值，倒过来计算起始位置  
* [NOT] LIKE  
    * 示例1：  
        mysql> SELECT 'MYSQL' LIKE 'M%';  
        +-------------------+  
        | 'MYSQL' LIKE 'M%' |  
        +-------------------+  
        |                 1 |  
        +-------------------+  
        1 row in set (0.00 sec)  
  
    * 示例2:   
        mysql> SELECT * FROM tdb_goods_types WHERE type_name LIKE '%P%';  
        +---------+-----------+-----------+  
        | type_id | type_name | parent_id |  
        +---------+-----------+-----------+  
        |      14 | CPU       |        10 |  
        +---------+-----------+-----------+  
        1 row in set (0.00 sec)  
  
    * 示例3：包含有%的查找方法  
        mysql> INSERT tdb_goods_types (type_name) VALUES ('TOM%');  
        Query OK, 1 row affected (0.01 sec)  
          
        mysql> **SELECT** * **FROM** tdb_goods_types **WHERE** type_name **LIKE** '%1%%' **ESCAPE** '1';  
        +---------+-----------+-----------+  
        | type_id | type_name | parent_id |  
        +---------+-----------+-----------+  
        |      16 | TOM%      |         0 |  
        +---------+-----------+-----------+  
        1 row in set (0.00 sec)  
          
        - ESCAPE ‘1’：表示1后边的%不用作通配符  
  
* REPLACE()：替换指定字符串  
    * mysql> SELECT REPLACE('??MYSQL???', '?', '');  
        +--------------------------------+  
        | REPLACE('??MYSQL???', '?', '') |  
        +--------------------------------+  
        | MYSQL                          |  
        +--------------------------------+  
        1 row in set (0.00 sec)  
  
### 数值运算符与函数  
  
* CEIL(): 向上取整  
* FLOOR(): 向下取整  
* DIV: 整数除法  
* MOD: 取余数  
* POWER(): 幂运算  
* ROUND(): 四舍五入  
* TRUNCATE(): 数字截断  
  
### 比较运算符与函数  
  
* [NOT] BETWEEN ... AND ...  
    * 闭合区间  
    * mysql> SELECT 15 BETWEEN 10 AND 15;  
        +----------------------+  
        | 15 BETWEEN 10 AND 15 |  
        +----------------------+  
        |                    1 |  
        +----------------------+  
        1 row in set (0.00 sec)  
  
* [NOT] IN()  
    * 集合  
* IS [NOT] NULL  
  
### 日期时间函数  
  
* NOW()：当前日期与时间  
    * mysql> select now();  
        +---------------------+  
        | now()               |  
        +---------------------+  
        | 2019-03-17 10:01:32 |  
        +---------------------+  
        1 row in set (0.00 sec)  
  
* CURDATE()：当时日期  
    * mysql> select curdate();  
        +------------+  
        | curdate()  |  
        +------------+  
        | 2019-03-17 |  
        +------------+  
        1 row in set (0.00 sec)  
  
* CURTIME()：当期时间  
    * mysql> select curtime();  
        +-----------+  
        | curtime() |  
        +-----------+  
        | 10:01:37  |  
        +-----------+  
        1 row in set (0.00 sec)  
  
* DATE_ADD()：日期变化，可增可减  

    ```sql
    mysql> SELECT DATE_ADD('2019-3-17', INTERVAL 365 DAY);  
    +-----------------------------------------+  
    | DATE_ADD('2019-3-17', INTERVAL 365 DAY) |  
    +-----------------------------------------+  
    | 2020-03-16                              |  
    +-----------------------------------------+  
    1 row in set (0.00 sec)  
  
    mysql> SELECT DATE_ADD('2019-3-17', INTERVAL -365 DAY);  
    +------------------------------------------+  
    | DATE_ADD('2019-3-17', INTERVAL -365 DAY) |  
    +------------------------------------------+  
    | 2018-03-17                               |  
    +------------------------------------------+  
    1 row in set (0.01 sec)  
    ```
  
* DATEDIFF()  

    ```sql
    mysql> select datediff('2019-03-17', ' 2018-03-17');  
    +---------------------------------------+  
    | datediff('2019-03-17', ' 2018-03-17') |  
    +---------------------------------------+  
    |                                   365 |  
    +---------------------------------------+  
    1 row in set (0.04 sec)  
    ```
  
* DATE_FORMAT()  

    ```sql
    mysql> select date_format('2019-03-17', '%m/%d/%Y');  
    +---------------------------------------+  
    | date_format('2019-03-17', '%m/%d/%Y') |  
    +---------------------------------------+  
    | 03/17/2019                            |  
    +---------------------------------------+  
    1 row in set (0.00 sec)1  
    ```
  
### 信息函数  
  
* CONNECTIOIN_ID():连接ID  
* DATEBASE(): 当前数据库  
* LAST_INSERT_ID(): 最后插入记录ID  
    * 注意：当同时插入多条记录时，last_insert_id()返回是最后操作时第一条记录的ID  
* USER(): 当前用户  
* VERSION(): 当前版本  
  
### 聚合函数  
  
* AVG()  
* COUNT()  
* MAX()  
* MIN()  
* SUM()  
  
### 加密函数  
  
* MD5()：信息摘要算法  
* PASSWORD()：密码算法  
    * 设定当前用户密码：  
        SET PASSWORD=PASSWORD('dimitar');  
  
## 自定义函数  
  
### User-defined function, UDF  
  
### **CREATE FUNCTION** func_name  
  
**RETURNS**  
{STRING|INTEGER|REAL|DECIMAL}  
routine_body  
  
### **DROP FUNCTION [IF EXISTS]** func_name  
  
### 函数体（routine_body)  
  
* 由合法的SQL语句构成  
* 可以是简单的SELECT或INSERT语句  
* 若为复合结构则使用BEGIN...END语句  
* 复合结构可包含声明、循环和控制结构  
  
### 示例1：中文格式日期时间  

```sql
mysql> CREATE FUNCTION now_zh() RETURNS VARCHAR(30) RETURN DATE_FORMAT(NOW(), '%Y年%m月%d日 %H时:%i分:%s秒');  
Query OK, 0 rows affected (0.01 sec)  
  
mysql> select now_zh();  
+-------------------------------------+  
| now_zh()                            |  
+-------------------------------------+  
| 2019年03月17日 10时:30分:36秒       |  
+-------------------------------------+  
1 row in set (0.00 sec)  
```
  
### 示例2：带参数  

```sql
mysql> CREATE FUNCTION f2(a SMALLINT UNSIGNED, b SMALLINT UNSIGNED) RETURNS FLOAT(10,2) UNSIGNED RETURN (a+b)/2;  
Query OK, 0 rows affected (0.01 sec)  
  
mysql> select f2(10, 9);  
+-----------+  
| f2(10, 9) |  
+-----------+  
|      9.50 |  
+-----------+  
1 row in set (0.01 sec)  
```
  
### 示例3：复合结构体  

```  
mysql> DELIMITER //  
mysql> CREATE FUNCTION adduser(name VARCHAR(20))  
    -> RETURNS INT UNSIGNED  
    -> BEGIN  
    -> INSERT users3(name) VALUES(name);  
    -> RETURN LAST_INSERT_ID();  
    -> END//  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> DELIMITER ;  
mysql> SELECT adduser('hello');  
+------------------+  
| adduser('hello') |  
+------------------+  
|                2 |  
+------------------+  
1 row in set (0.00 sec)  
```
  
## 存储过程 (PROCEDURE)  
  
### …  
  
### 存储过程：是SQL语句和控制语句的**预编译**集合，  
  
以一个名称存储并作为一个单元处理  
  
### 特点  
  
* **预编译**，省略每次解析、编译的过程  
* 仅第一次进行SQL解析、编译  
  
### 优点  
  
* 增强SQL语句的功能和灵活性  
* 实现较快的执行速度  
* 减少网络流量  
  
### 定义  
  
* **CREATE**  
    [DEFINER = {user | CURRENT_USER}]  
    **PROCEDURE** sp_name ([proc_parameter[,...]])  
    [characteristic ...] routine_body  
      
    proc_parameter:  
    [IN | OUT | INOUT] param_name type  
  
### 调用  
  
* CALL sp_name[()]  
* CALL sp_name([parameter[,...]])  
  
### 删除  
  
* DROP PROCEDURE sp_name  
  
### 参数(proc_parameter)  
  
* IN：必须在调用时指定  
* OUT: 表示可以被改变，并且可返回  
* INOUT: 必须调用时指定，表示可改变和返回  
  
### 特性(characteristic)  
  
* COMMENT string  
    | {CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }  
    | SQL SECURITY {DEFINER | INVOKER}  
  
* COMMET: 注释  
* CONTAINS SQL: 包含SQL语句，但不包含读或写数据的语句  
* NO SQL: 不包含SQL语句  
* READS SQL DATA: 包含读数据的语句  
* MODIFIES SQL DATA: 包含写数据的语句  
* SQL SECURITY {DEFINER | INVOKER}: 指明谁有权限来执行  
  
### 过程体(routine_body)  
  
* 由合法的SQL语句构成  
* 可以是任意的SQL语句  
* 可以是复合结构 BEGIN ... END 语句  
* 复合结构可以包含声明、循环和控制结构  
  
### 示例1：无参数的存储过程  
  
* CREATE PROCEDURE sp1() SELECT VERSION();  

    ```sql
    mysql> CALL sp1();  
    +-----------+  
    | VERSION() |  
    +-----------+  
    | 5.7.20    |  
    +-----------+  
    1 row in set (0.00 sec) 
    ``` 
  
### 示例2：带IN类型参数存储过程  
  
```sql
mysql> DELIMITER //  
mysql> CREATE PROCEDURE sp2(IN id INT UNSIGNED)  
    -> BEGIN  
    -> DELETE FROM users3 WHERE id = id;  
    -> END//  
Query OK, 0 rows affected (0.00 sec)  
mysql> CALL sp2(1)//  
Query OK, 2 rows affected (0.01 sec)  
  
mysql> SELECT * from users3;  
Empty set (0.00 sec)
```  
  
* 由于id与表中的id混淆，导致整表数据被删除
* 因此，参数名尽量不要与字段名相同，正确做法如下
  
```sql
mysql> DELIMITER //  
mysql> CREATE PROCEDURE sp2(IN user_id INT UNSIGNED)  
    -> BEGIN  
    -> DELETE FROM users3 WHERE id = user_id;  
    -> END//  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> call sp2(3);//  
Query OK, 1 row affected (0.00 sec)
```  
  
### 示例3：带IN和OUT类型参数存储过程  

```  
mysql> delimiter //  
mysql> CREATE PROCEDURE sp3(IN user_id INT UNSIGNED, OUT nums INT UNSIGNED)  
    -> BEGIN  
    -> DELETE FROM users3 WHERE id = user_id;  
    -> SELECT count(id) FROM users3 INTO nums;  
    -> END//  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> DELIMITER ;  
  
mysql> SELECT count(id) FROM users3;  
+-----------+  
| count(id) |  
+-----------+  
|         2 |  
+-----------+  
1 row in set (0.00 sec)  
mysql> CALL sp3(4, @nums);  
Query OK, 1 row affected (0.00 sec)  
  
mysql> SELECT @nums;  
+-------+  
| @nums |  
+-------+  
|     1 |  
+-------+  
1 row in set (0.00 sec)  
```
  
- **INTO nums** 是把结果赋值给nums  
- **@nums** 表示在SQL语句中定义一个变量，用于接收返回值  
- **SELECT @nums** 表示显示这个变量的值  
- **SET @i = 7** 也可以设定变量，同样也是用户变量（与客户端绑定的，以@开头）  
  
### 示例4：带IN和多个OUT类型参数存储过程  

```sql
mysql> CREATE PROCEDURE sp4(IN username VARCHAR(20), OUT d_nums SMALLINT UNSIGNED, OUT r_nums SMALLINT UNSIGNED)  
    -> BEGIN  
    -> DELETE FROM users3 WHERE name = username;  
    -> SELECT ROW_COUNT() INTO d_nums;  
    -> SELECT COUNT(id) FROM users3 INTO r_nums;  
    -> END//  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> DELIMITER ;  
```

```sql
mysql> CALL sp4('lily', @d_nums, @r_nums);  
Query OK, 1 row affected (0.00 sec)  
  
mysql> SELECT @d_nums, @r_nums;  
+---------+---------+  
| @d_nums | @r_nums |  
+---------+---------+  
|       1 |       3 |  
+---------+---------+  
1 row in set (0.00 sec)  
```
  
### 与自定义函数区别  
  
* - 存储过程实现的功能要复杂一些  
    - 函数的针对性更强  
  
* - 存储过程可以返回多个值  
    - 函数只能返回一个值  
  
* - 存储过程可以独立执行  
    - 函数可作为其他SQL语句组成部分来出现  
  
## 存储引擎  
  
### 定义：以不同的技术存储在文件（内存）中，这种技术就称为存储引擎  
  
### 引擎  
  
* MyISAM  
    * 适用于事务的处理不多的情况  
    * 存储可达256TB  
    * 支持索引  
    * 表级锁定  
    * 数据压缩  
* InnoDB  
    * 适用于事务处理比较多，需要外键支持的情况  
    * 存储可达64TB  
    * 支持事务和索引  
    * 锁颗粒为行锁  
* Memory  
* Archive  
* CSV  
    * 每一个表都是一个csv文本文件，不支持索引  
* BlackHole  
    * 黑洞引擎，写入数据都会消失，一般用于数据复制的中继  
  
### 区别  
  
* …  
  
### 锁  
  
* 共享锁（读锁）  
* 排他锁（写锁）  
  
### 锁颗粒  
  
* 表锁：一种开销最小的锁策略  
* 行锁：一种开销最大的锁策略  
  
### 事务处理  
  
* 事务用于保证数据库的**完整性**  
* 特性  
    * 原子性（Atomicity）  
    * 一致性（Consistency）  
    * 隔离性（Isolation）  
    * 持久性（Durability）  
  
### 外键  
  
* 是保证数据一致性的策略  
* 只有InnoDB支持外键  
  
### 索引  
  
* 对数据表中一列或多列的值进行排序的一种结构  
* 分类：普通索引、唯一索引、全文索引、btree索引、hash索引...  
  
### 配置方法  
  
* 配置默认引擎  
    * default-storage-engin = engine  
* 创建表时定义  
    * CREATE TABLE tbl_name (...) ENGINE = engine;  
* 修改数据表  
    * ALTER TABLE tbl_name ENGINE [=] engine;  
  
## 管理工具  
  
### PhpMyAdmin  
  
### Navicat  
  
### MySQL Workbench  
  
## 小技巧  
  
### SELECT * FROM tdb_goods\G;  
  
- 其中的`\G`表示以网络显示  
  
### SET NAMES gbk;  
  
- 表示客户端使用GBK编码  
- 不影响数据库的编码，通常用于Win终端  
  
### [**同样是条件ON与WHERE有什么区别吗**][1]  
  
* 通常使用ON设定连接条件  
* 使用WHERE过滤结果集  
  
### 修改结束符  
  
* DELIMITER //  
* 结束符从’;’改为了’//’  
  
[1]: https://www.imooc.com/qadetail/266469  
