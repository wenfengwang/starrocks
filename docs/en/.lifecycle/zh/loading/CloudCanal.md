---
displayed_sidebar: English
---

# 使用 CloudCanal 加载数据

## 介绍

CloudCanal 社区版是由 [ClouGence Co.，Ltd](https://www.cloudcanalx.com) 发布的免费数据迁移和同步平台，集成了架构迁移、全量数据迁移、验证、校正和实时增量同步功能。
CloudCanal 帮助用户以简单的方式构建现代数据堆栈。
![image.png](../assets/3.11-1.png)

## 下载

[CloudCanal 下载链接](https://www.cloudcanalx.com)

[CloudCanal 快速入门](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)

## 功能说明

- 强烈建议使用 CloudCanal 2.2.5.0 或更高版本，以便将数据高效地导入 StarRocks。
- 当使用 CloudCanal 将**增量数据**导入 StarRocks 时，建议控制数据采集频率。CloudCanal 默认的数据写入频率可以通过 `realFlushPauseSec` 参数进行调整，默认为 10 秒。
- 在当前社区版中，最大内存配置为 2GB。如果 DataJobs 遇到 OOM 异常或出现严重的 GC 暂停，建议减小批处理大小，以减少内存使用量。
  - 对于全量数据任务，可以调整 `fullBatchSize` 和 `fullRingBufferSize` 参数。
  - 对于增量数据任务，可以相应地调整 `increBatchSize` 和 `increRingBufferSize` 参数。
- 支持的源端点和功能：

  | 源端点 \ 功能 | 架构迁移 | 全量数据 | 增量 | 验证 |
    | --- | --- | --- | --- | --- |
  | Oracle                     | 是 | 是 | 是 | 是 |
  | PostgreSQL                 | 是 | 是 | 是 | 是 |
  | Greenplum                  | 是 | 是 | 否 | 是 |
  | MySQL                      | 是 | 是 | 是 | 是 |
  | Kafka                      | 否 | 否 | 是 | 否 |
  | OceanBase                  | 是 | 是 | 是 | 是 |
  | PolarDb for MySQL          | 是 | 是 | 是 | 是 |
  | Db2                        | 是 | 是 | 是 | 是 |

## 典型案例

CloudCanal 允许用户在可视化界面中执行操作，用户可以通过可视化界面轻松添加数据源并创建 DataJobs。这样可以实现自动架构迁移、全量数据迁移和实时增量同步。以下示例演示了如何从 MySQL 迁移和同步数据到 StarRocks。其他数据源和 StarRocks 之间的数据同步过程也类似。

### 先决条件

首先，参考 [CloudCanal 快速入门](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start) ，完成 CloudCanal 社区版的安装和部署。

### 添加数据源

- 登录 CloudCanal 平台
- 进入 **数据源管理** -> **添加数据源**
- 从自建数据库选项中选择 **StarRocks**

![image.png](../assets/3.11-2.png)

> 提示：
>
> - 客户端地址：StarRocks 服务器的 MySQL 客户端服务端口地址。CloudCanal 主要使用该地址查询数据库表的元数据信息。
>
> - HTTP 地址：HTTP 地址主要用于接收来自 CloudCanal 的数据导入请求。

### 创建 DataJob

成功添加数据源后，您可以按照以下步骤创建数据迁移和同步 DataJob。

- 进入 **DataJob 管理** -> **创建 DataJob** 在 CloudCanal 中
- 选择 DataJob 的源数据库和目标数据库
- 点击下一步

![image.png](../assets/3.11-3.png)

- 选择 **增量** 并启用 **全量数据**
- 选择 DDL 同步
- 点击下一步

![image.png](../assets/3.11-4.png)

- 选择要订阅的源表。请注意，架构迁移后自动执行的目标 StarRocks 表是主键表，因此暂不支持没有主键的源表**

- 点击下一步

![image.png](../assets/3.11-5.png)

- 配置列映射
- 点击下一步

![image.png](../assets/3.11-6.png)

- 创建 DataJob

![image.png](../assets/3.11-7.png)

- 检查 DataJob 的状态。DataJob 创建后将自动经历架构迁移、全量数据和增量阶段

![image.png](../assets/3.11-8.png)