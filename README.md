# demo-trino

Based on https://github.com/bitsondatadev/trino-getting-started

Docker compose for a federated queries Trino demo with :

- Hdfs3/Hive
- Minio
- MongoDB
- Mysql

Dataset : https://docs.snowflake.com/en/user-guide/sample-data-tpch.html#database-and-schemas

Connect to Trino

```bash
docker exec -it trino-hdfs3_trino-coordinator_1 trino
```

Connect to Hive

```bash
docker exec -it trino-hdfs3-mongo-mysql_hadoop-node_1 hive
```

Connect to Mongo

```bash
docker container exec -it trino-hdfs3-mongo-mysql_mongodb_1 mongo
```

Show Catalogs

```SQL
SHOW CATALOGS;
```

Show schemas from catalog

```SQL
SHOW SCHEMAS FROM tpc;
```

Show tables from schema

```SQL
SHOW TABLES FROM tpch.tiny;
```

Create Schema

```SQL
CREATE SCHEMA hdfs.demo;
CREATE SCHEMA mongo.demo;
```

Create Hive table from tpch dataset

```SQL
CREATE TABLE hdfs.demo.customer_sf10
WITH (
    format = 'ORC',
    external_location = 'hdfs://hadoop-node:9000/user/hive/warehouse/customer_sf10'
    )
    AS select * from tpch.sf10.customer;

CREATE TABLE hdfs.demo.customer_sf100
WITH (
    format = 'ORC',
    external_location = 'hdfs://hadoop-node:9000/user/hive/warehouse/customer_sf100'
    )
    AS select * from tpch.sf100.customer;
```

```SQL
CREATE TABLE mongo.demo.nation_sf100 AS SELECT * FROM tpch.sf100.nation;
```

Trino request

```SQL
select mktsegment, sum(acctbal) as sum_acctbal from hdfs.demo.customer_sf100 group by mktsegment order by mktsegment;
```

Hive request

```SQL
use demo;
select mktsegment, sum(acctbal) as sum_acctbal from customer_sf100 group by mktsegment order by mktsegment;
```

Federated query Hive/Mongo

```SQL
select * from hdfs.demo.customer_sf10 as cust inner join mongo.demo.nation_sf10 as nation on cust.nationkey = nation.nationkey;
```
