---
displayed_sidebar: English
---

# 使用 CloudCanal 加载数据

## 介绍

CloudCanal 社区版是由 [ClouGence Co., Ltd](https://www.cloudcanalx.com) 发布的一款免费的数据迁移和同步平台，它整合了架构迁移、全量数据迁移、验证、纠正以及实时增量同步功能。
CloudCanal 帮助用户以简单的方式构建现代数据栈。
![image.png](../assets/3.11-1.png)

## 下载

[CloudCanal 下载链接](https://www.cloudcanalx.com)

[CloudCanal 快速开始指南](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)

## 功能描述

- 强烈推荐使用 CloudCanal 版本 2.2.5.0 或更高版本，以便高效地将数据导入 StarRocks。
- 当使用 CloudCanal 将**增量数据**导入 StarRocks 时，建议控制数据摄入频率。CloudCanal 向 StarRocks写入数据的默认导入频率可以通过调整 `realFlushPauseSec` 参数来改变，该参数默认设置为 10 秒。
- 在当前社区版中，最大内存配置为 2GB，如果 DataJobs 遇到 OOM 异常或显著的 GC 暂停，建议减小批处理大小以减少内存使用。
  - 对于全量数据任务（Full DataTask），你可以调整 fullBatchSize 和 fullRingBufferSize 参数。
  - 对于增量数据任务（Incremental DataTask），可以相应地调整 increBatchSize 和 increRingBufferSize 参数。
- 支持的源端点和特性：

  |源端点\功能|架构迁移|完整数据|增量|验证|
|---|---|---|---|---|
  |Oracle|是|是|是|是|
  |PostgreSQL|是|是|是|是|
  |Greenplum|是|是|否|是|
  |MySQL|是|是|是|是|
  |卡夫卡|否|否|是|否|
  |OceanBase|是|是|是|是|
  |PolarDb for MySQL|是|是|是|是|
  |Db2|是|是|是|是|

## 典型示例

CloudCanal 允许用户在可视化界面中进行操作，用户可以无缝地添加数据源（DataSource）并通过可视化界面创建数据作业（DataJobs）。这使得自动化架构迁移、全量数据迁移和实时增量同步成为可能。以下示例演示了如何将数据从 MySQL 迁移并同步到 StarRocks。其他数据源与 StarRocks 之间的数据同步过程也是类似的。

### 先决条件

首先，参考[CloudCanal Quick Start](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)完成安装和部署CloudCanal社区版。

### 添加数据源

- 登录 CloudCanal 平台
- 进入**数据源管理**-	extgreater **添加数据源**
- 在自建数据库选项中选择 **StarRocks**

![image.png](../assets/3.11-2.png)

> 提示：
- 客户端地址：指 StarRocks 服务器 MySQL 客户端服务的端口地址。CloudCanal 主要通过此地址查询数据库表的元数据信息。

- HTTP 地址：主要用于接收 CloudCanal 的数据导入请求。

### 创建数据作业

成功添加数据源后，您可以按照以下步骤创建数据迁移和同步作业（DataJob）。

- 进入 **数据作业管理** -	extgreater 在 CloudCanal 中创建**数据作业**
- 为数据作业选择源数据库和目标数据库
- 点击“下一步”

![image.png](../assets/3.11-3.png)

- 选择**增量**并启用**全量数据**
- 选择“DDL 同步”
- 点击“下一步”

![image.png](../assets/3.11-4.png)

- 选择您想要订阅的源表。请注意，架构迁移后自动创建的目标 StarRocks 表是主键表，因此当前不支持没有主键的源表**

- 点击“下一步”

![image.png](../assets/3.11-5.png)

- 配置列映射
- 点击“下一步”

![image.png](../assets/3.11-6.png)

- 创建数据作业

![image.png](../assets/3.11-7.png)

- 检查数据作业的状态。数据作业创建后将自动进入架构迁移、全量数据和增量阶段

![image.png](../assets/3.11-8.png)
