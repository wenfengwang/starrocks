---
displayed_sidebar: English
---

# 星石版本 1.19

## 1.19.0

发布日期：2021年10月22日

### 新特性

* 实现全局运行时过滤器，可为 shuffle join 启用运行时过滤器。
* CBO Planner 默认启用，改进了 colocated join、bucket shuffle、统计信息估计等功能。
* 【实验功能】主键表发布：为了更好地支持实时/频繁更新特性，StarRocks 新增了一种表类型：主键表。主键表支持 Stream Load、Broker Load、Routine Load，并且还提供了基于 Flink-cdc 的 MySQL 数据秒级同步工具。
* 【实验功能】支持外部表的写入功能。支持通过外部表将数据写入另一个 StarRocks 集群表，解决读写分离需求，提供更好的资源隔离。

### 改进

#### StarRocks

* 性能优化：
  * - count distinct int 语句
  * - group by int 语句
  * - or 语句
* 优化磁盘平衡算法，单机添加磁盘后可自动平衡数据。
* 支持部分列导出。
* 优化 show processlist 以显示具体的 SQL。
* 在 SET_VAR 中支持多变量设置。
* 改善错误报告信息，包括 table_sink、routine load、物化视图的创建等。

#### StarRocks-DataX 连接器

* 支持设置 StarRocks-DataX Writer 的刷新间隔。

### Bug 修复

* 修复数据恢复操作完成后无法自动创建动态分区表的问题。[# 337](https://github.com/StarRocks/starrocks/issues/337)
* 修复开启 CBO 后 row_number 函数报错的问题。
* 修复因统计信息收集导致 FE 卡顿的问题。
* 修复 set_var 对会话生效但对语句不生效的问题。
* 修复 Hive 分区外部表 select count(*) 返回异常的问题。

## 1.19.1

发布日期：2021年11月2日

### 改进

* 优化 `show frontends` 的性能。 [# 507](https://github.com/StarRocks/starrocks/pull/507) [# 984](https://github.com/StarRocks/starrocks/pull/984)
* 增加慢查询监控。 [# 502](https://github.com/StarRocks/starrocks/pull/502) [# 891](https://github.com/StarRocks/starrocks/pull/891)
* 优化获取Hive外部元数据，实现并行抓取。[#425](https://github.com/StarRocks/starrocks/pull/425) [#451](https://github.com/StarRocks/starrocks/pull/451)

### Bug 修复

* 修复问题Thrift协议兼容性，使得Hive外部表能够与Kerberos连接。[# 184](https://github.com/StarRocks/starrocks/pull/184) [# 947](https://github.com/StarRocks/starrocks/pull/947) [# 995](https://github.com/StarRocks/starrocks/pull/995) [# 999](https://github.com/StarRocks/starrocks/pull/999)
* 修复视图创建中的多个错误。 [# 972](https://github.com/StarRocks/starrocks/pull/972) [# 987](https://github.com/StarRocks/starrocks/pull/987)[# 1001](https://github.com/StarRocks/starrocks/pull/1001)
* 修复 FE 灰度升级无法进行的问题。[#485](https://github.com/StarRocks/starrocks/pull/485) [#890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

发布日期：2021年11月20日

### 改进

* bucket shuffle join 支持**右连接**和**全外连接**。[# 1209](https://github.com/StarRocks/starrocks/pull/1209)  [# 31234](https://github.com/StarRocks/starrocks/pull/1234)

### Bug 修复

* 修复重复节点无法进行谓词下推的问题[# 1410](https://github.com/StarRocks/starrocks/pull/1410) [# 1417](https://github.com/StarRocks/starrocks/pull/1417)
* 修复例行加载过程中，集群更换 leader 节点可能导致数据丢失的问题。[# 1074](https://github.com/StarRocks/starrocks/pull/1074) [# 1272](https://github.com/StarRocks/starrocks/pull/1272)
* 修复创建视图时不支持 union 的问题。 [# 1083](https://github.com/StarRocks/starrocks/pull/1083)
* 修复一些Hive外部表的稳定性问题[# 1408](https://github.com/StarRocks/starrocks/pull/1408)
* 修复一个问题，使用[# 1231](https://github.com/StarRocks/starrocks/pull/1231)来查看

## 1.19.3

发布日期：2021年11月30日

### 改进

* 升级 jprotobuf 版本以提高安全性 [\# 1506](https://github.com/StarRocks/starrocks/issues/1506)

### Bug 修复

* 修复 group by 结果正确性的一些问题。
* 修复 grouping sets 的一些问题。[#1395](https://github.com/StarRocks/starrocks/issues/1395) [#1119](https://github.com/StarRocks/starrocks/pull/1119)
* 修复 date_format 的一些指标问题。
* 修复聚合流的边界条件问题[# 1584](https://github.com/StarRocks/starrocks/pull/1584)
* 详情请参考[链接](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)。

## 1.19.4

发布日期：2021年12月9日

### 改进

* 支持 cast(varchar as bitmap) [\#1941](https://github.com/StarRocks/starrocks/pull/1941)
* 更新Hive外部表的访问策略。[# 1394](https://github.com/StarRocks/starrocks/pull/1394) [# 1807](https://github.com/StarRocks/starrocks/pull/1807)

### Bug 修复

* 修复带有谓词的 Cross Join 查询结果错误的 bug。[# 1918](https://github.com/StarRocks/starrocks/pull/1918)
* 修复小数类型与时间类型转换的 bug。[# 1709](https://github.com/StarRocks/starrocks/pull/1709) [# 1738](https://github.com/StarRocks/starrocks/pull/1738)
* 修复 colocate join/replicate join 选择错误的 bug [# 1727](https://github.com/StarRocks/starrocks/pull/1727)
* 修复几个计划成本计算问题。

## 1.19.5

发布日期：2021年12月20日

### 改进

* 优化 shuffle join 的计划。[# 2184](https://github.com/StarRocks/starrocks/pull/2184)
* 优化多个大文件的导入。 [#2067](https://github.com/StarRocks/starrocks/pull/2067)

### Bug 修复

* 升级 Log4j2 至 2.17.0，修复安全漏洞[ #2284](https://github.com/StarRocks/starrocks/pull/2284)[ #2290](https://github.com/StarRocks/starrocks/pull/2290)
* 修复问题，Hive外部表空分区的问题[# 707](https://github.com/StarRocks/starrocks/pull/707)[# 2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

发布日期：2022年3月18日

### Bug 修复

修复了以下错误：

* - dataformat 在不同版本中可能产生不同结果。 [#4165](https://github.com/StarRocks/starrocks/pull/4165)
* - BE 节点可能因为数据加载过程中误删除 Parquet 文件而失败。[#3521](https://github.com/StarRocks/starrocks/pull/3521)
