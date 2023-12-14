---
displayed_sidebar: "Chinese"
---

# 使用CloudCanal加载数据

## 介绍

CloudCanal社区版是由[ClouGence Co., Ltd](https://www.cloudcanalx.com)发布的免费数据迁移与同步平台，集成了模式迁移、完整数据迁移、验证、纠正和实时增量同步的功能。CloudCanal帮助用户以简单的方式构建现代化的数据堆栈。
![image.png](../assets/3.11-1.png)


## 下载

[CloudCanal下载链接](https://www.cloudcanalx.com)

[CloudCanal快速入门](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)

## 功能描述

- 强烈建议使用CloudCanal 2.2.5.0或更高版本以有效地将数据导入StarRocks。
- 在使用CloudCanal将**增量数据**导入StarRocks时，建议控制摄取频率。CloudCanal向StarRocks写入数据的默认导入频率可以通过`realFlushPauseSec`参数进行调整，默认设置为10秒。
- 在当前社区版中，最大内存配置为2GB。如果DataJobs遇到OOM异常或显著的GC暂停，建议减小批处理大小以减少内存使用量。
  - 对于完整DataTask，可以调整`fullBatchSize`和`fullRingBufferSize`参数。
  - 对于增量DataTask，可以相应地调整`increBatchSize`和`increRingBufferSize`参数。
- 支持的源端点和功能：

  | 源端点 \ 功能 | 模式迁移 | 完整数据 | 增量 | 验证 |
    | --- | --- | --- | --- | --- |
  | Oracle       | 是 | 是 | 是 | 是 |
  | PostgreSQL   | 是 | 是 | 是 | 是 |
  | Greenplum    | 是 | 是 | 否 | 是 |
  | MySQL        | 是 | 是 | 是 | 是 |
  | Kafka        | 否 | 否 | 是 | 否 |
  | OceanBase    | 是 | 是 | 是 | 是 |
  | PolarDb for MySQL | 是 | 是 | 是 | 是 |
  | Db2          | 是 | 是 | 是 | 是 |

## 典型示例

CloudCanal允许用户在可视化界面中执行操作，用户可以通过可视化界面轻松添加数据源并创建DataJobs。这样可以实现自动模式迁移、完整数据迁移和实时增量同步。以下示例演示了如何从MySQL迁移和同步数据到StarRocks。对于其他数据源与StarRocks之间的数据同步，流程类似。

### 先决条件

首先，请参考[CloudCanal快速入门](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)完成安装和部署CloudCanal社区版。

### 添加数据源

- 登录CloudCanal平台
- 转至**数据源管理** -> **添加数据源**
- 从自建数据库选项中选择**StarRocks**

![image.png](../assets/3.11-2.png)

> 提示：
>
> - 客户端地址：StarRocks服务器MySQL客户端服务端口的地址。CloudCanal主要使用此地址查询数据库表的元数据信息。
>
> - HTTP地址：HTTP地址主要用于接收由CloudCanal发来的数据导入请求。

### 创建DataJob

一旦成功添加了数据源，您可以按照以下步骤创建数据迁移与同步DataJob。

- 转至CloudCanal的**DataJob管理** -> **创建DataJob**
- 选择DataJob的源和目标数据库
- 点击下一步

![image.png](../assets/3.11-3.png)

- 选择**增量**并启用**完整数据**
- 选择DDL同步
- 点击下一步

![image.png](../assets/3.11-4.png)

- 选择要订阅的源表。请注意，Schema Migration后自动成为主键表的StarRocks表，因此当前不支持没有主键的源表**

- 点击下一步

![image.png](../assets/3.11-5.png)

- 配置列映射
- 点击下一步

![image.png](../assets/3.11-6.png)

- 创建DataJob

![image.png](../assets/3.11-7.png)

- 检查DataJob的状态。DataJob创建后将自动经历模式迁移、完整数据和增量的阶段

![image.png](../assets/3.11-8.png)