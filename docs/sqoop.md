## Sqoop简单使用demo

##### 导入数据

​	导入是指，从关系型数据库(RDBMS)向非关系型向大数据集群(HDFS, HIVE, HBASE) 中传输数据，叫做导入，  使用关键字**import**

##### RDBMS到HDFS

###### 全部导入

```
bin/sqoop import --connect jdbc:mysql://hadoop102:3306/company --username root --password 1 --table staff --target-dir /user/company --delete-target-dir --num-mappers 1 --fields-terminated-by "\t"
```

###### 查询导入

```
#查询导入
bin/sqoop import --connect jdbc:mysql://hadoop102:3306/company --username root --password 1 --target-dir /user/company --delete-target-dir --num-mappers 1 --fields-terminated-by "\t" --query 'select name,sex from staff where id>=1 and $CONDITIONS;'
```

注意：$CONDITIONS必须加上 ,还要使用 **''** 括起来

###### 导入指定列

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 1 \
--table staff \
--target-dir /user/company \
--num-mappers 1 \
--columns id,name --delete-target-dir

```

###### 使用sqoop关键字筛选查询导入数据

```
bin/sqoop import --connect jdbc:mysql://hadoop102:3306/company --username root --password 1 --target-dir /user/company --delete-target-dir --num-mappers 1 --fields-terminated-by "\t" --table staff --where "id=5"
```

###### 将数据导入到Hive中

```
bin/sqoop import --connect jdbc:mysql://hadoop102:3306/company --username root --password 1 --table staff --num-mappers 1 --hive-import --fields-terminated-by "\t" --hive-overwrite --hive-table staff_hive
```

###### 将数据导入到Hbase

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 1 \
--table staff \
--columns "id,name,sex" \
--column-family "info" \
--hbase-create-table \
--hbase-row-key "id" \
--hbase-table "hbase_company" \
--num-mappers 1 \
--split-by id
```



#### 导出数据

###### Hive/HDFS 到RDBMS

```

```

