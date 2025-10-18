### 系统数据库

MySQL自带数据库说明：

| 数据库             | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| mysql              | 存储MySQL服务器正常运行所需要的各种信息 （时区、主从、用 户、权限等） |
| information_schema | 提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类 型及访问权限等 |
| performance_schema | 为MySQL服务器运行时状态提供了一个底层监控功能，主要用于收集 数据库服务器性能参数 |
| sys                | 包含了一系列方便 DBA 和开发人员利用 performance_schema 性能数据库进行性能调优和诊断的视图 |



### MySQL命令行客户端

- mysql

```bash
语法 ：    
    mysql   [options]   [database]
选项 ： 
    -u, --user=name         #指定用户名
    -p, --password[=name]           #指定密码
    -h, --host=name         #指定服务器IP或域名
    -P, --port=port             #指定连接端口
    -e, --execute=name          #执行SQL语句并退出
```

<br>

- mysqladmin，用于执行各种管理操作

```bash
mysqladmin -u root -p status	# 查看服务器状态
mysqladmin -u root -p variables	# 查看服务器变量
mysqladmin -u root -p shutdown	# 关闭服务器
mysqladmin -u root -p flush-logs      # 刷新日志
mysqladmin -u root -p flush-privileges # 刷新权限
mysqladmin -u root -p flush-status    # 重置状态计数器
```

<br>

- mysqlbinlog，由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog 日志管理工具

```bash
语法 ：    
	mysqlbinlog [options]  log-files1 log-files2 ...

选项 ： 
	-d, --database=name         指定数据库名称，只列出指定的数据库相关操作。
	-o, --offset=#              忽略掉日志中的前n行命令。
	-r,--result-file=name       将输出的文本格式日志输出到指定文件。
	-s, --short-form            显示简单格式， 省略掉一些信息。
	--start-datatime=date1  --stop-datetime=date2       指定日期间隔内的所有日志。
	--start-position=pos1 --stop-position=pos2          指定位置间隔内的所有日志。
```

<br>

- mysqlshow，对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索 引。

```bash
语法 ：    
    mysqlshow [options] [db_name [table_name [col_name]]]
选项 ： 
    --count     显示数据库及表的统计信息（数据库，表 均可以不指定）
    -i      显示指定数据库或者指定表的状态信息
示例：
 	#查询test库中每个表中的字段书，及行数
    mysqlshow -uroot -p2143 test --count
    
    #查询test库中book表的详细情况
    mysqlshow -uroot -p2143 test book --count
```

<br>

- mysqldump，用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及 插入表的SQL语句。

```bash
语法 ：    
    mysqldump [options] db_name [tables]
    mysqldump [options] --database/-B db1 [db2 db3...]
    mysqldump [options] --all-databases/-A
连接选项 ：  
    -u, --user=name                 指定用户名
    -p, --password[=name]           指定密码
    -h, --host=name                 指定服务器ip或域名
    -P, --port=#                    指定连接端口
输出选项：
    --add-drop-database         在每个数据库创建语句前加上 drop database 语句
    --add-drop-table            在每个表创建语句前加上 drop table 语句 , 默认开启 ; 不开启 (--skip-add-drop-table)
    -n, --no-create-db          不包含数据库的创建语句
    -t, --no-create-info        不包含数据表的创建语句
    -d --no-data                不包含数据
    -T, --tab=name             	自动生成两个文件：一个.sql文件，创建表结构的语句；一个.txt文件，数据文件
  
-- 备份数据库
mysqldump -uroot -p1234 db01 > db01.sql
```

<br>

-  mysqlimport ，是客户端数据导入工具，用来导入mysqldump 加 -T 参数后导出的文本文件。

```sql
语法 ：    
	mysqlimport [options]  db_name  textfile1  [textfile2...]
示例 ： 
	mysqlimport -uroot -p2143 test /tmp/city.txt
```

<br>

-  source ，如果需要导入sql文件,可以使用**mysql中的source** 指令

```sql
语法 ：    
source /root/xxxxx.sql
```

---
