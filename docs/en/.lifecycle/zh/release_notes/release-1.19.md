---
displayed_sidebar: English
---

# StarRocks 版本 1.19

## 1.19.0

发布日期：2021年10月22日

### 新功能

* 实现全局运行时过滤器，可以为 shuffle join 启用运行时过滤器。
* 默认启用 CBO Planner，改进了 colocated join、bucket shuffle、统计信息估计等。
* 【实验功能】主键表发布：为了更好地支持实时/频繁更新特性，StarRocks 新增了一种表类型：主键表。主键表支持 Stream Load、Broker Load、Routine Load，并且提供了基于 Flink-cdc 的 MySQL 数据秒级同步工具。
* [实验功能] 支持外部表的写入功能。支持通过外部表向另一个 StarRocks 集群表写入数据，解决读写分离需求，提供更好的资源隔离。

### 改进

#### StarRocks

* 性能优化。
  * count distinct int 语句
  * group by int 语句
  * or 语句
* 优化磁盘平衡算法。单机添加磁盘后可以自动平衡数据。
* 支持部分列导出。
* 优化 show processlist 以显示具体 SQL。
* 支持在 SET_VAR 中设置多个变量。
* 改进错误报告信息，包括 table_sink、routine load、物化视图创建等。

#### StarRocks-DataX Connector

* 支持设置 StarRocks-DataX Writer 的刷新间隔。

### Bug 修复

* 修复数据恢复操作完成后无法自动创建动态分区表的问题。[#337](https://github.com/StarRocks/starrocks/issues/337)
* 修复 CBO 开启后 row_number 函数报错的问题。
* 修复 FE 因统计信息收集而卡住的问题。
* 修复 set_var 对 session 生效但对 statements 不生效的问题。
* 修复 Hive 分区外部表 select count(*) 返回异常的问题。

## 1.19.1

发布日期：2021年11月2日

### 改进

* 优化 `show frontends` 的性能。[#507](https://github.com/StarRocks/starrocks/pull/507) [#984](https://github.com/StarRocks/starrocks/pull/984)
* 添加慢查询监控。[#502](https://github.com/StarRocks/starrocks/pull/502) [#891](https://github.com/StarRocks/starrocks/pull/891)
* 优化 Hive 外部元数据的获取，实现并行获取。[#425](https://github.com/StarRocks/starrocks/pull/425) [#451](https://github.com/StarRocks/starrocks/pull/451)

### Bug 修复

* 修复 Thrift 协议兼容性问题，使 Hive 外部表能够与 Kerberos 连接。[#184](https://github.com/StarRocks/starrocks/pull/184) [#947](https://github.com/StarRocks/starrocks/pull/947) [#995](https://github.com/StarRocks/starrocks/pull/995) [#999](https://github.com/StarRocks/starrocks/pull/999)
* 修复视图创建中的若干问题。[#972](https://github.com/StarRocks/starrocks/pull/972) [#987](https://github.com/StarRocks/starrocks/pull/987) [#1001](https://github.com/StarRocks/starrocks/pull/1001)
* 修复 FE 在灰度升级中的问题。[#485](https://github.com/StarRocks/starrocks/pull/485) [#890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

发布日期：2021年11月20日

### 改进

* bucket shuffle join 支持 right join 和 full outer join。[#1209](https://github.com/StarRocks/starrocks/pull/1209) [#1234](https://github.com/StarRocks/starrocks/pull/1234)

### Bug 修复

* 修复 repeat node 无法进行谓词下推的问题。[#1410](https://github.com/StarRocks/starrocks/pull/1410) [#1417](https://github.com/StarRocks/starrocks/pull/1417)
* 修复导入过程中集群更换 leader 节点时，Routine Load 可能丢失数据的问题。[#1074](https://github.com/StarRocks/starrocks/pull/1074) [#1272](https://github.com/StarRocks/starrocks/pull/1272)
* 修复创建视图不支持 union 的问题。[#1083](https://github.com/StarRocks/starrocks/pull/1083)
* 修复 Hive 外部表的一些稳定性问题。[#1408](https://github.com/StarRocks/starrocks/pull/1408)
* 修复按视图分组的问题。[#1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

发布日期：2021年11月30日

### 改进

* 升级 jprotobuf 版本以提高安全性。[#1506](https://github.com/StarRocks/starrocks/issues/1506)

### Bug 修复

* 修复 group by 结果正确性的一些问题。
* 修复与 grouping sets 相关的一些问题。[#1395](https://github.com/StarRocks/starrocks/issues/1395) [#1119](https://github.com/StarRocks/starrocks/pull/1119)
* 修复 date_format 的一些指标问题。
* 修复聚合流的边界条件问题。[#1584](https://github.com/StarRocks/starrocks/pull/1584)
* 详情请参考[链接](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)

## 1.19.4

发布日期：2021年12月9日

### 改进

* 支持 cast(varchar as bitmap)。[#1941](https://github.com/StarRocks/starrocks/pull/1941)
* 更新 Hive 外部表的访问策略。[#1394](https://github.com/StarRocks/starrocks/pull/1394) [#1807](https://github.com/StarRocks/starrocks/pull/1807)

### Bug 修复

* 修复带有谓词的 Cross Join 查询结果错误的 bug。[#1918](https://github.com/StarRocks/starrocks/pull/1918)
* 修复 decimal 类型和 time 类型转换的 bug。[#1709](https://github.com/StarRocks/starrocks/pull/1709) [#1738](https://github.com/StarRocks/starrocks/pull/1738)
* 修复 colocate join/replicate join 选择错误的 bug。[#1727](https://github.com/StarRocks/starrocks/pull/1727)
* 修复几个计划成本计算问题。

## 1.19.5

发布日期：2021年12月20日

### 改进

* 优化 shuffle join 的计划。[#2184](https://github.com/StarRocks/starrocks/pull/2184)
* 优化多个大文件导入。[#2067](https://github.com/StarRocks/starrocks/pull/2067)

### Bug 修复

* 升级 Log4j2 至 2.17.0，修复安全漏洞。[#2284](https://github.com/StarRocks/starrocks/pull/2284) [#2290](https://github.com/StarRocks/starrocks/pull/2290)
* 修复 Hive 外部表空分区的问题。[#707](https://github.com/StarRocks/starrocks/pull/707) [#2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

发布日期：2022年3月18日

### Bug 修复

修复了以下错误：

* dataformat 在不同版本中可能产生不同结果。[#4165](https://github.com/StarRocks/starrocks/pull/4165)
* BE 节点可能因误删除 Parquet 文件而在数据加载过程中失败。[#3521](https://github.com/StarRocks/starrocks/pull/3521)