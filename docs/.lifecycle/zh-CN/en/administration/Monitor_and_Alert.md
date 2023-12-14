---
displayed_sidebar: "Chinese"
---

# 监控和警报

您可以构建自己的监控服务，或者使用Prometheus + Grafana解决方案。StarRocks提供了与Prometheus兼容的接口，直接连接到BE和FE的HTTP端口，以从集群中获取监控信息。

## 指标

可用的指标有：

|指标|单位|类型|含义|
|---|:---:|:---:|---|
|be_broker_count|个|平均值|Broker数量|
|be_brpc_endpoint_count|个|平均值| bRPC中StubCache的数量|
|be_bytes_read_per_second|字节/秒|平均值| BE的读取速度|
|be_bytes_written_per_second|字节/秒|平均值|BE的写入速度|
|be_base_compaction_bytes_per_second|字节/秒|平均值|BE的基础压缩速度|
|be_cumulative_compaction_bytes_per_second|字节/秒|平均值|BE的累积压缩速度|
|be_base_compaction_rowsets_per_second|rowsets/秒|平均值|BE行集的基础压缩速度|
|be_cumulative_compaction_rowsets_per_second|rowsets/秒|平均值|BE行集的累积压缩速度|
|be_base_compaction_failed|个/秒|平均值|BE的基础压缩失败|
|be_clone_failed|个/秒|平均值|BE的复制失败|
|be_create_rollup_failed|个/秒|平均值|BE的Materialized View创建失败|
|be_create_tablet_failed|个/秒|平均值|BE的Tablet创建失败|
|be_cumulative_compaction_failed|个/秒|平均值|BE的累积压缩失败|
|be_delete_failed|个/秒|平均值|BE的删除失败|
|be_finish_task_failed|个/秒|平均值|BE的任务失败|
|be_publish_failed|个/秒|平均值|BE的版本发布失败|
|be_report_tables_failed|个/秒|平均值|BE的表报告失败|
|be_report_disk_failed|个/秒|平均值|BE的磁盘报告失败|
|be_report_tablet_failed|个/秒|平均值|BE的Tablet报告失败|
|be_report_task_failed|个/秒|平均值|BE的任务报告失败|
|be_schema_change_failed|个/秒|平均值|BE的Schema变更失败|
|be_base_compaction_requests|个/秒|平均值|BE的基础压缩请求|
|be_clone_total_requests|个/秒|平均值|BE的复制请求|
|be_create_rollup_requests|个/秒|平均值|BE的Materialized View创建请求|
|be_create_tablet_requests|个/秒|平均值|BE的Tablet创建请求|
|be_cumulative_compaction_requests|个/秒|平均值|BE的累积压缩请求|
|be_delete_requests|个/秒|平均值|BE的删除请求|
|be_finish_task_requests|个/秒|平均值|BE的任务完成请求|
|be_publish_requests|个/秒|平均值|BE的版本发布请求|
|be_report_tablets_requests|个/秒|平均值|BE的Tablet报告请求|
|be_report_disk_requests|个/秒|平均值|BE的磁盘报告请求|
|be_report_tablet_requests|个/秒|平均值|BE的Tablet报告请求|
|be_report_task_requests|个/秒|平均值|BE的任务报告请求|
|be_schema_change_requests|个/秒|平均值|BE的Schema变更报告请求|
|be_storage_migrate_requests|个/秒|平均值|BE的迁移请求|
|be_fragment_endpoint_count|个|平均值|BE DataStream的数量|
|be_fragment_request_latency_avg|毫秒|平均值|片段请求的延迟|
|be_fragment_requests_per_second|个/秒|平均值|片段请求的数量|
|be_http_request_latency_avg|毫秒|平均值|HTTP请求的延迟|
|be_http_requests_per_second|个/秒|平均值|HTTP请求的数量|
|be_http_request_send_bytes_per_second|字节/秒|平均值|HTTP请求发送字节数|
|fe_connections_per_second|连接数/秒|平均值|FE的新连接速率|
|fe_connection_total|连接数|累积值|FE的总连接数|
|fe_edit_log_read|操作数/秒|平均值|FE编辑日志的读取速度|
|fe_edit_log_size_bytes|字节/秒|平均值|FE编辑日志的大小|
|fe_edit_log_write|字节/秒|平均值|FE编辑日志的写入速度|
|fe_checkpoint_push_per_second|操作数/秒|平均值|FE的检查点数量|
|fe_pending_hadoop_load_job|个|平均值|待处理的Hadoop作业数量|
|fe_committed_hadoop_load_job|个|平均值|已提交的Hadoop作业数量|
|fe_loading_hadoop_load_job|个|平均值|正在加载的Hadoop作业数量|
|fe_finished_hadoop_load_job|个|平均值|已完成的Hadoop作业数量|
|fe_cancelled_hadoop_load_job|个|平均值|已取消的Hadoop作业数量|
|fe_pending_insert_load_job|个|平均值|待处理的插入作业数量|
|fe_loading_insert_load_job|个|平均值|正在加载的插入作业数量|
|fe_committed_insert_load_job|个|平均值|已提交的插入作业数量|
|fe_finished_insert_load_job|个|平均值|已完成的插入作业数量|
|fe_cancelled_insert_load_job|个|平均值|已取消的插入作业数量|
|fe_pending_broker_load_job|个|平均值|待处理的Broker作业数量|
|fe_loading_broker_load_job|个|平均值|正在加载的Broker作业数量|
|fe_committed_broker_load_job|个|平均值|已提交的Broker作业数量|
|fe_finished_broker_load_job|个|平均值|已完成的Broker作业数量|
|fe_cancelled_broker_load_job|个|平均值|已取消的Broker作业数量|
|fe_pending_delete_load_job|个|平均值|待处理的删除作业数量|
|fe_loading_delete_load_job|个|平均值|正在加载的删除作业数量|
|fe_committed_delete_load_job|个|平均值|已提交的删除作业数量|
|fe_finished_delete_load_job|个|平均值|已完成的删除作业数量|
|fe_cancelled_delete_load_job|个|平均值|已取消的删除作业数量|
|fe_rollup_running_alter_job|个|平均值|Rollup中创建的作业数量|
|fe_schema_change_running_job|个|平均值|Schema变更中的作业数量|
|cpu_util| 百分比|平均值|CPU使用率|
|cpu_system| 百分比|平均值|cpu_system使用率|
|cpu_user| 百分比|平均值|cpu_user使用率|
|cpu_idle| 百分比|平均值|cpu_idle使用率|
|cpu_guest| 百分比|平均值|cpu_guest使用率|
|cpu_iowait| 百分比|平均值|cpu_iowait使用率|
|cpu_irq| 百分比|平均值|cpu_irq使用率|
|cpu_nice| 百分比|平均值|cpu_nice使用率|
|cpu_softirq| 百分比|平均值|cpu_softirq使用率|
|cpu_steal| 百分比|平均值|cpu_steal使用率|
|disk_free|字节|平均值|磁盘可用容量|
|disk_io_svctm|毫秒|平均值|磁盘IO服务时间|
|disk_io_util|百分比|平均值|磁盘使用率|
|disk_used|字节|平均值|磁盘已用容量|
|starrocks_fe_meta_log_count|个|瞬时值|未进行检查点的Edit Logs数量，值在`100000`内被视为合理。|
|starrocks_fe_query_resource_group|个|累积值|每个资源组的查询数量|
|starrocks_fe_query_resource_group_latency|秒|平均值|每个资源组的查询延迟百分比|
|starrocks_fe_query_resource_group_err|个|累积值|每个资源组的错误查询数量|
|starrocks_be_resource_group_cpu_limit_ratio|百分比|瞬时值|资源组CPU配额比的瞬时值|
|starrocks_be_resource_group_cpu_use_ratio|百分比|平均值|资源组CPU时间使用比率与所有资源组CPU时间的比率|
|starrocks_be_resource_group_mem_limit_bytes|字节|瞬时值|资源组内存配额的瞬时值|
|starrocks_be_resource_group_mem_allocated_bytes|字节|瞬时值|资源组内存使用的瞬时值|
|starrocks_be_pipe_prepare_pool_queue_len|个|瞬时值|管道准备线程池任务队列长度的瞬时值|

## 监控告警最佳实践

关于监控系统的背景信息：

1. 该系统每15秒收集一次信息。
2. 一些指标按15秒为间隔，单位为每秒的数量。一些指标不按15秒为间隔，但数量仍然为15秒。
3. P90、P99和其他分位数值目前是在15秒内计算的。在更大的粒度（1分钟、5分钟等）进行计算时，应使用“大于某个值的告警数量”而不是“平均值是多少”。

### 参考

1. 监控的目的是只在异常条件下发出警报，而不是在正常条件下发出。
2. 不同的集群具有不同的资源（例如内存、磁盘）、不同的使用情况，并且需要设置为不同的值；但是，“百分比”作为测量单位是普遍适用的。
3. 对于诸如`失败数量`之类的指标，需要监控总数的变化，并根据一定比例（例如P90、P99、P999的数值）计算警报边界值。
4. `2倍或更高的值`或`高于峰值的值`通常可以用作使用/查询增长的警示值。

### 报警设置

#### 低频报警

如果发生一个或多个故障，则触发报警。如果有多个故障，则设置更高级别的报警。

对于不经常执行的操作（例如模式更改），“故障报警”就足够了。

#### 未启动的任务

一旦监控报警开启，可能会有很多成功和失败的任务。您可以设置`failed > 1`进行警报，并在以后进行修改。

#### 波动

##### 大波动

需要关注具有不同时间粒度的数据，因为具有大粒度的数据中的峰值和谷值可能被平均掉。通常情况下，您需要查看15天，3天，12小时，3小时和1小时（适用于不同的时间范围）。

监视间隔可能需要稍微长一些（例如3分钟，5分钟，甚至更长）以避免由波动引起的报警。

##### 小波动

设置更短的间隔以便在出现问题时快速获得报警。

##### 高峰值

这取决于是否需要对高峰值进行报警。如果有太多的高峰值，则设置更长的间隔可能有助于平滑处理高峰值。

#### 资源使用

##### 高资源使用

您可以设置报警以保留一些资源。例如，将内存警报设置为`mem_avaliable<=20%`。

##### 低资源使用

您可以设置比“高资源使用”更严格的值。例如，对于低使用率的CPU（低于20%），将报警设置为`cpu_idle<60%`。

### 注意

通常情况下，FE/BE一起进行监控，但有些数值只有FE或BE有。

有一些需要批量设置进行监控的机器。

### 附加信息

#### P99 批量计算规则

节点每15秒收集一次数据并计算一个值，99百分位数是在这15秒内的99百分点。当QPS不高时（例如QPS低于10），这些百分位数并不是非常精确。此外，在一分钟内生成的四个值（4 x 15秒），无论使用求和还是平均函数，都是毫无意义的。

对 P50、P90 等也是一样。

#### 集群监控错误

> 一些不期望的集群错误需要及时发现和解决以保持集群的稳定。如果错误不太关键（例如SQL语法错误等），但**不能从重要的错误项目中剔除出去**，建议先进行监控，然后在较后的阶段对其进行区分。

## 使用 Prometheus+Grafana

StarRocks 可以使用 [Prometheus](https://prometheus.io/) 来监控数据存储，使用 [Grafana](https://grafana.com/) 来可视化结果。

### 组件

> 本文档描述了基于 Prometheus 和 Grafana 实现的 StarRocks 可视化监控解决方案。StarRocks 不负责维护或开发这些组件。有关 Prometheus 和 Grafana 的更详细信息，请参阅它们的官方网站。

#### Prometheus

Prometheus 是一个具有多维数据模型和灵活查询语句的时间数据库。它通过从监控系统中拉取或推送数据来收集数据，并将这些数据存储在其时间数据库中。通过其丰富的多维数据查询语言，它满足了不同用户的需求。

#### Grafana

Grafana 是一个开源的度量分析和可视化系统，支持多种数据源。Grafana 通过相应的查询语句从数据源中检索数据。它允许用户创建图表和仪表板以可视化数据。

### 监控架构

![8.10.2-1](../assets/8.10.2-1.png)

Prometheus 从 FE/BE 接口获取指标，然后将数据存储到其时间数据库中。

在 Grafana 中，用户可以配置 Prometheus 作为数据源来自定义仪表板。

### 部署

#### Prometheus

**1.** 从 [Prometheus 官方网站](https://prometheus.io/download/) 下载最新版本的 Prometheus。以 prometheus-2.29.1.linux-amd64 版本为例。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** 在 `vi prometheus.yml` 中添加配置

```yml
# 全局配置
global:
  scrape_interval: 15s # 全局采集间隔，默认为1m，这里设置为15s
  evaluation_interval: 15s # 全局规则触发间隔，默认为1m，这里设置为15s

scrape_configs:
  # 作业名称将作为标签`job=<job_name>`添加到从该配置中获取的所有时间序列。
  - job_name: 'StarRocks_Cluster01' # 每个集群称为一个作业，作业名称可自定义
    metrics_path: '/metrics' # 指定用于获取指标的 Restful API

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe # 这里配置了包含3个前端的 FE 组

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # 在这里配置了包含三个后端的 BE 组
  - job_name: 'StarRocks_Cluster02' # Prometheus 可以监控多个 StarRocks 集群
    metrics_path: '/metrics'

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be
```

**3.** 启动 Prometheus

```bash
nohup ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --log.level="info" &
```

此命令在后台运行 Prometheus，并在端口 9090 上指定其 Web 端口。设置完成后，Prometheus 开始收集数据并将其存储在`. /data` 目录中。

**4.** 访问 Prometheus

可以通过 BUI 访问 Prometheus。只需在浏览器中打开端口 9090 即可。转到 `Status -> Targets` 可以查看所有分组作业的监控主机节点。在正常情况下，所有节点应为 `UP`。如果节点状态不是 `UP`，可以首先访问 StarRocks 指标（`http://fe_host:fe_http_port/metrics` 或 `http://be_host:be_http_port/metrics`）接口来检查是否可以访问，或者查阅 Prometheus 文档进行故障排除。

![8.10.2-6](../assets/8.10.2-6.png)

一个简单的 Prometheus 已建立并配置完毕。要了解更高级的用法，请参考[官方文档](https://prometheus.io/docs/introduction/overview/)

#### Grafana

**1.** 从 [Grafana 官方网站](https://grafana.com/grafana/download) 下载最新版本的 Grafana。以 grafana-8.0.6.linux-amd64 版本为例。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** 在 `vi . /conf/defaults.ini` 中添加配置

```ini
...
[paths]
data = ./data
logs = ./data/log
plugins = ./data/plugins
[server]
http_port = 8000
domain = localhost
...
```

**3.** 启动 Grafana

```Plain text
nohup ./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

### 仪表板

#### 仪表板配置

通过在之前的步骤中配置的地址 `http://grafana_host:8000` 以默认用户名，密码（即 admin,admin）登录 Grafana。

**1.** 添加数据源。

配置路径：`Configuration-->Data sources-->Add data source-->Prometheus`

数据源配置介绍

![8.10.2-2](../assets/8.10.2-2.png)

* Name：数据源名称。可以自定义，例如 starrocks_monitor
* URL：Prometheus 的网址，例如 `http://prometheus_host:9090`
* Access：选择服务器方法，即 Prometheus 访问所在的服务器的方法
其余选项为默认值。

在底部点击保存并测试，如果显示 `Data source is working`，表示数据源可用。

**2.** 添加仪表板。

下载一个仪表板。

> **注意**
>
> StarRocks v1.19.0 和 v2.4.0 中的指标名称已更改。您必须根据您的 StarRocks 版本下载仪表板模板：
> * [早于v1.19.0版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
> * [v1.19.0到v2.4.0（不含）之间的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
> * [v2.4.0及更高版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24.json)

仪表板模板将不时地进行更新。

在确认数据源可用后，点击“+”号以添加新的仪表板，这里我们使用上面下载的StarRocks仪表板模板。转到导入 -> 上传Json文件以加载下载的json文件。

加载完成后，您可以为仪表板命名，默认名称为StarRocks概述。然后选择`starrocks_monitor`作为数据源。
点击`导入`完成导入。然后您应该看到仪表板。

#### 仪表板描述

为您的仪表板添加描述。更新每个版本的描述。

**1.** 顶部工具栏

![8.10.2-3](../assets/8.10.2-3.png)

左上角显示仪表板名称。
右上角显示当前时间范围。使用下拉菜单选择不同的时间范围并指定页面刷新的间隔。
cluster_name：Prometheus配置文件中每个作业的`job_name`，表示一个StarRocks集群。您可以选择一个集群并在图表中查看其监控信息。

* fe_master：集群的主节点。
* fe_instance：相应集群的所有前端节点。选择以查看图表中的监控信息。
* be_instance：相应集群的所有后端节点。选择以查看图表中的监控信息。
* interval：某些图表显示与监控项相关的间隔。间隔是可定制的（注意：15秒的间隔可能导致一些图表无法显示）。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

在Grafana中，`行`的概念是图的集合。您可以通过点击来折叠`行`。当前仪表板具有以下`行`：

* 概述：显示所有StarRocks集群。
* 集群概述：显示所选集群。
* 查询统计：所选集群的查询监控。
* 作业：导入作业的监控。
* 事务：事务的监控。
* FE JVM：所选前端的JVM监控。
* BE：所选集群的后端显示。
* BE任务：所选集群的后端任务显示。

**3.** 典型的图表分为以下部分。

![8.10.2-5](../assets/8.10.2-5.png)

* 将鼠标悬停在左上角的i图标上以查看图表描述。
* 点击下方的图例以查看特定项。再次点击以全部显示。
* 在图表中拖放以选择时间范围。
* 所选集群的名称显示在标题的[]中。
* 值可能对应于左Y轴或右Y轴，可以通过图例末尾的-right来区分。
* 点击图表名称以编辑名称。

### 其他

如果需要访问您自己的Prometheus系统中的监控数据，通过以下接口进行访问。

* FE：fe_host:fe_http_port/metrics
* BE：be_host:be_web_server_port/metrics

如果需要JSON格式，改为以下访问。

* FE：fe_host:fe_http_port/metrics?type=json
* BE：be_host:be_web_server_port/metrics?type=json