### DDL
**DDL** 是 **Data Definition Language** 的缩写，意思是“**数据定义语言**”。  
**DDL 语句**就是用来**定义或修改数据库结构**的 SQL 语句，**不涉及数据的增删改查（那是 DML）**，只关心“库、表、列、索引、约束”这些结构本身。

---

 ✅ 常见的 MySQL DDL 语句包括：

| 操作类型      | 示例语句                                                         | 说明      |
| :-------- | :----------------------------------------------------------- | :------ |
| **创建数据库** | `CREATE DATABASE mydb;`                                      | 新建一个数据库 |
| **删除数据库** | `DROP DATABASE mydb;`                                        | 删除整个数据库 |
| **创建表**   | `CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50));` | 新建一张表   |
| **修改表结构** | `ALTER TABLE users ADD COLUMN age INT;`                      | 给表加字段   |
| **删除表**   | `DROP TABLE users;`                                          | 删除整张表   |
| **创建索引**  | `CREATE INDEX idx_name ON users(name);`                      | 给字段加索引  |
| **删除索引**  | `DROP INDEX idx_name ON users;`                              | 删除索引    |

---

 ❗注意：

- DDL 语句**自动提交事务**，不能回滚（在 MySQL 中，即使你用 `START TRANSACTION` 也一样）。
    
- 执行前要小心，尤其是 `DROP` 和 `ALTER`，可能会丢数据或锁表。

### DML
DML 是 **Data Manipulation Language** 的缩写，意思是“**数据操作语言**”。  
**DML 语句**负责**对表里的数据进行增、删、改、查**，**不改变表结构**，只折腾“里面的内容”。

---

 ✅ 常见的 MySQL DML 语句（就 4 条核心）

|操作|示例|说明|
|:--|:--|:--|
|**增**|`INSERT INTO users(id,name) VALUES (1,'张三');`|往表里插一行或多行数据|
|**删**|`DELETE FROM users WHERE id=1;`|按条件删除行，**不写 WHERE 会全表清空**|
|**改**|`UPDATE users SET name='李四' WHERE id=1;`|按条件修改已有行的列值|
|**查**|`SELECT * FROM users WHERE age>=18;`|把符合条件的数据读出来|

---
**DML 语句就是“折腾表里数据”的 SQL：插行、删行、改值、查结果——不动骨架，只动内容。**

### 常用语句
```shell
mysqldump -uroot -p123456 -R -E --default-character-set=utf8mb4 --set-gtid-purged=OFF dbname table_name > a.sql

mysql -uroot -p123456 --default-character-set=utf8mb4 -vvv -c dbname   < a.sql   > a.log

mysqldump -R -E -uroot -p123456  --default-character-set=utf8 dbname > dbname.bak
mysql -uroot -p123456 --default-character-set=utf8 -c dbname < dbname.bak

grant all on `dbname`.* to user@'%' identified by '123456';
grant all on `dbname`.* to user@'1.2.3.4' identified by '123456';

create user root@'%' identified by '123456';
grant all on *.* to root@'127.0.0.1';

```
