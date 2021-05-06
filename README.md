# 元数据管理器实现

## 实现方案

- 使用csv文件进行表的结构存储
- 使用python、pandas读写csv，以dataframe作为数据结构进行增删查改

## 设计

### 表的构建

- columns表

  ```csv
  table_schema,table_name,column_name,ordinal_position,default_value,nullable,data_type,max_char_length,auto_increment,column_key
  test_db,tasks,task_id,1,,,int,,True,PRI
  test_db,tasks,subject,2,,,varchar,,False,
  test_db,tasks,start_date,3,,,date,,False,
  test_db,tasks,end_date,4,,,date,,False,
  test_db,tasks,description,5,,,varchar,,False,
  test_db,tasks,new_column,6,,0.0,,,False,
  ```

  

- schema表

  ```
  db_name,charset,collate
  ```

- tables表

  ```
  table_schema,table_name,table_type,engine,table_rows,create_time,auto_increment,update_time,table_collation
  test_db,tasks,base_table,innodb,0,2021-05-05 17:43:18,,2021-05-05 17:43:18,utf8
  ```

## SQL解析进度

### Create

- 创建数据库

  ```mysql
  create database test_db
  ```

- 创建数据表

  格式：

  ```
  CREATE TABLE <表名> ([表定义选项])[表选项][分区选项];
  [表定义选项]: <列名1> <类型1> [,…] <列名n> <类型n>
  ```

  ```mysql
  CREATE TABLE tasks (
    task_id INT(11) NOT NULL AUTO_INCREMENT,
    subject VARCHAR(45) DEFAULT NULL,
    start_date DATE DEFAULT NULL,
    end_date DATE DEFAULT NULL,
    description VARCHAR(200) DEFAULT NULL,
    PRIMARY KEY (task_id)
  ) ENGINE=innodb,CHARSET=utf8;
  ```

  **重复性检查**：当已经存在时，进行报错：

  ```mysql
  [ERROR] db [db_name] table [table_name] already exists!
  ```

  

### Select

- **字段**查询、**where**子句**多条件**查询

  ```mysql
  select * from table where table_name='test'
  select column_name, data_type from columns where table_name='tasks' and column_name='subject'
  ```

### Insert

- Insert语句仅会影响到表的`table_rows`属性和`update_time`属性，因此不做具体解析

  ```mysql
  insert into test values(1,2,3)
  ```

### Update

- 修改字段名

  ```mysql
  UPDATE test SET name='szj' WHERE sid=20180001;
  ```

  **存在性检查**：update不存在的表或者字段时进行报错

  ```mysql
  [ERROR] db [db_name] table [table_name] column [column_name] does not exist!	
  ```

  

### Delete

- Delete语句和insert语句的影响一致

  ```mysql
  DELETE FROM tasks WHERE sid=20184376;
  ```

### Drop数据库

- drop数据库

  ```mysql
  DROP DATBASE DB_NAME;
  ```

### Alter

#### Drop

- drop一个表的字段

  ```mysql
  ALTER TABLE tasks DROP COLUMN description;
  ```

#### Add

- add一个表的字段

  ```mysql
  ALTER TABLE tasks ADD COLUMN new_column int not null;
  ```

#### Change

- 修改字段名和属性

  ```mysql
  ALTER TABLE tasks change description my_column varchar(128) not null;		
  ```

#### Modify

- 修改字段属性

  ```mysql
  ALTER TABLE tasks modify my_column int not null default -1
  ```

### 针对索引和约束的操作

- 如**添加索引**:

  唯一索引

  ```mysql
  ALTER  TABLE  table_name  ADD  UNIQUE (column_name)
  ```

  普通索引

  ```mysql
  ALTER  TABLE  test  ADD  INDEX index_name (column_name)
  ```

- 删除索引

  ```mysql
  ALTER TABLE table_name DROP INDEX index_name
  ```

  ```mysql
  ALTER TABLE table_name DROP PRIMARY KEY
  ```

### 其他

- 如优化select，使用如

  ```mysql
  show index;
  show tables;
  show columns;
  ```

- 支持多行输入，以;作为结束符

- 存在性检查，错误提示

- 同步删除

- 通过`use database`切换当前数据库

