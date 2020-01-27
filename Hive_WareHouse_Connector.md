Hive Warehouse Connector : 
====

Let's look at the how data can be integrated with Spark-Hive in HDP 2.6.X cluster(Apache Hive 2.1.0). The tables cretaed in hive 
will be accessable with the spark.


## Creating the DB-tables in beeline :

```root@c4199-node3 ~]# beeline 

beeline> !connect jdbc:hive2://c4199-node2.squadron.support.hortonworks.com:2181,c4199-node3.squadron.support.hortonworks.com:2181,c4199-node4.squadron.support.hortonworks.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2

0: jdbc:hive2://c4199-node2.squadron.support.> show databases;
+----------------+--+
| database_name  |
+----------------+--+
| akshay         |
| default        |
+----------------+--+
2 rows selected (2.235 seconds)

0: jdbc:hive2://c4199-node2.squadron.support.> create database test_hs2_db1;

0: jdbc:hive2://c4199-node2.squadron.support.> create table test_hs2(id int, name String);

0: jdbc:hive2://c4199-node2.squadron.support.> show tables;
+-----------+--+
| tab_name  |
+-----------+--+
| test_hs2  |
+-----------+--+

0: jdbc:hive2://c4199-node2.squadron.support.> insert into test_hs2(id, name) values(2, "Akash");
0: jdbc:hive2://c4199-node2.squadron.support.> insert into test_hs2(id, name) values(3, "Nithin");
0: jdbc:hive2://c4199-node2.squadron.support.> insert into test_hs2(id, name) values(1, "Akshay");

0: jdbc:hive2://c4199-node2.squadron.support.> select * from test_hs2;
+--------------+----------------+--+
| test_hs2.id  | test_hs2.name  |
+--------------+----------------+--+
| 1            | Akshay         |
| 2            | Akash          |
| 3            | Nithin         |
+--------------+----------------+--+
```

## Accessing the same tables in Sparl-sql shell.

```[root@c4199-node3 ~]# spark-sql 
akshay
default
test_hs2_db1


spark-sql> select * from test_hs2;
1	Akshay
2	Akash
3	Nithin
Time taken: 0.554 seconds, Fetched 3 row(s)
````

++++++++++

From HDP 3.0, catalogs for Apache Hive and Apache Spark are separated, and they use their own catalog; namely, they are mutually exclusive - Apache Hive catalog can only be accessed by Apache Hive or this library, and Apache Spark catalog can 
only be accessed by existing APIs in Apache Spark . In other words, some features such as ACID tables or Apache Ranger with 
Apache Hive table are only available via this library in Apache Spark. Those tables in Hive should not directly be accessible 
within Apache Spark APIs themselves.

1. A table created by spark resides in the spark catalog
2. A table created by hive resides in the hive catalog
3. HWC API can be used to access any hive catalog table from Spark
4. Spark API can be used to access any spark catalog table
5. Must use LLAP to access read ACID tables
6. To write to an ACID table we do not need LLAP

let's try to access the tables in spark with the help of HWC:

## Creating the databsases in beeline :

Table orc_tbl is the ORC table and parquet_table is the parquet table here.

```
[root@c493-node2 hive]# beeline
1: jdbc:hive2://c493-node2.squadron.support.h> create database hive_llap;
1: jdbc:hive2://c493-node2.squadron.support.h> use hive_llap;

1: jdbc:hive2://c493-node2.squadron.support.h> create table hive_llap_tbl(id int, name String);
1: jdbc:hive2://c493-node2.squadron.support.h> insert into test_hs2(id, name) values(2, "Akash");
1: jdbc:hive2://c493-node2.squadron.support.h> insert into test_hs2(id, name) values(1, "Akshay");

1: jdbc:hive2://c493-node2.squadron.support.h> select * from hive_llap_tbl;

+-------------------+---------------------+
| hive_llap_tbl.id  | hive_llap_tbl.name  |
+-------------------+---------------------+
| 2                 | Akash               |
| 1                 | Akshay              |
+-------------------+---------------------+

1: jdbc:hive2://c493-node2.squadron.support.h> CREATE TABLE orc_tbl(name STRING,color STRING) STORED AS ORC;
1: jdbc:hive2://c493-node2.squadron.support.h> CREATE TABLE parquet_table(name STRING,color STRING) STORED AS parquet;

1: jdbc:hive2://c493-node2.squadron.support.h> show tables;

+----------------+
|    tab_name    |
+----------------+
| hive_llap_tbl  |
| orc_tbl        |
| parquet_table  |
+----------------+


1: jdbc:hive2://c493-node2.squadron.support.h> show create table hive_llap;

+----------------------------------------------------+
|                   createtab_stmt                   |
+----------------------------------------------------+
| CREATE TABLE `hive_llap_tbl`(                      |
|   `id` int,                                        |
|   `name` string)                                   |
| ROW FORMAT SERDE                                   |
|   'org.apache.hadoop.hive.ql.io.orc.OrcSerde'      |
| STORED AS INPUTFORMAT                              |
|   'org.apache.hadoop.hive.ql.io.orc.OrcInputFormat'  |
| OUTPUTFORMAT                                       |
|   'org.apache.hadoop.hive.ql.io.orc.OrcOutputFormat' |
| LOCATION                                           |
|   'hdfs://c493-node2.squadron.support.hortonworks.com:8020/warehouse/tablespace/managed/hive/hive_llap.db/hive_llap_tbl' |
| TBLPROPERTIES (                                    |
|   'bucketing_version'='2',                         |
|   'transactional'='true',                          |
|   'transactional_properties'='default',            |
|   'transient_lastDdlTime'='1579685308')            |
+----------------------------------------------------+


1: jdbc:hive2://c493-node2.squadron.support.h> show create table parquet_table;

+----------------------------------------------------+
|                   createtab_stmt                   |
+----------------------------------------------------+
| CREATE TABLE `parquet_table`(                      |
|   `name` string,                                   |
|   `color` string)                                  |
| ROW FORMAT SERDE                                   |
|   'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'  |
| STORED AS INPUTFORMAT                              |
|   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat'  |
| OUTPUTFORMAT                                       |
|   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat' |
| LOCATION                                           |
|   'hdfs://c493-node2.squadron.support.hortonworks.com:8020/warehouse/tablespace/managed/hive/hive_llap.db/parquet_table' |
| TBLPROPERTIES (                                    |
|   'bucketing_version'='2',                         |
|   'transactional'='true',                          |
|   'transactional_properties'='insert_only',        |
|   'transient_lastDdlTime'='1579685700')            |
+----------------------------------------------------+

```

## Accessing the recently created tables in Spark-Shell

```
[root@c493-node3 ~]#spark-shell --master yarn --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-1.0.0.3.1.4.0-315.jar --conf spark.security.credentials.hiveserver2.enabled=false

scala> import com.hortonworks.hwc.HiveWarehouseSession
import com.hortonworks.hwc.HiveWarehouseSession

scala> val hive = HiveWarehouseSession.session(spark).build()
hive: com.hortonworks.spark.sql.hive.llap.HiveWarehouseSessionImpl = com.hortonworks.spark.sql.hive.llap.HiveWarehouseSessionImpl@4377b35b


scala> hive.executeQuery("select * from hive_llap.hive_llap_tbl").show()
20/01/22 06:43:30 WARN TaskSetManager: Stage 0 contains a task of very large size (446 KB). The maximum recommended task size is 100 KB.
+---+------+                                                                    
| id|  name|
+---+------+
|  2| Akash|
|  1|Akshay|
+---+------+


scala> hive.executeQuery("select * from hive_llap.parquet_table").show()
20/01/22 06:44:01 WARN TaskSetManager: Stage 1 contains a task of very large size (445 KB). The maximum recommended task size is 100 KB.
+------+-----+                                                                  
|  name|color|
+------+-----+
|Nidhin| Pink|
+------+-----+

```



### Referance :

1. http://docs.hortonworks.com.s3.amazonaws.com/HDPDocuments/HDP3/HDP-3.0.0/integrating-hive/hive_integrating_hive_and_bi.pdf
2. https://community.cloudera.com/t5/Community-Articles/Integrating-Apache-Hive-with-Apache-Spark-Hive-Warehouse/ta-p/249035
