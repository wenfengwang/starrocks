---
displayed_sidebar: "Chinese"
---

# 本地连接

对于洗牌连接和广播连接，如果满足连接条件，则两个连接表的数据行将合并为单个节点，完成连接。这两种连接方法都无法避免由节点间数据网络传输引起的延迟或开销。

核心思想是保持同一共位组中表的分桶键、副本数量和副本位置一致。如果连接列是一个分桶键，计算节点只需执行本地连接，而无需从其他节点获取数据。本地连接支持等值连接。

本文介绍了本地连接的原则、实现、用法和注意事项。

## 术语

* **Co-location Group (CG)**: 一个 CG 中将包含一个或多个 Tables。CG 中的 Tables 具有相同的分桶和复制位置，并且使用 Co-location Group Schema 进行描述。
* **Co-location Group Schema (CGS)**: CGS 包含 CG 的分桶键、桶数和副本数。

## 原则

本地连接是为了形成一个具有一组具有相同 CGS 的 Tables 的 CG，并确保这些 Tables 的对应桶副本会落在相同的 BE 节点集上。当 CG 中的 Tables 在分桶列上执行 Join 操作时，本地数据可以直接进行连接，节省了节点间数据传输的时间。

Bucket Seq 由 `hash(key) mod buckets` 获得。假设一个表有 8 个 buckets，那么就有 \[0, 1, 2, 3, 4, 5, 6, 7\] 这 8 个 buckets，每个 Bucket 都有一个或多个子表，子表的数量取决于分区的数量。如果是多分区表，就会有多个 tablets。

为了具有相同的数据分布，同一个 CG 中的表必须遵守以下规则。

1. 同一个 CG 中的表必须具有相同的分桶键（类型、数量、顺序）和相同的桶数，以便多个表的数据切片可以被逐个分配和控制。分桶键是在表创建语句 `DISTRIBUTED BY HASH(col1, col2, ...)` 中指定的列。分桶键决定了哪些数据列被散列到不同的 Bucket Seqs 中。分桶键的名称对于同一 CG 中的表可能是不同的。分桶列在创建语句中可以不同，但`DISTRIBUTED BY HASH(col1, col2, ...)` 中相应数据类型的顺序必须完全相同 。
2. 同一个 CG 中的表必须具有相同数量的分区副本。如果不是，可能会出现一个 tablet 副本在同一个 BE 的分区中没有相应的副本。
3. 同一个 CG 中的表可能具有不同数量的分区和不同的分区键。

在创建表时，CG 通过表属性中的 `"colocate_with" = "group_name"` 进行指定。如果 CG 不存在，则意味着该表是 CG 的第一张表，并称为父表。父表的数据分布（拆分分桶键的类型、数量和顺序、拆分桶的数量以及副本数量）决定了 CGS。如果 CG 存在，则检查表的数据分布是否与 CGS 一致。

同一个 CG 中的表的副本放置满足以下条件：

1. 所有 Tables 的 Bucket Seq 与 BE 节点的映射方式与 Parent Table 的相同。
2. 所有 Parent Table 中的所有 Partitions 的 Bucket Seq 与 BE 节点的映射方式与第一 Partition 的相同。
3. Parent Table 的第一个 Partition 的 Bucket Seq 与 BE 节点的映射方式根据原生的 Round Robin 算法确定。

一致的数据分布和映射保证了相同分桶键的数据行落在同一个 BE 上。因此，当使用分桶键进行连接列时，只需要本地连接。

## 用法

### 创建表

创建表时，可以在 PROPERTIES 中指定属性 `"colocate_with" = "group_name"`，以表示该表是本地连接表，属于指定的 Co-location Group。
> **注意**
>
> 从版本 2.5.4 开始，可以在来自不同数据库的表上执行本地连接。只需在创建表时指定相同的 `colocate_with` 属性即可。

例如：

~~~SQL
CREATE TABLE tbl (k1 int, v1 int sum)
DISTRIBUTED BY HASH(k1)
BUCKETS 8
PROPERTIES(
    "colocate_with" = "group1"
);
~~~

如果指定的 Group 不存在，StarRocks 会自动创建一个只包含当前表的 Group。如果 Group 存在，StarRocks 会检查当前表是否符合 Co-location Group Schema。如果符合，则创建该表并将其添加到 Group。同时，根据现有 Group 的数据分布规则，为该表创建一个分区和复制。

Co-location Group 属于数据库。Co-location Group 的名称在数据库中是唯一的。在内部存储中，Co-location Group 的全名是 `dbId_groupName`，但您只需感知到 `groupName`。
> **注意**
>
> 如果要将来自不同数据库的表关联进行本地连接，同一个 Co-location Group 必须存在于这些数据库中。您可以运行 `show proc "/colocation_group"` 来检查不同数据库中的 Co-location Group。

### 删除

完全删除是指在回收站中删除。通常，在使用 `DROP TABLE` 命令删除表后，默认情况下，该表将在被删除前在回收站中保存一天。当 Group 中的最后一个表被完全删除时，Group 也将被自动删除。

### 查看 Group 信息

以下命令可用于查看集群中已经存在的 Group 信息。

~~~Plain Text
SHOW PROC '/colocation_group';

+-------------+--------------+--------------+------------+----------------+----------+----------+
| GroupId     | GroupName    | TableIds     | BucketsNum | ReplicationNum | DistCols | IsStable |
+-------------+--------------+--------------+------------+----------------+----------+----------+
| 10005.10008 | 10005_group1 | 10007, 10040 | 10         | 3              | int(11)  | true     |
+-------------+--------------+--------------+------------+----------------+----------+----------+
~~~

* **GroupId**: Group 的集群唯一标识符，前半部分是数据库 id，后半部分是 Group id。
* **GroupName**: Group 的全名。
* **TabletIds**: Group 中表的 id 列表。
* **BucketsNum**: 桶数。
* **ReplicationNum**: 复制数。
* **DistCols**: 分发列，即分桶列类型。
* **IsStable**: Group 是否稳定（稳定的定义，请参见 Co-location Replica Balancing and Repair 部分）。

您还可以使用以下命令进一步查看 Group 的数据分布。

~~~Plain Text
SHOW PROC '/colocation_group/10005.10008';

+-------------+---------------------+
| BucketIndex | BackendIds          |
+-------------+---------------------+
| 0           | 10004, 10002, 10001 |
| 1           | 10003, 10002, 10004 |
| 2           | 10002, 10004, 10001 |
| 3           | 10003, 10002, 10004 |
| 4           | 10002, 10004, 10003 |
| 5           | 10003, 10002, 10001 |
| 6           | 10003, 10004, 10001 |
| 7           | 10003, 10004, 10002 |
+-------------+---------------------+
~~~

* **BucketIndex**: 桶序列的下标。
* **BackendIds**: 桶数据切片所在的 BE 节点的 id。

> 注意：上述命令需要具有 AMDIN 权限。常规用户无法访问它。

### 修改表组属性

可以修改表的 Co-location 组属性。例如：

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "group2");
~~~

如果表之前尚未分配到 Group，该命令将检查 Schema 并将表添加到 Group（如果 Group 不存在，则首先创建 Group）。如果表之前已分配到其他 Group，该命令将从原始 Group 中移除表并将其添加到新的 Group（如果 Group 不存在，则首先创建 Group）。

您还可以使用以下命令去除表的 Co-location 属性。

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "");
~~~

### 其他相关操作

在使用 `ADD PARTITION` 添加分区或修改具有 Co-location 属性的表的副本数量时，StarRocks 会检查该操作是否会违反 Co-location Group Schema，并在违反时拒绝该操作。

## Co-location 副本平衡和修复

Colocation表的副本分布需要遵循Group模式中指定的分布规则，因此在副本修复和平衡方面与普通分片有所不同。

组本身有一个`stable`属性。当`stable`为`true`时，意味着在Group中未对表分片进行任何更改，并且Colocation功能正常工作。当`stable`为`false`时，意味着当前Group中的一些表分片正在进行修复或迁移，并且受影响的表的Colocate Join将降级为普通Join。

### 副本修复

副本只能存储在指定的BE节点上。StarRocks将寻找负载最轻的BE节点来替换不可用的BE（例如，下线、停用）。替换后，旧BE上的所有桶数据片将被修复。在迁移期间，该Group被标记为**不稳定**。

### 副本平衡

StarRocks尝试将Colocation表片块均匀分布在所有BE节点上。普通表的平衡是在副本级别进行的，也就是说，每个副本单独找到一个负载较低的BE节点。Colocation表的平衡是在Bucket级别进行的，也就是说，同一个Bucket中的所有副本一起迁移。我们使用一个简单的平衡算法，它会在所有BE节点上均匀分布`BucketsSequnce`，而不考虑副本的实际大小，只考虑副本的数量。详细算法可以在`ColocateTableBalancer.java`的代码注释中找到。

> 注意1：当前的Colocation副本平衡和修复算法可能不适用于部署异构的StarRocks集群。所谓的异构部署是指BE节点的磁盘容量、磁盘数量和磁盘类型（SSD和HDD）不一致。在异构部署的情况下，可能发生小容量BE节点存储与大容量BE节点相同数量的副本的情况。

> 注意2：当一个Group处于不稳定状态时，其表的Join将降级为普通Join，这可能会显著降低集群的查询性能。如果不希望系统自动平衡，请在适当的时候设置FE配置`disable_colocate_balance`以禁用自动平衡，并在适当的时候启用它。 (详细信息请参见高级操作（#高级操作）部分)

## 查询

Colocation表的查询与普通表的查询方式相同。如果Colocation表所在的Group处于不稳定状态，则它将自动降级为普通Join，如下例所示。

表1:

~~~SQL
CREATE TABLE `tbl1` (
    `k1` date NOT NULL COMMENT "",
    `k2` int(11) NOT NULL COMMENT "",
    `v1` int(11) SUM NOT NULL COMMENT ""
) ENGINE=OLAP
AGGREGATE KEY(`k1`, `k2`)
PARTITION BY RANGE(`k1`)
(
    PARTITION p1 VALUES LESS THAN ('2019-05-31'),
    PARTITION p2 VALUES LESS THAN ('2019-06-30')
)
DISTRIBUTED BY HASH(`k2`)
PROPERTIES (
    "colocate_with" = "group1"
);
~~~

表2:

~~~SQL
CREATE TABLE `tbl2` (
    `k1` datetime NOT NULL COMMENT "",
    `k2` int(11) NOT NULL COMMENT "",
    `v1` double SUM NOT NULL COMMENT ""
) ENGINE=OLAP
AGGREGATE KEY(`k1`, `k2`)
DISTRIBUTED BY HASH(`k2`)
PROPERTIES (
    "colocate_with" = "group1"
);
~~~

查询计划:

~~~Plain Text
DESC SELECT * FROM tbl1 INNER JOIN tbl2 ON (tbl1.k2 = tbl2.k2);

+----------------------------------------------------+
| Explain String                                     |
+----------------------------------------------------+
| PLAN FRAGMENT 0                                    |
|  OUTPUT EXPRS:`tbl1`.`k1` |                        |
|   PARTITION: RANDOM                                |
|                                                    |
|   RESULT SINK                                      |
|                                                    |
|   2:HASH JOIN                                      |
|   |  join op: INNER JOIN                           |
|   |  hash predicates:                              |
|   |  colocate: true                                |
|   |    `tbl1`.`k2` = `tbl2`.`k2`                   |
|   |  tuple ids: 0 1                                |
|   |                                                |
|   |----1:OlapScanNode                              |
|   |       TABLE: tbl2                              |
|   |       PREAGGREGATION: OFF. Reason: null        |
|   |       partitions=0/1                           |
|   |       rollup: null                             |
|   |       buckets=0/0                              |
|   |       cardinality=-1                           |
|   |       avgRowSize=0.0                           |
|   |       numNodes=0                               |
|   |       tuple ids: 1                             |
|   |                                                |
|   0:OlapScanNode                                   |
|      TABLE: tbl1                                   |
|      PREAGGREGATION: OFF. Reason: No AggregateInfo |
|      partitions=0/2                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 0                                  |
+----------------------------------------------------+
~~~

如果Colocate Join生效，则Hash Join节点将显示`colocate: true`。

如果Colocate Join未生效，则查询计划如下:

~~~Plain Text
+----------------------------------------------------+
| Explain String                                     |
+----------------------------------------------------+
| PLAN FRAGMENT 0                                    |
|  OUTPUT EXPRS:`tbl1`.`k1` |                        |
|   PARTITION: RANDOM                                |
|                                                    |
|   RESULT SINK                                      |
|                                                    |
|   2:HASH JOIN                                      |
|   |  join op: INNER JOIN (BROADCAST)               |
|   |  hash predicates:                              |
|   |  colocate: false, reason: group is not stable  |
|   |    `tbl1`.`k2` = `tbl2`.`k2`                   |
|   |  tuple ids: 0 1                                |
|   |                                                |
|   |----3:EXCHANGE                                  |
|   |       tuple ids: 1                             |
|   |                                                |
|   0:OlapScanNode                                   |
|      TABLE: tbl1                                   |
|      PREAGGREGATION: OFF. Reason: No AggregateInfo |
|      partitions=0/2                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 0                                  |
|                                                    |
| PLAN FRAGMENT 1                                    |
|  OUTPUT EXPRS:                                     |
|   PARTITION: RANDOM                                |
|                                                    |
|   STREAM DATA SINK                                 |
|     EXCHANGE ID: 03                                |
|     UNPARTITIONED                                  |
|                                                    |
|   1:OlapScanNode                                   |
|      TABLE: tbl2                                   |
|      PREAGGREGATION: OFF. Reason: null             |
|      partitions=0/1                                |
|      rollup: null                                  |
|      buckets=0/0                                   |
|      cardinality=-1                                |
|      avgRowSize=0.0                                |
|      numNodes=0                                    |
|      tuple ids: 1                                  |
+----------------------------------------------------+
~~~

HASH JOIN节点将显示相应的原因: `colocate: false, reason: group is not stable`。与此同时还会生成一个EXCHANGE节点。

## 高级操作

### FE配置项

* **disable_colocate_relocate**

    是否禁用StarRocks的Colocation副本修复自动功能。默认值为false，表示已打开。此参数仅影响Colocation表的副本修复，不影响普通表的副本修复。

* **disable_colocate_balance**

    是否禁用StarRocks的Colocation副本平衡自动功能。默认值为false，表示已打开。此参数仅影响Colocation表的副本平衡，不影响普通表的副本平衡。

* **disable_colocate_join**

    通过更改此变量，可以在会话级别禁用Colocate join。

* **disable_colocate_join**

    通过更改此变量，可以禁用Colocate join功能。

### HTTP Restful API

StarRocks提供了几个与Colocate Join相关的HTTP Restful API，用于查看和修改Colocation组。

此API在FE上实现，并且可以使用具有ADMIN权限的`fe_host:fe_http_port`进行访问。

1. 查看集群的所有Colocation信息

    ~~~bash
    curl --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate'  
    ~~~

    ~~~JSON
    //以Json格式返回内部Colocation信息。
    {
        "colocate_meta": {
            "groupName2Id": {
                "g1": {
                    "dbId": 10005,
                    "grpId": 10008
                }
            },
            "group2Tables": {},
            "table2Group": {
                "10007": {
                    "dbId": 10005,
    "grpId": 10008
                },
                "10040": {
                    "dbId": 10005,
                    "grpId": 10008
                }
            },
            "group2Schema": {
                "10005.10008": {
                    "groupId": {
                        "dbId": 10005,
                        "grpId": 10008
                    },
                    "distributionColTypes": [{
                        "type": "INT",
                        "len": -1,
                        "isAssignedStrLenInColDefinition": false,
                        "precision": 0,
                        "scale": 0
                    }],
                    "bucketsNum": 10,
                    "replicationNum": 2
                }
            },
            "group2BackendsPerBucketSeq": {
                "10005.10008": [
                    [10004, 10002],
                    [10003, 10002],
                    [10002, 10004],
                    [10003, 10002],
                    [10002, 10004],
                    [10003, 10002],
                    [10003, 10004],
                    [10003, 10004],
                    [10003, 10004],
                    [10002, 10004]
                ]
            },
            "unstableGroups": []
        },
        "status": "OK"
    }
    ~~~

2. 将组标记为稳定或不稳定

    ~~~bash
    # 标记为稳定
    curl -XPOST --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_stable?db_id=<dbId>&group_id=<grpId>'
    # 标记为不稳定
    curl -XPOST --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_unstable?db_id=<dbId>&group_id=<grpId>'
    ~~~

    如果返回结果为`200`，则成功将组标记为稳定或不稳定。

3. 设置组的数据分布

    此接口允许您强制设置组的数字分布。

    `POST /api/colocate/bucketseq?db_id=10005&group_id= 10008`

    `Body:`

    `[[10004,10002],[10003,10002],[10002,10004],[10003,10002],[10002,10004],[10003,10002],[10003,10004],[10003,10004],[10003,10004],[10002,10004]]`

    `返回: 200`

    其中`Body`是以嵌套数组表示的`BucketsSequence`，以及存放分桶切片的BE的id。

    > 请注意，要使用此命令，您可能需要将FE配置`disable_colocate_relocate`和`disable_colocate_balance`设置为true，即禁用系统执行自动Colocation副本修复和平衡。否则，在修改后系统可能会自动重置。