---
displayed_sidebar: "Chinese"
---

# StarRocks 版本 1.19

## 1.19.0

发布日期：2021年10月22日

### 新功能

* 实现全局运行时过滤器，可以为洗牌连接启用运行时过滤器。
* CBO Planner 默认启用，改进了共同定位连接、桶洗牌、统计信息估算等。
* [实验性功能] 主键表发布：为了更好地支持实时/频繁更新功能，StarRocks 添加了一种新的表类型：主键表。主键表支持流式加载、Broker 加载、例行加载，并且还提供了基于 Flink-cdc 的 MySQL 数据的二级同步工具。
* [实验性功能] 支持外部表的写入功能。支持通过外部表将数据写入到另一个 StarRocks 集群表，以解决读/写分离需求，并提供更好的资源隔离。

### 改进

#### StarRocks

* 性能优化。
  * 计算不同的 int 语句
  * 按 int 语句分组
  * 或语句
* 优化磁盘平衡算法。在为单台机器添加磁盘后，数据可以自动平衡。
* 支持部分列导出。
* 优化 show processlist 以显示特定 SQL。
* 在 SET_VAR 中支持多个变量设置。
* 改进错误报告信息，包括 table_sink、例行加载、物化视图创建等。

#### StarRocks-DataX Connector

* 支持设置间隔刷新 StarRocks-DataX Writer。

### 缺陷修复

* 修复数据恢复操作完成后无法自动创建动态分区表的问题。[# 337](https://github.com/StarRocks/starrocks/issues/337)
* 修复打开 CBO 后 row_number 函数报错的问题。
* 修复 FE 被卡住的问题，原因是统计信息收集。
* 修复 set_var 对会话生效但对语句不生效的问题。
* 修复 Hive 分区外部表上 select count(*) 返回异常的问题。

## 1.19.1

发布日期：2021年11月2日

### 改进

* 优化 `show frontends` 的性能。[# 507](https://github.com/StarRocks/starrocks/pull/507) [# 984](https://github.com/StarRocks/starrocks/pull/984)
* 添加慢查询的监控。[# 502](https://github.com/StarRocks/starrocks/pull/502) [# 891](https://github.com/StarRocks/starrocks/pull/891)
* 优化 Hive 外部元数据的获取以实现并行获取。[# 425](https://github.com/StarRocks/starrocks/pull/425) [# 451](https://github.com/StarRocks/starrocks/pull/451)

### 缺陷修复

* 修复 Thrift 协议兼容性问题，使得 Hive 外部表可以与 Kerberos 连接。[# 184](https://github.com/StarRocks/starrocks/pull/184) [# 947](https://github.com/StarRocks/starrocks/pull/947) [# 995](https://github.com/StarRocks/starrocks/pull/995) [# 999](https://github.com/StarRocks/starrocks/pull/999)
* 修复视图创建中的若干问题。[# 972](https://github.com/StarRocks/starrocks/pull/972) [# 987](https://github.com/StarRocks/starrocks/pull/987) [# 1001](https://github.com/StarRocks/starrocks/pull/1001)
* 修复 FE 无法在灰度中升级的问题。[# 485](https://github.com/StarRocks/starrocks/pull/485) [# 890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

发布日期：2021年11月20日

### 改进

* 桶洗牌连接支持右连接和全外连接 [# 1209](https://github.com/StarRocks/starrocks/pull/1209)  [# 31234](https://github.com/StarRocks/starrocks/pull/1234)

### 缺陷修复

* 修复重复节点无法执行谓词下推的问题[# 1410](https://github.com/StarRocks/starrocks/pull/1410) [# 1417](https://github.com/StarRocks/starrocks/pull/1417)
* 修复例行加载在导入期间替换主导节点时可能丢失数据的问题。[# 1074](https://github.com/StarRocks/starrocks/pull/1074) [# 1272](https://github.com/StarRocks/starrocks/pull/1272)
* 修复创建视图不支持 union 的问题 [# 1083](https://github.com/StarRocks/starrocks/pull/1083)
* 修复 Hive 外部表的若干稳定性问题[# 1408](https://github.com/StarRocks/starrocks/pull/1408)
* 修复分组视图的问题[# 1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

发布日期：2021年11月30日

### 改进

* 升级 jprotobuf 版本以提高安全性 [# 1506](https://github.com/StarRocks/starrocks/issues/1506)

### 缺陷修复

* 修复分组结果正确性的若干问题
* 修复分组集的若干问题[# 1395](https://github.com/StarRocks/starrocks/issues/1395) [# 1119](https://github.com/StarRocks/starrocks/pull/1119)
* 修复 date_format 的若干指标问题
* 修复聚合流的边界条件问题[# 1584](https://github.com/StarRocks/starrocks/pull/1584)
* 详细信息请参阅[链接](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)

## 1.19.4

发布日期：2021年12月9日

### 改进

* 支持 cast(varchar as bitmap) [# 1941](https://github.com/StarRocks/starrocks/pull/1941)
* 更新 Hive 外部表的访问策略 [# 1394](https://github.com/StarRocks/starrocks/pull/1394) [# 1807](https://github.com/StarRocks/starrocks/pull/1807)

### 缺陷修复

* 修复谓词 Cross Join 查询结果错误的问题 [# 1918](https://github.com/StarRocks/starrocks/pull/1918)
* 修复 decimal 类型和 time 类型转换的问题 [# 1709](https://github.com/StarRocks/starrocks/pull/1709) [# 1738](https://github.com/StarRocks/starrocks/pull/1738)
* 修复共同定位连接/复制连接选择错误的问题 [# 1727](https://github.com/StarRocks/starrocks/pull/1727)
* 修复若干计划成本计算问题

## 1.19.5

发布日期：2021年12月20日

### 改进

* 优化洗牌连接的计划 [# 2184](https://github.com/StarRocks/starrocks/pull/2184)
* 优化多个大文件导入 [# 2067](https://github.com/StarRocks/starrocks/pull/2067)

### 缺陷修复

* 升级 Log4j2 至 2.17.0，修复安全漏洞[# 2284](https://github.com/StarRocks/starrocks/pull/2284)[# 2290](https://github.com/StarRocks/starrocks/pull/2290)
* 修复 Hive 外部表中空分区的问题[# 707](https://github.com/StarRocks/starrocks/pull/707)[# 2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

发布日期：2022年3月18日

### 缺陷修复

以下问题已修复：

* 数据格式在不同版本下会产生不同结果。[#4165](https://github.com/StarRocks/starrocks/pull/4165)
* 由于数据加载期间错误删除 Parquet 文件，BE 节点可能会失败。[#3521](https://github.com/StarRocks/starrocks/pull/3521)