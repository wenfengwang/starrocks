---
displayed_sidebar: English
---

# 监控和报警

您可以构建自己的监控服务，也可以使用 Prometheus + Grafana 解决方案。StarRocks 提供了兼容 Prometheus 的接口，直接链接到 BE 和 FE 的 HTTP 端口，用于从集群获取监控信息。

## 指标

可用指标包括：

|指标|单位|类型|含义|
|---|:---:|:---:|---|
|be_broker_count|个|平均|经纪人数量 |
|be_brpc_endpoint_count|个|平均|bRPC 中的 StubCache 数量|
|be_bytes_read_per_second|字节/秒|平均| BE的读取速度 |
|be_bytes_written_per_second|字节/秒|平均|BE的写入速度 |
|be_base_compaction_bytes_per_second|字节/秒|平均|BE基础压实速度|
|be_cumulative_compaction_bytes_per_second|字节/秒|平均|BE累计压实速度|
|be_base_compaction_rowsets_per_second|行集/秒|平均| BE 行集的基本压缩速度|
|be_cumulative_compaction_rowsets_per_second|行集/秒|平均| BE 行集的累积压缩速度 |
|be_base_compaction_failed|次/秒|平均|BE的基层压实失败 |
|be_clone_failed| 次/秒 |平均|BE 克隆失败 |
|be_create_rollup_failed| 次/秒 |平均|BE 的物化视图创建失败 |
|be_create_tablet_failed| 次/秒 |平均|BE 的平板电脑创建失败 |
|be_cumulative_compaction_failed| 次/秒 |平均|BE累积压实失效 |
|be_delete_failed| 次/秒 |平均| BE删除失败 |
|be_finish_task_failed| 次/秒 |平均|BE任务失败 |
|be_publish_failed| 次/秒 |平均| BE版本发布失败 |
|be_report_tables_failed| 次/秒 |平均| 表报告 BE 失败 |
|be_report_disk_failed| 次/秒 |平均|BE 磁盘报告失败 |
|be_report_tablet_failed| 次/秒 |平均|平板电脑报告 BE 失败 |
|be_report_task_failed| 次/秒 |平均|BE任务报告失败 |
|be_schema_change_failed| 次/秒 |平均|BE 的架构更改失败 |
|be_base_compaction_requests| 次/秒 |平均|BE 的基础压实请求 |
|be_clone_total_requests| 次/秒 |平均|BE 的克隆请求 |
|be_create_rollup_requests| 次/秒 |平均| BE的物化视图创建请求 |
|be_create_tablet_requests|次/秒|平均| BE的平板电脑创建请求 |
|be_cumulative_compaction_requests|次/秒|平均|BE 的累积压实请求 |
|be_delete_requests|次/秒|平均| BE的删除请求 |
|be_finish_task_requests|次/秒|平均| BE的任务完成请求 |
|be_publish_requests|次/秒|平均| BE版本发布请求 |
|be_report_tablets_requests|次/秒|平均|BE的平板电脑报告请求 |
|be_report_disk_requests|次/秒|平均|BE的磁盘报告请求 |
|be_report_tablet_requests|次/秒|平均|BE的平板电脑报告请求 |
|be_report_task_requests|次/秒|平均|BE的任务报告请求 |
|be_schema_change_requests|次/秒|平均|BE的架构变更报告请求 |
|be_storage_migrate_requests|次/秒|平均| BE的迁移请求 |
|be_fragment_endpoint_count|个|平均|BE DataStream 数量 |
|be_fragment_request_latency_avg|毫秒|平均| 片段请求的延迟 |
|be_fragment_requests_per_second|次/秒|平均|片段请求数|
|be_http_request_latency_avg|毫秒|平均|HTTP 请求的延迟 |
|be_http_requests_per_second|次/秒|平均|HTTP 请求数|
|be_http_request_send_bytes_per_second|字节/秒|平均| 为 HTTP 请求发送的字节数 |
|fe_connections_per_second|连接数/秒|平均| FE新连接率 |
|fe_connection_total|连接| 累积 | FE 连接总数 |
|fe_edit_log_read|操作/秒|平均|FE 编辑日志读取速度 |
|fe_edit_log_size_bytes|字节/秒|平均|FE 编辑日志的大小 |
|fe_edit_log_write|字节/秒|平均|FE 编辑日志的写入速度 |
|fe_checkpoint_push_per_second|操作/秒|平均|FE 检查点数 |
|fe_pending_hadoop_load_job|个|平均| 挂起的 hadoop 作业数|
|fe_committed_hadoop_load_job|个|平均| 已提交的 hadoop 作业数|
|fe_loading_hadoop_load_job|个|平均| 加载 hadoop 作业数|
|fe_finished_hadoop_load_job|个|平均| 已完成的 hadoop 作业数|
|fe_cancelled_hadoop_load_job|个|平均| 已取消的 hadoop 作业数|
|fe_pending_insert_load_job|个|平均| 待处理的插入作业数 |
|fe_loading_insert_load_job|个|平均| 装载刀片作业数量|
|fe_committed_insert_load_job|个|平均| 已提交的插入作业数|
|fe_finished_insert_load_job|个|平均| 已完成的插入作业数|
|fe_cancelled_insert_load_job|个|平均| 已取消的刀片作业数|
|fe_pending_broker_load_job|个|平均| 待处理的代理作业数|
|fe_loading_broker_load_job|个|平均| 加载代理作业数
|fe_committed_broker_load_job|个|平均| 已提交的代理作业数|
|fe_finished_broker_load_job|个|平均| 已完成的代理作业数|
|fe_cancelled_broker_load_job|个|平均| 已取消的代理作业数 |
|fe_pending_delete_load_job|个|平均| 挂起的删除作业数|
|fe_loading_delete_load_job|个|平均| 加载删除作业数|
|fe_committed_delete_load_job|个|平均| 已提交的删除作业数|
|fe_finished_delete_load_job|个|平均| 已完成的删除作业数|
|fe_cancelled_delete_load_job|个|平均| 已取消的删除作业数|
|fe_rollup_running_alter_job|个|平均| 汇总中创建的作业数 |
|fe_schema_change_running_job|个|平均| 架构更改中的作业数 |
|cpu_util| 百分比|平均|CPU 使用率 |
|cpu_system | 百分比|平均|cpu_system使用率 |
|cpu_user| 百分比|平均|cpu_user使用率 |
|cpu_idle| 百分比|平均|cpu_idle使用率 |
|cpu_guest| 百分比|平均|cpu_guest使用率 |
|cpu_iowait| 百分比|平均|cpu_iowait使用率 |
|cpu_irq| 百分比|平均|cpu_irq使用率 |
|cpu_nice| 百分比|平均|cpu_nice使用率 |
|cpu_softirq| 百分比|平均|cpu_softirq使用率 |
|cpu_steal| 百分比|平均|cpu_steal使用率 |
|disk_free|字节|平均| 可用磁盘容量 |
|disk_io_svctm|毫秒|平均| 磁盘 IO 服务时间 |
|disk_io_util|百分比|平均| 磁盘使用情况 |
|disk_used|字节|平均| 已用磁盘容量 |
|starrocks_fe_meta_log_count|个|瞬时|没有检查点的编辑日志数。内的值 `100000` 被认为是合理的。|
|starrocks_fe_query_resource_group|个|累积|每个资源组的查询数|
|starrocks_fe_query_resource_group_latency|秒|平均|每个资源组的查询延迟百分位数|
|starrocks_fe_query_resource_group_err|个|累积|每个资源组的错误查询数|
|starrocks_be_resource_group_cpu_limit_ratio|百分比|瞬时|资源组 cpu 配额比率的瞬时值|
|starrocks_be_resource_group_cpu_use_ratio|百分比|平均|资源组使用的 CPU 时间与所有资源组的 CPU 时间之比|
|starrocks_be_resource_group_mem_limit_bytes|字节|瞬时|资源组内存配额的瞬时值|
|starrocks_be_resource_group_mem_allocated_bytes|字节|瞬时|资源组内存使用量的瞬时值|
|starrocks_be_pipe_prepare_pool_queue_len|个|瞬时|流水线准备线程池任务队列长度的瞬时值|

## 监控告警最佳实践

监控系统的背景信息如下：

1. 系统每 15 秒收集一次信息。
2. 有些指标除以 15 秒，单位为 count/s。有些指标没有划分，计数仍然是 15 秒。
3. P90、P99 和其他分位数值目前在 15 秒内计数。当以更大的粒度（1 分钟、5 分钟等）计算时，请使用“大于某个值的警报数”而不是“平均值是多少”。

### 引用

1. 监视的目的是仅在异常情况下发出警报，而不是在正常情况下发出警报。
2. 不同的集群有不同的资源（如内存、磁盘），不同的使用量，需要设置不同的值;但是，“百分比”作为测量单位是通用的。
3. 对于诸如 `number of failures` 的指标，需要监测总数的变化，并按照一定比例计算报警边界值（例如，对于P90、P99、P999的量）。
4. `A value of 2x or more` 或者 `a value higher than the peak` 通常可以用作 used/query 增长的警告值。

### 告警设置

#### 低频率告警

如果发生一个或多个故障，则触发警报。如果出现多个故障，请设置更高级的警报。

对于不经常执行的操作（例如，模式更改），“故障警报”就足够了。

#### 未启动任务

一旦开启监控告警，可能会有很多成功和失败的任务。您可以设置为 `failed > 1` 警报并在以后修改它。

#### 波动

##### 波动大

需要关注不同时间粒度的数据，因为大粒度数据中的峰谷可能会被平均化。一般来说，您需要查看 15 天、3 天、12 小时、3 小时和 1 小时（针对不同的时间范围）。

监测间隔可能需要稍长一些（例如 3 分钟、5 分钟甚至更长），以屏蔽波动引起的警报。

##### 波动小

设置较短的间隔，以便在出现问题时快速发出警报。

##### 高尖峰


这取决于是否需要对尖峰进行警报。如果尖峰太多，设置更长的间隔可能有助于平滑处理尖峰。

#### 资源使用

##### 资源使用高

您可以设置警报以保留一些资源。例如，将内存警报设置为 `mem_avaliable<=20%`。

##### 资源使用低

您可以设置比“资源使用高”更严格的值。例如，对于使用率较低的 CPU（低于20%），将警报设置为 `cpu_idle<60%`。

### 注意事项

通常前端/后端一起进行监控，但有些值只有前端或后端才有。

可能有一些机器需要批量设置以进行监控。

### 附加信息

#### P99 批量计算规则

节点每15秒收集一次数据并计算一个值，99百分位数是这15秒内的第99百分位数。当QPS不高时（例如QPS低于10），这些百分位数不太准确。此外，无论使用求和函数还是平均值函数，聚合在一分钟（4 x 15秒）内生成的四个值都是没有意义的。

这同样适用于P50、P90等。

#### 集群监视错误

> 需要及时发现并解决一些不需要的集群错误，以保持集群稳定。如果错误不太严重（例如SQL语法错误等），但**无法从重要的错误项中剥离**出来，建议先监视并在稍后阶段区分这些错误。

## 使用 Prometheus+Grafana

StarRocks可以使用[Prometheus](https://prometheus.io/)来监控数据存储，并使用[Grafana](https://grafana.com/)来可视化结果。

### 组件

>本文介绍了StarRocks基于Prometheus和Grafana实现的可视化监控解决方案。StarRocks不负责维护或开发这些组件。有关Prometheus和Grafana的更多详细信息，请参阅它们的官方网站。

#### Prometheus

Prometheus是一个具有多维数据模型和灵活查询语句的时态数据库。它通过从受监控系统中提取或推送数据来收集数据，并将这些数据存储在其时态数据库中。它通过其丰富的多维度数据查询语言满足不同的用户需求。

#### Grafana

Grafana是一个开源的指标分析和可视化系统，支持多种数据源。Grafana使用相应的查询语句从数据源中检索数据。它允许用户创建图表和仪表板来可视化数据。

### 监视架构

![8.10.2-1](../assets/8.10.2-1.png)

Prometheus从前端/后端接口拉取指标，然后将数据存储到其时态数据库中。

在Grafana中，用户可以配置Prometheus作为数据源来自定义仪表板。

### 部署

#### Prometheus

**1.** 从[Prometheus官方网站](https://prometheus.io/download/)下载最新版本的Prometheus。以prometheus-2.29.1.linux-amd64版本为例。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** 在`vi prometheus.yml`中添加配置

```yml
# 我的全局配置
global:
  scrape_interval: 15s # 默认为1分钟的全局采集间隔，这里设置为15秒
  evaluation_interval: 15s # 默认为1分钟的全局规则触发间隔，这里设置为15秒

scrape_configs:
  # 作业名称被添加为标签`job=<job_name>`到从此配置中抓取的任何时间序列。
  - job_name: 'StarRocks_Cluster01' # 每个集群称为一个作业，作业名称可自定义
    metrics_path: '/metrics' # 指定用于获取指标的Restful API

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe # 这里配置了包含3个前端的FE组

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # 这里配置了包含三个后端的BE组
  - job_name: 'StarRocks_Cluster02' # Prometheus可以监控多个StarRocks集群
    metrics_path: '/metrics'

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be
```

**3.** 启动Prometheus

```bash
nohup ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --log.level="info" &
```

此命令在后台运行Prometheus，并将其Web端口指定为9090。设置完成后，Prometheus开始收集数据并将其存储在`. /data`目录中。

**4.** 访问Prometheus

可以通过BUI访问Prometheus。只需在浏览器中打开端口9090。转到`Status -> Targets`查看所有分组作业的受监控主机节点。在正常情况下，所有节点都应该是`UP`。如果节点状态不是`UP`，可以先访问StarRocks指标（`http://fe_host:fe_http_port/metrics`或`http://be_host:be_http_port/metrics`）界面检查是否可访问，或者查看Prometheus文档进行故障排除。

![8.10.2-6](../assets/8.10.2-6.png)

一个简单的Prometheus已经构建和配置。有关更高级的用法，请参考[官方文档](https://prometheus.io/docs/introduction/overview/)

#### Grafana

**1.** 从[Grafana官方网站](https://grafana.com/grafana/download)下载最新版本的Grafana。以grafana-8.0.6.linux-amd64版本为例。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** 在`vi . /conf/defaults.ini`中添加配置

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

**3.** 启动Grafana

```Plain text
nohup ./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

### 仪表板

#### 仪表板配置

通过在上一步中配置的地址`http://grafana_host:8000`使用默认用户名、密码（即admin，admin）登录Grafana。

**1.** 添加数据源。

配置路径：`Configuration-->Data sources-->Add data source-->Prometheus`

数据源配置介绍

![8.10.2-2](../assets/8.10.2-2.png)

* 名称：数据源的名称。可以定制，例如starrocks_monitor
* URL：Prometheus的网址，例如`http://prometheus_host:9090`
* 访问：选择服务器方式，即Grafana所在的服务器供Prometheus访问。
其余选项为默认值。

单击底部的Save & Test，如果显示`Data source is working`，则表示数据源可用。

**2.** 添加仪表板。

下载一个仪表板。

> **注意**
>
> StarRocks v1.19.0和v2.4.0中的指标名称已更改。您必须根据StarRocks版本下载一个仪表板模板：
>
> * [v1.19.0之前版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
> * [v1.19.0到v2.4.0（不含）的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
> * [v2.4.0及以后版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24-new.json)

仪表板模板将不时更新。

确认数据源可用后，单击`+`号添加新的仪表板，这里我们使用上面下载的StarRocks仪表板模板。转到导入 -> 上传Json文件以加载下载的json文件。

加载后，您可以命名仪表板。默认名称为StarRocks Overview。然后选择`starrocks_monitor`作为数据源。
单击`Import`完成导入。然后，您应该会看到仪表板。

#### 仪表板说明

为您的仪表板添加说明。更新每个版本的说明。

**1.** 顶部栏

![8.10.2-3](../assets/8.10.2-3.png)

左上角显示仪表板名称。
右上角显示当前时间范围。使用下拉列表选择不同的时间范围并指定页面刷新的间隔。
cluster_name：Prometheus配置文件中每个作业的`job_name`，代表一个StarRocks集群。您可以选择一个集群，并在图表中查看其监控信息。

* fe_master：集群的领导节点。
* fe_instance：所选集群的所有前端节点。选择以查看图表中的监控信息。
* be_instance：所选集群的所有后端节点。选择以查看图表中的监控信息。
* interval：部分图表显示与监控项相关的间隔。间隔是可自定义的（注意：15秒的间隔可能会导致某些图表不显示）。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

在Grafana中，`Row`的概念是图表的集合。单击`Row`可折叠它。当前仪表板具有以下`Rows`：

* 概览：显示所有StarRocks集群。
* 集群概览：显示所选集群。
* 查询统计：监控所选集群的查询。
* 作业：监视导入作业。
* 事务：监视事务。
* FE JVM：监视所选前端的JVM。
* BE：显示所选集群的后端。
* BE Task：显示所选集群的后端任务。

**3.** 典型图表分为以下几个部分。

![8.10.2-5](../assets/8.10.2-5.png)

* 将鼠标悬停在左上角的i图标上可查看图表说明。
* 单击下面的图例可查看特定项目。再次单击以显示全部。
* 在图表中拖放以选择时间范围。
* 所选集群的名称将显示在标题的[]中。
* 值可能对应于左Y轴或右Y轴，可以通过图例末尾的-right来区分。
* 单击图表名称以编辑名称。

### 其他

如果您需要访问自己的 Prometheus 系统中的监控数据，请通过以下接口进行访问。

* FE：fe_host:fe_http_port/metrics
* BE：be_host:be_web_server_port/metrics

如果需要 JSON 格式，请改为访问以下内容。

* FE：fe_host:fe_http_port/metrics?type=json
* BE：be_host:be_web_server_port/metrics?type=json