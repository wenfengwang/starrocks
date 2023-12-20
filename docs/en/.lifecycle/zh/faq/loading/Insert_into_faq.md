---
displayed_sidebar: English
---

# 插入

## 在执行数据插入时，每次SQL插入需要50到100毫秒。有没有办法可以提高效率？

不建议将数据逐条插入到OLAP中。通常是批量插入。这两种方法所需的时间是相同的。

## 'Insert into select' 任务报告错误：index channel 有不可容忍的失败

您可以通过更改 Stream Load RPC 的超时时长来解决此问题。在 **be.conf** 中更改以下配置项并重启机器以使更改生效：

`streaming_load_rpc_max_alive_time_sec`：Stream Load 的 RPC 超时时间。单位：秒。默认值：`1200`。

或者您可以使用以下变量来设置查询超时：

`query_timeout`：查询的超时时长。单位是秒，默认值是 `300`。

## 当我运行 INSERT INTO SELECT 命令来加载大量数据时，出现了“执行超时”的错误

默认情况下，查询的超时时长是300秒。您可以设置变量 `query_timeout` 来延长这个时长。单位是秒。