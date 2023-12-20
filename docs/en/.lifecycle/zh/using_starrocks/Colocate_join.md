---
displayed_sidebar: English
---

# Colocate Join

对于 shuffle join 和 broadcast join，如果满足 join 条件，两个参与 join 的表的数据行会被合并到单个节点上完成 join 操作。这两种 join 方法都无法避免因节点间数据网络传输而产生的延迟或开销。

核心思想是保持同一 Colocation Group 中的表具有一致的分桶键、副本数量和副本放置策略。如果 join 列是分桶键，计算节点只需执行本地 join，无需从其他节点获取数据。Colocate Join 支持等值连接。

本文档介绍了 Colocate Join 的原理、实现、使用和注意事项。

## 术语

* **Colocation Group (CG)**：一个 CG 包含一个或多个表。CG 内的表具有相同的分桶策略和副本放置策略，并使用 Colocation Group Schema 描述。
* **Colocation Group Schema (CGS)**：一个 CGS 包含 CG 的分桶键、桶数量和副本数量。

## 原理

Colocate Join 的目的是将一组具有相同 CGS 的表形成一个 CG，并确保这些表对应的 bucket 副本分布在相同的 BE 节点集上。当 CG 中的表在分桶列上执行 Join 操作时，可以直接进行本地数据连接，节省了节点间数据传输的时间。

Bucket Seq 通过 `hash(key) mod buckets` 计算得出。假设一个表有 8 个 bucket，则存在 \[0, 1, 2, 3, 4, 5, 6, 7\] 共 8 个 bucket，每个 Bucket 可能包含一个或多个子表，子表的数量取决于分区的数量。如果是多分区表，则会有多个 tablet。

为了保证数据分布的一致性，同一 CG 内的表必须遵循以下规则：

1. 同一 CG 内的表必须具有相同的分桶键（类型、数量、顺序）和相同数量的 bucket，以便可以逐一控制多个表的数据分片。分桶键是在建表语句 `DISTRIBUTED BY HASH(col1, col2, ...)` 中指定的列。分桶键决定了哪些数据列会被哈希到不同的 Bucket Seq 中。同一 CG 内的表的分桶键名称可以不同。创建语句中的分桶列可以不同，但在 `DISTRIBUTED BY HASH(col1, col2, ...)` 中相应数据类型的顺序必须完全相同。
2. 同一 CG 内的表必须具有相同数量的分区副本。如果不这样，可能会出现某个 tablet 副本在同一 BE 的分区中没有对应副本的情况。
3. 同一 CG 内的表可以有不同数量的分区和不同的分区键。

创建表时，通过表的 PROPERTIES 中的属性 `"colocate_with" = "group_name"` 指定 CG。如果 CG 不存在，则意味着该表是 CG 的第一个表，称为 Parent Table。Parent Table 的数据分布（分桶键的类型、数量和顺序，副本数量和 bucket 数量）决定了 CGS。如果 CG 已存在，则检查表的数据分布是否与 CGS 一致。

同一 CG 内的表的副本放置满足以下条件：

1. 所有表的 Bucket Seq 与 BE 节点之间的映射关系与 Parent Table 相同。
2. Parent Table 中所有 Partition 的 Bucket Seq 与 BE 节点的映射关系与第一个 Partition 相同。
3. Parent Table 的第一个 Partition 的 Bucket Seq 与 BE 节点之间的映射使用原生的 Round Robin 算法确定。

一致的数据分布和映射确保了具有相同分桶键值的数据行落在同一 BE 上。因此，使用分桶键进行 join 时，只需进行本地 join。

## 使用方法

### 创建表

创建表时，可以在 PROPERTIES 中指定属性 `"colocate_with" = "group_name"`，以表明该表是 Colocate Join 表，并属于指定的 Colocation Group。
> **注意**
> 从版本 2.5.4 开始，可以在不同数据库的表之间执行 Colocate Join。你只需要在创建表时指定相同的 `colocate_with` 属性即可。

例如：

```SQL
CREATE TABLE tbl (k1 int, v1 int sum)
DISTRIBUTED BY HASH(k1)
BUCKETS 8
PROPERTIES(
    "colocate_with" = "group1"
);
```

如果指定的 Group 不存在，StarRocks 会自动创建一个只包含当前表的 Group。如果 Group 已存在，StarRocks 会检查当前表是否符合 Colocation Group Schema。如果符合，则会创建该表并将其添加到 Group 中。同时，该表会根据现有 Group 的数据分布规则创建分区和 tablet。

Colocation Group 属于一个数据库。Colocation Group 的名称在数据库内是唯一的。在内部存储中，Colocation Group 的全名是 `dbId_groupName`，但你只需要感知 `groupName`。
> **注意**
> 如果你指定相同的 Colocation Group 来关联不同数据库中的表进行 Colocate Join，Colocation Group 将在每个这样的数据库中存在。你可以运行 `show proc "/colocation_group"` 来检查不同数据库中的 Colocation Group。

### 删除

完全删除是指从回收站彻底删除。通常，使用 `DROP TABLE` 命令删除表后，默认情况下表会在回收站中保留一天，之后才会被彻底删除。当 Group 中的最后一张表被完全删除时，该 Group 也会自动被删除。

### 查看组信息

以下命令允许你查看集群中已存在的 Group 信息。

```Plain
SHOW PROC '/colocation_group';

+-------------+--------------+--------------+------------+----------------+----------+----------+
| GroupId     | GroupName    | TableIds     | BucketsNum | ReplicationNum | DistCols | IsStable |
+-------------+--------------+--------------+------------+----------------+----------+----------+
| 10005.10008 | 10005_group1 | 10007, 10040 | 10         | 3              | int(11)  | true     |
+-------------+--------------+--------------+------------+----------------+----------+----------+
```

* **GroupId**：Group 在集群中的唯一标识符，前半部分是 db id，后半部分是 group id。
* **GroupName**：Group 的全名。
* **TableIds**：Group 中表的 id 列表。
* **BucketsNum**：bucket 的数量。
* **ReplicationNum**：副本的数量。
* **DistCols**：分布列，即 bucketing 列的类型。
* **IsStable**：Group 是否稳定（稳定性的定义见 Colocation 副本平衡和修复部分）。

你可以使用以下命令进一步查看 Group 的数据分布。

```Plain
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
```

* **BucketIndex**：bucket 序列的下标。
* **BackendIds**：bucket 数据片所在的 BE 节点的 id。

> 注意：上述命令需要 ADMIN 权限。普通用户无法访问。

### 修改表组属性

你可以修改表的 Colocation Group 属性。例如：

```SQL
ALTER TABLE tbl SET ("colocate_with" = "group2");
```

如果表之前未被分配到任何 Group，则该命令会检查 Schema 并将表添加到 Group 中（如果 Group 不存在，则会首先创建）。如果表之前已被分配到另一个 Group，则该命令会将表从原 Group 中移除，并添加到新 Group 中（如果 Group 不存在，则会首先创建）。

你也可以使用以下命令移除表的 Colocation 属性。

```SQL
ALTER TABLE tbl SET ("colocate_with" = "");
```

### 其他相关操作

当对具有 Colocation 属性的表使用 `ADD PARTITION` 添加分区或修改副本数量时，StarRocks 会检查操作是否违反了 Colocation Group Schema，并在违反时拒绝该操作。

## Colocation 副本平衡和修复

Colocation 表的副本分布需要遵循 Group schema 中指定的规则，因此在副本修复和平衡方面与普通分片有所不同。

Group 本身具有 `stable` 属性。当 `stable` 为 `true` 时，表示 Group 中的表分片没有发生变化，Colocation 功能正常工作。当 `stable` 为 `false` 时，表示当前 Group 中的某些表分片正在进行修复或迁移，受影响的表的 Colocate Join 将降级为普通 Join。

### 副本修复

副本只能存储在指定的 BE 节点上。StarRocks 会寻找负载最轻的 BE 来替换不可用的 BE（例如，宕机、退役）。替换后，旧 BE 上的所有 bucket 数据片都将被修复。在迁移期间，Group 被标记为 **不稳定**。

### 副本平衡

```
StarRocks 试图将 Colocation 表切片均匀地分布在所有 BE 节点上。普通表的平衡是在副本级别，即每个副本单独寻找一个负载较低的 BE 节点。Colocation 表的平衡是在 Bucket 级别，即 Bucket 内的所有副本都一起迁移。我们使用一种简单的平衡算法，将 `BucketsSequence` 均匀地分布在所有 BE 节点上，而不考虑副本的实际大小，只考虑副本的数量。具体的算法可以在 `ColocateTableBalancer.java` 的代码注释中找到。

> 注 1：当前的 Colocation 副本平衡和修复算法可能不适用于异构部署的 StarRocks 集群。所谓异构部署，是指 BE 节点的磁盘容量、磁盘数量、磁盘类型（SSD 和 HDD）不一致。在异构部署的情况下，可能会出现小容量 BE 节点存储与大容量 BE 节点相同数量的副本的情况。
> 注 2：当 Group 处于 Unstable 状态时，其表的 Join 会降级为普通 Join，这可能会显著降低集群的查询性能。如果您不希望系统自动平衡，请设置 FE 配置 `disable_colocate_balance` 以禁用自动平衡，并在适当的时候重新启用它。（有关详细信息，请参阅高级操作（#高级操作）部分）

## 查询

Colocation 表的查询方式与普通表相同。如果 Colocation 表所在的 Group 处于 Unstable 状态，则会自动降级为普通 Join，如下例所示。

表 1：

```SQL
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
```

表 2：

```SQL
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
```

查看查询计划：

```Plain
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
```

如果 Colocate Join 生效，Hash Join 节点显示 `colocate: true`。

如果不生效，查询计划如下：

```Plain
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
```

HASH JOIN 节点会显示相应的原因：`colocate: false, reason: group is not stable`。同时会生成一个 EXCHANGE 节点。

## 高级操作

### FE 配置项

* **disable_colocate_relocate**

是否禁用 StarRocks 的自动 Colocation 副本修复。默认为 false，表示开启。该参数仅影响 Colocation 表的副本修复，而不影响普通表。

* **disable_colocate_balance**

是否禁用 StarRocks 的自动 Colocation 副本平衡。默认为 false，表示开启。该参数只影响 Colocation 表的副本平衡，不影响普通表。

* **disable_colocate_join**

您可以通过更改此变量来禁用会话粒度的 Colocate Join。

### HTTP Restful API

StarRocks 提供了多个与 Colocate Join 相关的 HTTP Restful API，用于查看和修改 Colocation Group。

该 API 在 FE 上实现，可以使用具有 ADMIN 权限的 `fe_host:fe_http_port` 进行访问。

1. 查看集群所有 Colocation 信息

   ```bash
   curl --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate'  
   ```

   ```JSON
   // 返回 Json 格式的内部 Colocation 信息。
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
   ```

2. 将组标记为稳定或不稳定

   ```bash
   # 标记为稳定
   curl -XPOST --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_stable?db_id=<dbId>&group_id=<grpId>'
   # 标记为不稳定
   curl -XPOST --location-trusted -u<username>:<password> 'http://<fe_host>:<fe_http_port>/api/colocate/group_unstable?db_id=<dbId>&group_id=<grpId>'
   ```

   如果返回结果为 `200`，则该组成功标记为稳定或不稳定。

3. 设置 Group 的数据分布

   该接口允许您强制设置一个组的数据分布。

   `POST /api/colocate/bucketseq?db_id=10005&group_id=10008`

   `Body:`

   `[[10004,10002],[10003,10002],[10002,10004],[10003,10002],[10002,10004],[10003,10002],[10003,10004],[10003,10004],[10003,10004],[10002,10004]]`

   `返回：200`

   其中 `Body` 是表示为嵌套数组的 `BucketsSequence`，以及存放分桶切片的 BE 的 ID。

   > 注意：要使用此命令，您可能需要将 FE 配置 `disable_colocate_relocate` 和 `disable_colocate_balance` 设置为 true，即禁用系统执行自动 Colocation 副本修复和平衡。否则，可能会在修改后被系统自动重置。
```markdown
   [[10004,10002],[10003,10002],[10002,10004],[10003,10002],[10002,10004],[10003,10002],[10003,10004],[10003,10004],[10003,10004],[10002,10004]]

   返回：200

   其中`Body`是`BucketsSequence`表示为嵌套数组，以及桶切片所在的BE的id。

      > 注意，要使用此命令，您可能需要将FE配置`disable_colocate_relocate`和`disable_colocate_balance`设置为`true`，即禁用系统执行自动Colocation副本修复和平衡。否则，系统可能会在修改后自动重置。