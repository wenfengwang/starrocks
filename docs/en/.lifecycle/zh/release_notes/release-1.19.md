---
displayed_sidebar: English
---

# StarRocks 1.19 版本

## 1.19.0

发布日期：2021年10月22日

### 新功能

* 实现全局运行时筛选器，可为洗牌联接启用运行时筛选器。
* 默认启用 CBO Planner，改进了并置联接、存储桶洗牌、统计信息估算等功能。
* [实验功能] 主键表发布：为更好地支持实时/频繁更新功能，StarRocks新增了一种表类型：主键表。主键表支持Stream Load、Broker Load、Routine Load，并基于Flink-cdc为MySQL数据提供了二级同步工具。
* [实验功能] 支持外部表的写入功能。支持通过外部表将数据写入另一个StarRocks集群表，以解决读写分离需求，并提供更好的资源隔离。

### 改进

#### StarRocks

* 性能优化。
  * count distinct int语句
  * group by int语句
  * or语句
* 优化磁盘平衡算法。在单台计算机添加磁盘后，数据可以自动平衡。
* 支持部分列导出。
* 优化show processlist以显示特定SQL。
* 在SET_VAR中支持多个变量设置。
* 改进错误报告信息，包括table_sink、routine load、物化视图创建等。

#### StarRocks-DataX连接器

* 支持设置间隔刷新StarRocks-DataX Writer。

### Bug修复

* 修复数据恢复操作完成后无法自动创建动态分区表的问题。[#337](https://github.com/StarRocks/starrocks/issues/337)
* 修复CBO打开后row_number函数报错的问题。
* 修复统计信息采集导致FE卡顿的问题。
* 修复set_var对会话生效但语句无效的问题。
* 修复Hive分区外部表中select count(*)返回异常的问题。

## 1.19.1

发布日期：2021年11月2日

### 改进

* 优化`show frontends`的性能。[#507](https://github.com/StarRocks/starrocks/pull/507) [#984](https://github.com/StarRocks/starrocks/pull/984)
* 增加慢查询监控。#[502](https://github.com/StarRocks/starrocks/pull/502)#[891](https://github.com/StarRocks/starrocks/pull/891)
* 优化Hive外部元数据的获取，实现并行获取。#[425](https://github.com/StarRocks/starrocks/pull/425) [#451](https://github.com/StarRocks/starrocks/pull/451)

### Bug修复

* 修复Thrift协议兼容性问题，使Hive外部表可以与Kerberos连接。#[184](https://github.com/StarRocks/starrocks/pull/184)#[947](https://github.com/StarRocks/starrocks/pull/947)#[995](https://github.com/StarRocks/starrocks/pull/995)#[999](https://github.com/StarRocks/starrocks/pull/999)
* 修复视图创建过程中的若干bug。#[972](https://github.com/StarRocks/starrocks/pull/972)#[987](https://github.com/StarRocks/starrocks/pull/987)#[1001](https://github.com/StarRocks/starrocks/pull/1001)
* 修复FE无法灰度升级的问题。#[485](https://github.com/StarRocks/starrocks/pull/485)#[890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

发布日期：2021年11月20日

### 改进

* 存储桶洗牌联接支持右联接和完全外联接。#[1209](https://github.com/StarRocks/starrocks/pull/1209)  #[31234](https://github.com/StarRocks/starrocks/pull/1234)

### Bug修复

* 修复重复节点无法进行谓词下推的问题。#[1410](https://github.com/StarRocks/starrocks/pull/1410)#[1417](https://github.com/StarRocks/starrocks/pull/1417)
* 修复导入过程中集群更改领导节点时例程加载可能丢失数据的问题。#[1074](https://github.com/StarRocks/starrocks/pull/1074)#[1272](https://github.com/StarRocks/starrocks/pull/1272)
* 修复创建视图无法支持union的问题。#[1083](https://github.com/StarRocks/starrocks/pull/1083)
* 修复Hive外部表的若干稳定性问题。#[1408](https://github.com/StarRocks/starrocks/pull/1408)
* 修复group by view的问题。#[1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

发布日期：2021年11月30日

### 改进

* 升级jprotobuf版本以提高安全性。#[1506](https://github.com/StarRocks/starrocks/issues/1506)

### Bug修复

* 修复了按结果分组的一些问题。
* 修复分组集的一些问题。#[1395](https://github.com/StarRocks/starrocks/issues/1395)#[1119](https://github.com/StarRocks/starrocks/pull/1119)
* 修复部分date_format指标的问题。
* 修复了聚合流式处理的边界条件问题。#[1584](https://github.com/StarRocks/starrocks/pull/1584)
* 详情请参阅[链接](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)

## 1.19.4

发布日期：2021年12月9日

### 改进

* 支持将varchar强制转换为bitmap。#[1941](https://github.com/StarRocks/starrocks/pull/1941)
* 更新Hive外部表的访问策略。#[1394](https://github.com/StarRocks/starrocks/pull/1394)#[1807](https://github.com/StarRocks/starrocks/pull/1807)

### Bug修复

* 修复谓词Cross Join查询结果错误的bug。#[1918](https://github.com/StarRocks/starrocks/pull/1918)
* 修复十进制类型和时间类型转换的bug。#[1709](https://github.com/StarRocks/starrocks/pull/1709)#[1738](https://github.com/StarRocks/starrocks/pull/1738)
* 修复并置联接/复制联接选择错误的bug。#[1727](https://github.com/StarRocks/starrocks/pull/1727)
* 修复了若干计划成本计算问题。

## 1.19.5

发布日期：2021年12月20日

### 改进

* 优化洗牌联接的计划。#[2184](https://github.com/StarRocks/starrocks/pull/2184)
* 优化多个大文件导入。#[2067](https://github.com/StarRocks/starrocks/pull/2067)

### Bug修复

* 升级Log4j2到2.17.0，修复安全漏洞。#[2284](https://github.com/StarRocks/starrocks/pull/2284)#[2290](https://github.com/StarRocks/starrocks/pull/2290)
* 修复Hive外部表中空分区的问题。#[707](https://github.com/StarRocks/starrocks/pull/707)#[2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

发布日期：2022年3月18日

### Bug修复

已修复以下问题：

* 数据格式在不同版本中会产生不同的结果。#[4165](https://github.com/StarRocks/starrocks/pull/4165)
* BE节点可能因为错误删除Parquet文件而失败。#[3521](https://github.com/StarRocks/starrocks/pull/3521)
