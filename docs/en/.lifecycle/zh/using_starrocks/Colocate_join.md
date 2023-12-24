---
displayed_sidebar: English
---

# 共置连接

对于洗牌连接和广播连接，如果满足连接条件，则将两个连接表的数据行合并为单个节点，完成连接。这两种连接方式都无法避免节点间数据网络传输带来的延迟或开销。

其核心思想是保持同一主机托管组中表的存储桶键、副本数和副本位置一致。如果 join 列是 bucketing key，则计算节点只需要进行本地 join，无需从其他节点获取数据。Colocate Join 支持 equi join。

本文档介绍了Colocate Join的原理、实现、使用方法和注意事项。

## 术语

* **主机托管组 (CG)**: 一个CG将包含一个或多个表。CG中的表具有相同的分桶和副本放置，并使用主机托管组架构进行描述。
* **主机托管组架构 (CGS)**: 一个CGS包含CG的存储桶密钥、存储桶数量和副本数量。

## 原则

Colocate Join是与具有相同CGS的一组表形成一个CG，并确保这些表对应的存储桶副本将落在同一组BE节点上。当CG中的表对Bucket列进行Join操作时，可以直接连接本地数据，从而节省了节点间传输数据的时间。

Bucket Seq通过`hash(key) mod buckets`获得。假设一个表有8个Bucket，那么有\[0, 1, 2, 3, 4, 5, 6, 7\] 8个Bucket，每个Bucket有一个或多个子表，子表的数量取决于分区的数量。如果是多分区表，则会有多个平板电脑。

为了具有相同的数据分布，同一CG中的表必须符合以下要求。

1. 同一CG中的表必须具有相同的bucketing键（类型、编号、顺序）和相同的桶数，以便可以逐个分发和控制多个表的数据切片。存储桶键是表创建语句中指定的列`DISTRIBUTED BY HASH(col1, col2, ...)`。存储桶键确定将哪些数据列哈希到不同的存储桶序列中。对于同一CG中的表，存储桶键的名称可能会有所不同。创建语句中的存储桶列可以不同，但中相应数据类型的顺序`DISTRIBUTED BY HASH(col1, col2, ...)`应完全相同。
2. 同一CG中的表必须具有相同数量的分区副本。否则，可能会发生tablet副本在同一BE的分区中没有对应副本的情况。
3. 同一CG中的表可能具有不同的分区数和不同的分区键。

创建表时，CG由`"colocate_with" = "group_name"`表PROPERTIES中的属性指定。如果CG不存在，则表示该表是CG的第一个表，称为父表。父表的数据分布（拆分存储桶键的类型、数量和顺序、副本数和拆分存储桶数量）决定了CGS。如果CG存在，请检查表的数据分布是否与CGS一致。

在同一CG中复制表的位置满足以下条件:

1. 所有Table的Bucket Seq和BE节点之间的映射与父表的映射相同。
2. 父表中所有分区的Bucket Seq和BE节点之间的映射关系与第一个分区的映射关系相同。
3. 父表的第一个分区的Bucket Seq和BE节点之间的映射是使用本机循环算法确定的。

一致的数据分布和映射保证了bucketing key获取的相同值的数据行落在同一个BE上。因此，在使用存储桶键联接列时，只需要本地联接。

## 用法

### 表创建

创建表时，可以在PROPERTIES中指定属性`"colocate_with" = "group_name"`，以指示该表是共置联接表，并且属于指定的共置组。
> **注意**
>
> 从版本2.5.4开始，可以对来自不同数据库的表执行共置联接。只需`colocate_with`在创建表时指定相同的属性。

例如:

~~~SQL
CREATE TABLE tbl (k1 int, v1 int sum)
DISTRIBUTED BY HASH(k1)
BUCKETS 8
PROPERTIES(
    "colocate_with" = "group1"
);
~~~

如果指定的Group不存在，StarRocks会自动创建一个仅包含当前表的Group。如果Group存在，StarRocks会检查当前表是否符合Colocation Group Schema。如果是这样，它将创建表并将其添加到组中。同时，该表根据现有Group的数据分布规则创建分区和Tablet。

主机托管组属于数据库。主机托管组的名称在数据库中是唯一的。在内部存储中，托管组的全称是，`dbId_groupName`但您只感知到`groupName`。
> **注意**
>
> 如果指定相同的共置组来关联来自不同数据库的表进行共置联接，则每个数据库中都存在共置组。您可以运行`show proc "/colocation_group"`以检查不同数据库中的主机托管组。

### 删除

完全删除是从回收站中删除。通常，使用命令删除表后`DROP TABLE`，默认情况下它会在回收站中保留一天，然后才会被删除）。当组中的最后一个表被完全删除时，该组也将自动删除。

### 查看组信息

以下命令允许您查看集群中已存在的组信息。

~~~Plain Text
SHOW PROC '/colocation_group';

+-------------+--------------+--------------+------------+----------------+----------+----------+
| GroupId     | GroupName    | TableIds     | BucketsNum | ReplicationNum | DistCols | IsStable |
+-------------+--------------+--------------+------------+----------------+----------+----------+
| 10005.10008 | 10005_group1 | 10007, 10040 | 10         | 3              | int(11)  | true     |
+-------------+--------------+--------------+------------+----------------+----------+----------+
~~~

* **GroupId**: Group的集群范围唯一标识符，前半部分是数据库ID，后半部分是组ID。
* **GroupName**: 组的全名。
* **TabletIds**: 组中表的ID列表。
* **BucketsNum**: 桶数。
* **ReplicationNum**: 副本数。
* **DistCols**: 分布列，即分桶列类型。
* **IsStable**: 组是否稳定（有关稳定性的定义，请参阅托管副本平衡和修复部分）。

您可以使用以下命令进一步查看Group的数据分布。

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

* **BucketIndex**: 存储桶序列的下标。
* **BackendIds**: 存储桶数据切片t所在的BE节点的ID。

> 注意：上述命令需要AMDIN权限。普通用户无法访问它。

### 修改表组属性

您可以修改表的“共置组”属性。例如:

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "group2");
~~~

如果之前没有将该表分配给某个组，该命令将检查Schema并将该表添加到该组中（如果该组不存在，则将首先创建该组）。如果该表之前已分配给另一个组，则该命令将从原始组中删除该表并将其添加到新组（如果该组不存在，则将首先创建该组）。

还可以使用以下命令删除表的Colocation属性。

~~~SQL
ALTER TABLE tbl SET ("colocate_with" = "");
~~~

### 其他相关操作

`ADD PARTITION`在使用或修改具有Colocation属性的表中添加分区或修改副本数时，StarRocks会检查该操作是否会违反Colocation Group Schema，如果违反，则拒绝。

## 主机托管副本平衡和修复

Colocation表的副本分发需要遵循Group架构中指定的分发规则，因此在副本修复和均衡方面与普通分片不同。

集团本身拥有财产`stable`。当`stable`为`true`时，表示未对组中的表切片进行任何更改，并且Colocation功能正常工作。当`stable` is `false`时，表示当前组中的某些表切片正在修复或迁移，受影响表的Colocate Join将降级为正常Join。

### 复制副本修复

副本只能存储在指定的BE节点上。StarRocks会寻找负载最少的BE来替换不可用的BE（例如，down、decommission）。更换后，旧BE上的所有桶数据切片都会被修复。在迁移过程中，组被标记为**不稳定**。

### 副本均衡

StarRocks 尝试将 Colocation 表切片均匀分布在所有 BE 节点上。普通表的平衡是在副本级别进行的，即每个副本单独查找负载较低的 BE 节点。Colocation 表的平衡是在 Bucket 级别进行的，即 Bucket 中的所有副本都一起迁移。我们使用一种简单的平衡算法，该算法会在所有 BE 节点上均匀分布 `BucketsSequnce`，而不考虑副本的实际大小，只考虑副本的数量。确切的算法可以在 `ColocateTableBalancer.java` 的代码注释中找到。

> 注 1：目前的 Colocation 副本平衡和修复算法可能不适用于具有异构部署的 StarRocks 集群。所谓的异构部署是指 BE 节点的磁盘容量、磁盘数量和磁盘类型（SSD 和 HDD）不一致。在异构部署的情况下，可能会出现小容量 BE 节点存储与大容量 BE 节点相同数量的副本。

> 注 2：当 Group 处于不稳定状态时，其表的 Join 会降级为普通的 Join，这可能会显著降低集群的查询性能。如果您不希望系统自动平衡，请将 FE 配置 `disable_colocate_balance` 设置为禁用自动平衡，并在适当的时候重新启用（有关详细信息，请参阅高级操作（#Advanced Operations）部分）。

## 查询

Colocation 表的查询方式与普通表相同。如果 Colocation 表所在的 Group 处于不稳定状态，它将自动降级为普通的 Join，如以下示例所示。

表 1：

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

表 2：

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

查看查询计划：

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

如果 Colocate Join 生效，则 Hash Join 节点将显示 `colocate: true`。

如果不生效，则查询计划如下：

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

HASH JOIN 节点将显示相应的原因： `colocate: false, reason: group is not stable`。同时将生成一个 EXCHANGE 节点。

## 高级操作

### FE 配置项

* **disable_colocate_relocate**

是否禁用 StarRocks 的 Colocation 副本自动修复。默认值为 false，表示已启用。此参数仅影响共置表的副本修复，不影响普通表的副本修复。

* **disable_colocate_balance**

是否禁用 StarRocks 的 Colocation 副本自动平衡。默认值为 false，表示已启用。此参数仅影响 Colocation 表的副本平衡，不影响普通表的副本平衡。

* **disable_colocate_join**

    您可以通过更改此变量来禁用会话级别的共置联接。

* **disable_colocate_join**

    可以通过更改此变量来禁用 Colocate 联接功能。

### HTTP Restful API

StarRocks 提供了多个与 Colocation Join 相关的 HTTP Restful API，用于查看和修改 Colocation Group。

此 API 在 FE 上实现，可以使用具有 ADMIN 权限的 `fe_host:fe_http_port` 进行访问。

1. 查看集群的所有 Colocation 信息

    ~~~bash
    curl --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate'  
    ~~~

    ~~~JSON
    // 以 JSON 格式返回内部 Colocation 信息。
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
    curl -XPOST --location-trusted -u<username>:<password> ​'http://<fe_host>:<fe_http_port>/api/colocate/group_stable?db_id=<dbId>&group_id=<grpId>​'
    # 标记为不稳定
    curl -XPOST --location-trusted -u<username>:<password> ​'http://<fe_host>:<fe_http_port>/api/colocate/group_unstable?db_id=<dbId>&group_id=<grpId>​'
    ~~~

    如果返回结果为 `200`，则成功将组标记为稳定或不稳定。

3. 设置组的数据分布

    此接口允许您强制设置组的数据分布。

    `POST /api/colocate/bucketseq?db_id=10005&group_id= 10008`

    `Body:`

    `[[10004,10002],[10003,10002],[10002,10004],[10003,10002],[10002,10004],[10003,10002],[10003,10004],[10003,10004],[10003,10004],[10002,10004]]`

    `返回：200`
    其中 `Body` 是 `BucketsSequence`，表示为嵌套数组，其中包含存储桶切片所在的 BE 的 ID。

    > 请注意，要使用此命令，您可能需要将 FE 配置中的 `disable_colocate_relocate` 和 `disable_colocate_balance` 设置为 true，即禁用系统执行自动的副本重定位和平衡。否则，在修改后可能会被系统自动重置。