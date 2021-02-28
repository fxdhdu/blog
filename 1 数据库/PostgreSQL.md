## PostgreSQL支持的分区类型	

- Range Partitioning
  The table is partitioned into “ranges” defined by a key column or set of columns, with no overlap between the ranges of values assigned to different partitions. For example, one might partition by date ranges, or by ranges of identifiers for particular business objects.

范围分区表：在创建时，就要指定各个分区的范围和分区名称。或者在创建完主分区表后，手动创建分表。当插入的数据不在任何分区时会报错。也可以新增日期字段，在存入数据时，通过时间戳计算出当前日期，从而改用分区表。

```
CREATE TABLE measurement (
    city_id         int not null,
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY RANGE (logdate);
```

```
CREATE TABLE measurement_y2006m02 PARTITION OF measurement
    FOR VALUES FROM ('2006-02-01') TO ('2006-03-01');
```

- List Partitioning
  The table is partitioned by explicitly listing which key values appear in each partition.

- Hash Partitioning
  The table is partitioned by specifying a modulus and a remainder for each partition. Each partition will hold the rows for which the hash value of the partition key divided by the specified modulus will produce the specified remainder.

## Postgresql的索引类型

- BTREE

  - 索引字段长度超过1/3索引页的大小，创建索引时会报错。
  - 技术上是B+树结构
  - High Key：B+树的每一个节点都有一个High-Key值，此值表示此节点或者此节点的子节点的最大值。

- 哈希索引（HASH）

Postgresql添加索引

  - sql语法参考：https://www.postgresql.org/docs/current/sql-createindex.html
  - 当不指定索引方法时，默认为BTREE。其他包括哈希索引、全文索引等。