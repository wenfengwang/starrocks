---
displayed_sidebar: English
---

# 使用 CloudCanal 加载数据

## 介绍

CloudCanal 社区版是由 [ClouGence Co., Ltd](https://www.cloudcanalx.com) 发布的免费数据迁移和同步平台，集成了 Schema Migration、Full Data Migration、Verification、Correction 和 real-time Incremental Synchronization。
CloudCanal 帮助用户以简单的方式构建现代化的数据栈。
![image.png](../assets/3.11-1.png)

## 下载

[CloudCanal 下载链接](https://www.cloudcanalx.com)

[CloudCanal 快速入门](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)

## 功能描述

- 强烈推荐使用 CloudCanal 版本 2.2.5.0 或更高版本，以高效地将数据导入 StarRocks。
- 使用 CloudCanal 将 **增量数据** 导入 StarRocks 时，建议控制摄取频率。CloudCanal 向 StarRocks 写入数据的默认导入频率可以通过 `realFlushPauseSec` 参数进行调整，默认设置为 10 秒。
- 在当前社区版中，最大内存配置为 2GB，如果 DataJobs 遇到 OOM 异常或显著的 GC 暂停，建议减小批量大小以减少内存使用。
  - 对于 Full DataTask，您可以调整 `fullBatchSize` 和 `fullRingBufferSize` 参数。
  - 对于 Incremental DataTask，可以调整 `increBatchSize` 和 `increRingBufferSize` 参数。
- 支持的源端点和特性：

  |源端点 \ 特性|Schema Migration|Full Data|Incremental|Verification|
|---|---|---|---|---|
  |Oracle|是|是|是|是|
  |PostgreSQL|是|是|是|是|
  |Greenplum|是|是|否|是|
  |MySQL|是|是|是|是|
  |Kafka|否|否|是|否|
  |OceanBase|是|是|是|是|
  |PolarDB for MySQL|是|是|是|是|
  |Db2|是|是|是|是|

## 典型示例

CloudCanal 允许用户在可视化界面中执行操作，用户可以通过可视化界面无缝添加 DataSources 并创建 DataJobs。这使得自动化 Schema Migration、Full Data Migration 和实时 Incremental Synchronization 成为可能。以下示例演示了如何将数据从 MySQL 迁移并同步到 StarRocks。其他数据源与 StarRocks 之间的数据同步过程类似。

### 先决条件

首先，参考 [CloudCanal 快速入门](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start) 完成 CloudCanal 社区版的安装和部署。

### 添加数据源

- 登录 CloudCanal 平台
- 进入 **DataSource Management** -> **Add DataSource**
- 选择 **StarRocks** 自建数据库选项

![image.png](../assets/3.11-2.png)

> 提示：
- 客户端地址：StarRocks 服务器的 MySQL 客户端服务端口地址。CloudCanal 主要使用此地址来查询数据库表的元数据信息。

- HTTP 地址：HTTP 地址主要用于接收 CloudCanal 的数据导入请求。

### 创建 DataJob

成功添加 DataSource 后，您可以按照以下步骤创建数据迁移和同步 DataJob。

- 进入 **DataJob Management** -> **Create DataJob** 在 CloudCanal 中
- 选择 DataJob 的源数据库和目标数据库
- 点击“下一步”

![image.png](../assets/3.11-3.png)

- 选择 **Incremental** 并启用 **Full Data**
- 选择 DDL Sync
- 点击“下一步”

![image.png](../assets/3.11-4.png)

- 选择您要订阅的源表。请注意，Schema Migration 后自动生成的目标 StarRocks 表为主键表，因此当前不支持没有主键的源表**

- 点击“下一步”

![image.png](../assets/3.11-5.png)

- 配置列映射
- 点击“下一步”

![image.png](../assets/3.11-6.png)

- 创建 DataJob

![image.png](../assets/3.11-7.png)

- 检查 DataJob 的状态。创建后的 DataJob 会自动经历 Schema Migration、Full Data 和 Incremental 阶段

![image.png](../assets/3.11-8.png)