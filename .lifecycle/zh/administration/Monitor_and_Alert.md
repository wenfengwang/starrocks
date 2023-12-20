---
displayed_sidebar: English
---

# 监控和报警

您可以构建自己的监控服务，或者采用Prometheus + Grafana方案。StarRocks提供了与Prometheus兼容的接口，可以直接连接到BE和FE的HTTP端口，以获取集群的监控信息。

## 指标

可用的指标包括：

|公制|单位|类型|含义|
|---|---|---|---|
|be_broker_count|计数|平均|经纪商数量|
|be_brpc_endpoint_count|count|average|bRPC 中 StubCache 的数量|
|be_bytes_read_per_second|字节/秒|平均|BE读取速度|
|be_bytes_writing_per_second|字节/秒|平均|BE 写入速度|
|be_base_compaction_bytes_per_second|bytes/s|average|BE 的基本压缩速度|
|be_cumulative_compaction_bytes_per_second|bytes/s|average|BE的累积压缩速度|
|be_base_compaction_rowsets_per_second|行集/秒|平均|BE 行集的基本压缩速度|
|be_cumulative_compaction_rowsets_per_second|rowsets/s|average|BE 行集的累积压缩速度|
|be_base_compaction_failed|count/s|average|BE 的基础压缩失败|
|be_clone_failed|计数/秒|平均值|BE 克隆失败|
|be_create_rollup_failed|count/s|average|BE 物化视图创建失败|
|be_create_tablet_failed|count/s|average|BE|tablet创建失败|
|be_cumulative_compaction_failed|count/s|average|BE的累积压缩失败次数|
|be_delete_failed|count/s|average|BE删除失败|
|be_finish_task_failed|count/s|average|BE 任务失败|
|be_publish_failed|count/s|average|BE版本发布失败|
|be_report_tables_failed|count/s|average|BE|表报告失败|
|be_report_disk_failed|count/s|average|BE磁盘报告失败|
|be_report_tablet_failed|count/s|average|平板上报BE失败|
|be_report_task_failed|count/s|average|BE任务上报失败|
|be_schema_change_failed|count/s|average|BE 架构更改失败|
|be_base_compaction_requests|count/s|average|BE 的基本压缩请求|
|be_clone_total_requests|count/s|average|BE 的克隆请求|
|be_create_rollup_requests|count/s|average|BE的物化视图创建请求|
|be_create_tablet_requests|count/s|average|BE 的 Tablet 创建请求|
|be_cumulative_compaction_requests|count/s|average|BE的累积压缩请求|
|be_delete_requests|count/s|average|BE删除请求|
|be_finish_task_requests|count/s|average|BE 的任务完成请求|
|be_publish_requests|count/s|average|BE版本发布请求|
|be_report_tablets_requests|count/s|average|BE 平板电脑报告请求|
|be_report_disk_requests|count/s|average|BE磁盘报告请求|
|be_report_tablet_requests|count/s|average|BE 的平板电脑报告请求|
|be_report_task_requests|count/s|average|BE任务报告请求|
|be_schema_change_requests|count/s|average|BE 的架构更改报告请求|
|be_storage_migrate_requests|count/s|average|BE 迁移请求数|
|be_fragment_endpoint_count|count|average|BE 数据流数量|
|be_fragment_request_latency_avg|ms|average|片段请求的延迟|
|be_fragment_requests_per_second|count/s|average|片段请求数|
|be_http_request_latency_avg|ms|平均|HTTP 请求的延迟|
|be_http_requests_per_second|count/s|平均值|HTTP 请求数|
|be_http_request_send_bytes_per_second|字节/秒|平均值|HTTP 请求发送的字节数|
|fe_connections_per_second|connections/s|average|FE新建连接率|
|fe_connection_total|连接|累积|FE 连接总数|
|fe_edit_log_read|操作/秒|平均|FE编辑日志读取速度|
|fe_edit_log_size_bytes|字节/秒|平均值|FE 编辑日志的大小|
|fe_edit_log_write|bytes/s|average|FE编辑日志写入速度|
|fe_checkpoint_push_per_second|操作/秒|平均值|FE 检查点数量|
|fe_pending_hadoop_load_job|count|average|挂起的 hadoop 作业数量|
|fe_commited_hadoop_load_job|count|average|已提交的 hadoop 作业数量|
|fe_loading_hadoop_load_job|count|average|加载 hadoop 作业的数量|
|fe_finished_hadoop_load_job|count|average|已完成的 hadoop 作业数量|
|fe_cancelled_hadoop_load_job|count|average|已取消的 hadoop 作业数量|
|fe_pending_insert_load_job|count|average|挂起的插入作业数量|
|fe_loading_insert_load_job|count|average|加载插入作业数量|
|fe_commissed_insert_load_job|count|average|已提交的插入作业数|
|fe_finished_insert_load_job|count|average|已完成的插入作业数量|
|fe_cancelled_insert_load_job|count|average|取消的插入作业数量|
|fe_pending_broker_load_job|count|average|待处理的代理作业数量|
|fe_loading_broker_load_job|count|average|加载代理作业数量|
|fe_commissed_broker_load_job|count|average|已提交的代理作业数量|
|fe_finished_broker_load_job|count|average|已完成的代理作业数量|
|fe_cancelled_broker_load_job|count|average|取消的代理作业数量|
|fe_pending_delete_load_job|count|average|待删除作业的数量|
|fe_loading_delete_load_job|count|average|加载删除作业数量|
|fe_commissed_delete_load_job|count|average|已提交的删除作业数|
|fe_finished_delete_load_job|count|average|已完成删除作业的数量|
|fe_cancelled_delete_load_job|count|average|取消的删除作业数量|
|fe_rollup_running_alter_job|count|average|汇总中创建的作业数量|
|fe_schema_change_running_job|count|average|架构更改中的作业数量|
|cpu_util|百分比|平均|CPU使用率|
|cpu_system|百分比|平均|cpu_system使用率|
|cpu_user|百分比|平均|cpu_user使用率|
|cpu_idle|百分比|平均|cpu_idle使用率|
|cpu_guest|百分比|平均|cpu_guest使用率|
|cpu_iowait|百分比|平均|cpu_iowait使用率|
|cpu_irq|百分比|平均|cpu_irq使用率|
|cpu_nice|百分比|平均|cpu_nice使用率|
|cpu_softirq|百分比|平均|cpu_softirq使用率|
|cpu_steal|百分比|平均|cpu_steal使用率|
|disk_free|字节|平均|可用磁盘容量|
|disk_io_svctm|ms|平均|磁盘IO服务时间|
|disk_io_util|百分比|平均|磁盘使用情况|
|disk_used|字节|平均值|已用磁盘容量|
|starrocks_fe_meta_log_count|count|瞬时|没有检查点的编辑日志的数量。 100000 以内的值被认为是合理的。|
|starrocks_fe_query_resource_group|count|cumulative|每个资源组的查询数量|
|starrocks_fe_query_resource_group_latency|秒|平均值|每个资源组的查询延迟百分位|
|starrocks_fe_query_resource_group_err|count|cumulative|每个资源组的错误查询数量|
|starrocks_be_resource_group_cpu_limit_ratio|百分比|瞬时|资源组cpu配额比例瞬时值|
|starrocks_be_resource_group_cpu_use_ratio|percentage|average|资源组使用的CPU时间与所有资源组CPU时间的比率|
|starrocks_be_resource_group_mem_limit_bytes|byte|瞬时|资源组内存配额瞬时值|
|starrocks_be_resource_group_mem_allocated_bytes|byte|Instantaneous|资源组内存使用情况的瞬时值|
|starrocks_be_pipe_prepare_pool_queue_len|count|Instantaneous|管道准备线程池任务队列长度的瞬时值|

## 监控报警最佳实践

监控系统背景信息：

1. 系统每15秒收集一次信息。
2. 部分指标除以15秒，单位是次/秒。有些指标未除以15秒，计数依然是15秒。
3. P90、P99等分位数值目前在15秒内计算。当以更大的时间粒度（如1分钟、5分钟等）计算时，应使用“超过某个值的报警数量”而非“平均值是多少”。

### 参考

1. 监控的目的是仅对异常情况发出警报，而非正常情况。
2. 不同的集群有不同的资源（例如，内存、磁盘），不同的使用情况，需要设置不同的阈值；然而，“百分比”作为度量单位是通用的。
3. 对于如失败次数等指标，需要监控总数的变化，并根据一定比例（例如，P90、P99、P999的数量）计算报警边界值。
4. 使用量/查询增长的2倍或更高的值，或高于峰值的值，通常可以作为警告值。

### 报警设置

#### 低频报警

如果出现一个或多个故障，则触发报警。若多次出现故障，应设置更高级别的报警。

对于不频繁执行的操作（例如，模式更改），仅在失败时报警即可。

#### 无任务启动

开启监控报警后，可能会记录大量成功和失败的任务。可以设置失败次数大于1时进行报警，并在后期根据情况调整。

#### 波动性

##### 大幅波动

需要关注在不同时间粒度下的数据，因为大时间粒度的数据中的波峰和波谷可能会被平均化。通常，需要观察15天、3天、12小时、3小时和1小时的数据（针对不同时间范围）。

监控间隔可能需要适当延长（例如，3分钟、5分钟或更长），以避免因数据波动而误报。

##### 小幅波动

设置较短的间隔，以便在问题发生时能迅速得到报警。

##### 尖峰现象

是否需要对尖峰现象进行报警取决于具体情况。如果尖峰现象频繁，设置较长的间隔可能有助于平滑这些尖峰。

#### 资源使用

##### 资源使用率高

可以设置报警阈值以留出一定的资源余量。例如，将内存报警阈值设置为mem_available<=20%。

##### 资源使用率低

可以设置比“资源使用率高”更严格的阈值。例如，对于低使用率（少于20%）的CPU，将报警阈值设置为cpu_idle<60%。

### 注意事项

通常需要同时监控FE/BE，但某些指标仅适用于FE或BE。

可能需要批量设置某些机器的监控。

### 附加信息

#### P99批处理计算规则

节点每15秒收集一次数据并计算一个值，99百分位数是这15秒内的第99个百分位数。当QPS不高（例如，QPS低于10）时，这些百分位数的准确性不高。此外，将一分钟内（4 x 15秒）生成的四个值进行求和或平均都是没有意义的。

P50、P90等其他百分位数也适用同样的规则。

#### 集群错误监控

> 需要及时发现并解决某些不期望的集群错误，以维持集群稳定。如果错误较为轻微（例如，SQL语法错误等），但**不能从重要错误中剔除**，建议先监控，然后在后期进行区分。

## 使用Prometheus+Grafana

StarRocks可以利用[Prometheus](https://prometheus.io/)来监控数据存储，并使用[Grafana](https://grafana.com/)进行结果可视化。

### 组件

> 本文档描述了基于Prometheus和Grafana实现的StarRocks可视化监控方案。StarRocks不负责这些组件的维护或开发。关于Prometheus和Grafana的更多详细信息，请参阅它们的官方网站。

#### Prometheus

Prometheus是一个时序数据库，具有多维数据模型和灵活的查询语言。它通过从被监控系统中拉取或推送数据来收集数据，并将这些数据存储在它的时序数据库中。通过其丰富的多维数据查询语言，满足不同用户的需求。

#### Grafana

Grafana是一个开源的指标分析和可视化系统，支持多种数据源。Grafana通过相应的查询语句从数据源检索数据，并允许用户创建图表和仪表板来可视化数据。

### 监控架构

![8.10.2-1](../assets/8.10.2-1.png)

Prometheus从FE/BE接口拉取指标，然后将数据存储到它的时序数据库中。

在Grafana中，用户可以配置Prometheus作为数据源，自定义Dashboard。

### 部署

#### Prometheus

**1.** 从[Prometheus官方网站](https://prometheus.io/download/)下载最新版本的Prometheus。以prometheus-2.29.1.linux-amd64为例。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** 在`vi prometheus.yml`中添加配置。

```yml
# my global config
global:
  scrape_interval: 15s # global acquisition interval, 1m by default, here set to 15s
  evaluation_interval: 15s # global rule trigger interval, 1m by default, here set to 15s

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'StarRocks_Cluster01' # Each cluster is called a job, job name is customizable
    metrics_path: '/metrics' # Specify the Restful API to get metrics

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe # Here the group of FE is configured which contains 3 Frontends

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # The group of BE is configured here which contains three Backends
  - job_name: 'StarRocks_Cluster02' # Multiple StarRocks clusters can be monitored in Prometheus
metrics_path: '/metrics'

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be
```

**3.** 启动Prometheus。

```bash
nohup ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --log.level="info" &
```

该命令将Prometheus作为后台进程运行，并将其Web端口设置为9090。配置完成后，Prometheus开始收集数据并将其存储在./data目录中。

**4.** 访问Prometheus。

可以通过BUI访问Prometheus。只需在浏览器中打开9090端口。转到“状态 -> 目标”，查看所有分组作业的监控主机节点。在正常情况下，所有节点都应该是UP状态。如果节点状态不是UP，可以先访问StarRocks的metrics接口（http://fe_host:fe_http_port/metrics或http://be_host:be_http_port/metrics）检查是否可访问，或查阅Prometheus文档进行故障排除。

![8.10.2-6](../assets/8.10.2-6.png)

一个简单的**Prometheus**已经搭建并配置完成。更高级的使用方法，请参考[官方文档](https://prometheus.io/docs/introduction/overview/)。

#### Grafana

**1.** 从[Grafana官方网站](https://grafana.com/grafana/download)下载最新版本的Grafana。以grafana-8.0.6.linux-amd64版本为例。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** 在`vi . /conf/defaults.ini`中添加配置。

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

**3.** 启动Grafana。

```Plain
nohup ./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

### Dashboard配置

#### 通过之前步骤配置的地址http://grafana_host:8000登录Grafana，使用默认的用户名和密码（即admin, admin）。

1. 添加数据源。

**1.** 添加一个数据源。

数据源配置简介：

名称：数据源的名称，可以自定义，例如starrocks_monitor。

![8.10.2-2](../assets/8.10.2-2.png)

* URL：Prometheus的Web地址，例如http://prometheus_host:9090。
* 访问方式：选择Server，即Grafana所在服务器让Prometheus进行访问。其余选项保持默认。
* 点击页面底部的“保存并测试”，如果显示“数据源正在工作”，则表示数据源已经可用。

2. 添加Dashboard。

**2.** 添加一个仪表板。

注意：

> **注意**StarRocks v1.19.0和v2.4.0的指标名称有更改。根据您的StarRocks版本下载相应的Dashboard模板：
> StarRocks v1.19.0和v2.4.0中的指标名称已更改。您必须根据您的StarRocks版本下载仪表板模板：v1.19.0之前版本的Dashboard模板。
* [v1.19.0到v2.4.0（不包括）的Dashboard模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)。
* v2.4.0及以后版本的[Dashboard template for v1.19.0 to v2.4.0 (exclusive)](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
* [Dashboard template for v2.4.0 and later](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24-new.json)会不定期更新。

确认数据源可用后，点击加号添加新的Dashboard。这里我们使用上面下载的StarRocks Dashboard模板。转到导入 -> 上传Json文件，载入下载的json文件。

载入后，您可以为Dashboard命名。默认名称为StarRocks概览。然后选择starrocks_monitor作为数据源。点击导入完成导入。之后，您应该能看到Dashboard。

Dashboard描述

#### 为您的Dashboard添加描述。每个版本更新描述。

1. 顶部栏

**1.** Top bar

![8.10.2-3](../assets/8.10.2-3.png)

fe_master：集群的领导节点。

* fe_instance：对应集群的所有前端节点。选择以查看图表中的监控信息。
* be_instance：对应集群的所有后端节点。选择以查看图表中的监控信息。
* 间隔：一些图表显示与监控项相关的间隔。间隔是可自定义的（注意：15秒间隔可能导致某些图表无法显示）。
* 2. 行

**2.** Row，在Grafana中，行是图表的集合。点击行可以折叠它。当前Dashboard包含以下行：

![8.10.2-4](../assets/8.10.2-4.png)

概览：展示所有StarRocks集群的情况。

* 集群概览：展示所选集群的情况。
* 查询统计：监控所选集群的查询情况。
* 作业：监控导入作业。
* 事务：监控事务。
* FE JVM：监控所选前端的JVM。
* BE：展示所选集群的后端情况。
* BE任务：展示所选集群后端任务的情况。
* 3. 典型图表分为以下几部分：

**3.** 一个典型的图表被分成以下几个部分。

![8.10.2-5](../assets/8.10.2-5.png)

* 点击下方图例可以单独查看某个项目。再次点击可以显示所有项目。
* 在图表中拖动选择特定的时间范围。
* 所选集群的名称显示在标题的方括号[]中。
* 数值可能对应左侧或右侧的Y轴，这可以通过图例末尾的-right来区分。
* 点击图表名称可以编辑名称。
* 其他

### 如果您需要在自己的Prometheus系统中访问监控数据，请通过以下接口访问：

FE: fe_host:fe_http_port/metrics

* BE: be_host:be_web_server_port/metrics
* 如果需要JSON格式，请访问以下接口：

FE: fe_host:fe_http_port/metrics?type=json

* BE: be_host:be_web_server_port/metrics?type=json
* BE：be_host:be_web_server_port/metrics?type=json
