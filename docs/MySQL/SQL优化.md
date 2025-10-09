
### insert优化

- 插入多条数据时，使用批量插入

```sql
 Insert  into  tb_test  values(1,'Tom'),(2,'Cat'),(3,'Jerry');
```

- 手动提交事务

```sql
begin[start transaction];
...
commit;
```

- 主键顺序插入性能高于乱序插入

```sql
主键乱序插入 : 8  1  9  21  88  2  4  15  89  5  7  3  
主键顺序插入 : 1  2  3  4  5  7  8  9  15  21  88  89
```



- 大批量数据插入

如果一次性需要插入大批量数据(比如: 几百万的记录)，使用insert语句插入性能较低，此时可以使用MySQL数据库提供的load指令进行插入。

```sql
-- 客户端连接服务端时，加上参数  -–local-infile
mysql –-local-infile  -u  root  -p

-- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set  global  local_infile = 1;

-- 执行load指令将准备好的数据，加载到表结构中
load  data  local  infile  '/root/sql1.log'  into  table  tb_user  fields terminated  by  ','  lines  terminated  by  '\n' ; 
```

!!!info
    文件的内容并不是SQL语句，而是符合表结构的数据，类似CSV文件，具体可使用搜索引擎查询

<br>

### 主键优化

- 满足业务需求的情况下，尽量降低主键的长度。 
- 插入数据时，尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键。 
- 尽量不要使用UUID做主键或者是其他自然主键，如身份证号。 
- 业务操作时，避免对主键的修改。

<br>

### order by优化

MySQL的排序有两种方式： 

- Using filesort : 通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。 
- Using index : 通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。



**优化原则**

1. 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。 
2. 尽量使用覆盖索引。 
3. 多字段排序, 一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）。 
4. 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小  sort_buffer_size(默认256k)。

<br>

### group by优化

在分组操作中，我们需要通过以下两点进行优化，以提升性能： 

1. 分组操作时，可以通过索引来提高效率。 
2. 分组操作时，索引的使用也满足最左前缀法则。

<br>

### limit优化

在数据量比较大时，如果进行limit分页查询，在查询时，页数越往后，分页查询效率越低。

!!!example
    当在进行分页查询时，如果执行 limit 2000000,10 ，此时需要MySQL排序前2000009 记 录，仅仅返回 2000000 - 2000009 的记录，其他记录丢弃，查询排序的代价非常大 。



优化思路: 一般分页查询时，通过创建 覆盖索引 能够比较好地提高性能，可以通过**覆盖索引加子查询**形式进行优化。

```sql
explain   select  t.*  from  tb_sku  t  ,  (select  id  from  tb_sku  order  by  id 
limit  2000000,10)  a  where t.id  =  a.id;
```

<br>

### count优化

- *MyISAM* 引擎把一个表的总行数存在了磁盘上，因此执行 count(*) 的时候会直接返回这个数，效率很高； 但是如果是带条件的count，MyISAM也慢。
- *InnoDB* 引擎就麻烦了，它执行 count(*) 的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数。

如果说要大幅度提升InnoDB表的count效率，主要的优化思路：自己计数(可以**借助于redis**这样的数据库进行,但是如果是带条件的count又比较麻烦了)。



**count的几种用法**

|      用法       |            语法示例             |          执行机制          |    效率    |       适用场景       |                  注意事项                  |
| :-------------: | :-----------------------------: | :------------------------: | :--------: | :------------------: | :----------------------------------------: |
|  **COUNT(\*)**  |  `SELECT COUNT(*) FROM table`   | 直接统计行数，不解析具体列 | ★★★★★ 最快 |     统计表总行数     |        优先使用，InnoDB做了特殊优化        |
| **COUNT(主键)** |  `SELECT COUNT(id) FROM table`  |  遍历主键索引统计非NULL值  | ★★★★ 较快  | 需要显式指定主键统计 |        比COUNT(*)稍慢，因需读取索引        |
| **COUNT(字段)** | `SELECT COUNT(name) FROM users` |  遍历指定字段统计非NULL值  |  ★★ 较慢   | 统计某列非NULL值数量 | 避免在可为NULL的列使用，不走索引时效率最低 |
| **COUNT(数字)** |  `SELECT COUNT(1) FROM table`   |   生成数字常量列统计行数   |  ★★★★ 快   |  与COUNT(*)功能相同  |     MySQL 8.0+优化后与COUNT(*)性能相当     |

<br>

### update优化

更新条件尽量使用索引字段且该索引不能失效，否则MySQL会对整个表加锁，影响性能


