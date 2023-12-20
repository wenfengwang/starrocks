---
displayed_sidebar: English
---

您可以设置比“资源使用率高”更严格的值。例如，对于CPU使用率低（低于20%），将警报设置为`cpu_idle<60%`。

### 注意事项

通常，FE/BE是一起监控的，但有些值只有FE或BE有。

可能需要批量设置一些机器进行监控。

### 附加信息

#### P99 批量计算规则

节点每 15 秒收集一次数据并计算一个值，99 分位数是这 15 秒内的 99 分位数。当 QPS 不高时（例如，QPS 低于 10），这些百分位数并不是非常准确的。此外，在一分钟内生成的四个值（4 x 15 秒）无论是使用 sum 还是 average 函数，都是没有意义的。

P50、P90 等也是如此。

#### 集群错误监控

> 一些不希望出现的集群错误需要及时发现和解决，以保持集群的稳定性。如果错误不是很关键（例如 SQL 语法错误等），但是**不能从重要的错误项中剥离出来**，建议先进行监控，稍后再区分。

## 使用 Prometheus+Grafana

StarRocks 可以使用 [Prometheus](https://prometheus.io/) 监控数据存储，并使用 [Grafana](https://grafana.com/) 可视化结果。

### 组件

> 本文档描述了基于 Prometheus 和 Grafana 实现的 StarRocks 可视化监控方案。StarRocks 不负责维护或开发这些组件。有关 Prometheus 和 Grafana 的更详细信息，请参阅它们的官方网站。

#### Prometheus

Prometheus 是一个具有多维数据模型和灵活查询语句的时间数据库。它通过从被监控系统中拉取或推送数据来收集数据，并将这些数据存储在其时间数据库中。通过其丰富的多维数据查询语言，它满足不同用户的需求。

#### Grafana

Grafana 是一个开源的度量分析和可视化系统，支持多种数据源。Grafana 使用相应的查询语句从数据源中检索数据。它允许用户创建图表和仪表板来可视化数据。

### 监控架构

![8.10.2-1](../assets/8.10.2-1.png)

Prometheus 从 FE/BE 接口拉取指标，然后将数据存储到其时间数据库中。

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
# my global config
global:
  scrape_interval: 15s # 全局采集间隔，默认为 1m，这里设置为 15s
  evaluation_interval: 15s # 全局规则触发间隔，默认为 1m，这里设置为 15s

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'StarRocks_Cluster01' # 每个集群称为一个 job，job 名称可自定义
    metrics_path: '/metrics' # 指定获取指标的 Restful API

    static_configs:
      - targets: ['fe_host1:http_port','fe_host3:http_port','fe_host3:http_port']
        labels:
          group: fe # 这里配置了 FE 的分组，包含 3 个前端节点

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # 这里配置了 BE 的分组，包含 3 个后端节点
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

该命令在后台运行 Prometheus，并将其 Web 端口设置为 9090。设置完成后，Prometheus 开始收集数据并将其存储在 `. /data` 目录中。

**4.** 访问 Prometheus

可以通过 BUI 访问 Prometheus。只需在浏览器中打开端口 9090。转到`Status -> Targets`，查看所有分组作业的监控主机节点。在正常情况下，所有节点应为“UP”。如果节点状态不是“UP”，可以先访问 StarRocks 指标（`http://fe_host:fe_http_port/metrics` 或 `http://be_host:be_http_port/metrics`）接口检查其是否可访问，或者查看 Prometheus 文档进行故障排除。

![8.10.2-6](../assets/8.10.2-6.png)

已经建立并配置了一个简单的 Prometheus。有关更高级的用法，请参阅[官方文档](https://prometheus.io/docs/introduction/overview/)。

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

```Plain
nohup ./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

### 仪表板

#### 仪表板配置

通过在上一步配置的地址 `http://grafana_host:8000` 使用默认的用户名和密码（即 admin、admin）登录 Grafana。

**1.** 添加数据源。

配置路径：`Configuration-->Data sources-->Add data source-->Prometheus`

数据源配置介绍

![8.10.2-2](../assets/8.10.2-2.png)

* Name：数据源的名称。可自定义，例如 starrocks_monitor。
* URL：Prometheus 的 Web 地址，例如 `http://prometheus_host:9090`。
* Access：选择 Server 方法，即 Prometheus 访问的 Grafana 所在的服务器。
其余选项为默认。

在底部点击 Save & Test，如果显示 `Data source is working`，表示数据源可用。

**2.** 添加仪表板。

下载一个仪表板。

> **注意**
> StarRocks v1.19.0 和 v2.4.0 的指标名称已更改。您必须根据您的 StarRocks 版本下载仪表板模板：
* [v1.19.0 之前版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
* [v1.19.0 到 v2.4.0（不含）版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
* [v2.4.0 及更高版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24-new.json)

仪表板模板将不时更新。

确认数据源可用后，点击`+`号添加新的仪表板，这里我们使用上面下载的 StarRocks 仪表板模板。转到 Import -> Upload Json File，加载下载的 json 文件。

加载完成后，可以为仪表板命名。默认名称为 StarRocks Overview。然后选择`starrocks_monitor`作为数据源。
点击`Import`完成导入。然后您应该可以看到仪表板。

#### 仪表板描述

为您的仪表板添加描述。根据每个版本更新描述。

**1.** 顶部栏

![8.10.2-3](../assets/8.10.2-3.png)

左上角显示仪表板名称。
右上角显示当前时间范围。使用下拉菜单选择不同的时间范围，并指定页面刷新的间隔。
cluster_name：Prometheus 配置文件中每个作业的 `job_name`，表示一个 StarRocks 集群。您可以选择一个集群，并在图表中查看其监控信息。

* fe_master：集群的 Leader 节点。
* fe_instance：所选集群的所有前端节点。选择以查看图表中的监控信息。
* be_instance：所选集群的所有后端节点。选择以查看图表中的监控信息。
* interval：某些图表显示与监控项相关的间隔。间隔可自定义（注意：15s 的间隔可能导致某些图表不显示）。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

在 Grafana 中，`Row` 的概念是一组图表。您可以通过单击来折叠一个`Row`。当前仪表板有以下`Rows`：

* Overview：显示所有 StarRocks 集群的概览。
* Cluster Overview：显示所选集群的概览。
* Query Statistic：显示所选集群的查询统计信息。
* Jobs：显示导入作业的监控信息。
* Transaction：显示事务的监控信息。
* FE JVM：显示所选前端的 JVM 监控信息。
* BE：显示所选集群的后端节点。
* BE Task：显示所选集群的后端任务。

**3.** 典型图表分为以下几个部分。

![8.10.2-5](../assets/8.10.2-5.png)

* 将鼠标悬停在左上角的 i 图标上，可以查看图表描述。
* 单击标题下方的图例以查看特定项。再次单击以显示所有项。
* 在图表中拖动以选择时间范围。
* 标题的 [] 中显示所选集群的名称。
* 值可能对应于左 Y 轴或右 Y 轴，可以通过图例末尾的 -right 进行区分。
* 单击图表名称以编辑名称。

### 其他

如果需要在自己的 Prometheus 系统中访问监控数据，请通过以下接口访问。

* FE：fe_host:fe_http_port/metrics
* BE：be_host:be_web_server_port/metrics

如果需要 JSON 格式的数据，请访问以下接口。

* FE：fe_host:fe_http_port/metrics?type=json
* BE：be_host:be_web_server_port/metrics?type=json
```
您可以为低资源使用率设置比“高资源使用率”更严格的值。例如，对于CPU使用率较低（低于20%）的情况，可以将警报阈值设置为`cpu_idle<60%`。

### 注意事项

通常FE（Frontend）和BE（Backend）会一起进行监控，但有些指标只适用于FE或BE。

可能需要对一些机器进行批量设置以便监控。

### 附加信息

#### P99批量计算规则

节点每15秒收集一次数据并计算一个值，第99个百分位数是这15秒内的第99个百分位数。当QPS（每秒查询率）不高（例如，QPS低于10）时，这些百分位数的准确性不高。此外，将一分钟内（4 x 15秒）生成的四个值进行聚合，无论是使用求和还是平均函数，都没有意义。

对于P50、P90等其他百分位数也是如此。

#### 集群错误监控

> 一些不期望的集群错误需要及时发现并解决，以保持集群的稳定性。如果错误较为轻微（例如SQL语法错误等），但**不能从重要错误项中剔除**，建议先进行监控，随后再进行区分。

## 使用Prometheus+Grafana

StarRocks可以利用[Prometheus](https://prometheus.io/)来监控数据存储，并使用[Grafana](https://grafana.com/)来实现结果的可视化。

### 组件

> 本文档介绍了基于Prometheus和Grafana实现的StarRocks可视化监控解决方案。StarRocks不负责维护或开发这些组件。有关Prometheus和Grafana的更多详细信息，请参阅它们的官方网站。

#### Prometheus

Prometheus是一个时间序列数据库，具有多维数据模型和灵活的查询语句。它通过从被监控系统中拉取或推送数据来收集数据，并将这些数据存储在其时间序列数据库中。它通过丰富的多维数据查询语言满足不同用户的需求。

#### Grafana

Grafana是一个开源的指标分析和可视化系统，支持多种数据源。Grafana通过相应的查询语句从数据源检索数据，并允许用户创建图表和仪表板来可视化数据。

### 监控架构

![8.10.2-1](../assets/8.10.2-1.png)

Prometheus从FE/BE接口拉取指标，然后将数据存储到其时间序列数据库中。

在Grafana中，用户可以配置Prometheus作为数据源，以自定义Dashboard。

### 部署

#### Prometheus

**1.** 从[Prometheus官方网站](https://prometheus.io/download/)下载最新版本的Prometheus。以prometheus-2.29.1.linux-amd64版本为例。

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.29.1/prometheus-2.29.1.linux-amd64.tar.gz
tar -xf prometheus-2.29.1.linux-amd64.tar.gz
```

**2.** 在`vi prometheus.yml`中添加配置

```yml
# my global config
global:
  scrape_interval: 15s # 全局采集间隔，默认为1m，这里设置为15s
  evaluation_interval: 15s # 全局规则触发间隔，默认为1m，这里设置为15s

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'StarRocks_Cluster01' # 每个集群称为一个作业，作业名称可自定义
    metrics_path: '/metrics' # 指定获取指标的Restful API

    static_configs:
      - targets: ['fe_host1:http_port','fe_host2:http_port','fe_host3:http_port']
        labels:
          group: fe # 这里配置了包含3个Frontends的FE组

      - targets: ['be_host1:http_port', 'be_host2:http_port', 'be_host3:http_port']
        labels:
          group: be # 这里配置了包含3个Backends的BE组
  - job_name: 'StarRocks_Cluster02' # Prometheus可以监控多个StarRocks集群
    metrics_path: '/metrics'

    static_configs:
      - targets: ['fe_host1:http_port','fe_host2:http_port','fe_host3:http_port']
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

该命令将Prometheus作为后台服务运行，并将其Web端口设置为9090。配置完成后，Prometheus开始收集数据并将其存储在`./data`目录下。

**4.** 访问Prometheus

可以通过浏览器界面访问Prometheus。只需在浏览器中打开9090端口即可。转到`状态 -> 目标`查看所有分组作业的监控主机节点。在正常情况下，所有节点应为`UP`状态。如果节点状态不是`UP`，可以先访问StarRocks的metrics接口（`http://fe_host:fe_http_port/metrics`或`http://be_host:be_http_port/metrics`）检查是否可访问，或查看Prometheus文档进行故障排除。

![8.10.2-6](../assets/8.10.2-6.png)

一个简单的Prometheus已经构建并配置完成。更高级的使用方法，请参考[官方文档](https://prometheus.io/docs/introduction/overview/)。

#### Grafana

**1.** 从[Grafana官方网站](https://grafana.com/grafana/download)下载最新版本的Grafana。以grafana-8.0.6.linux-amd64版本为例。

```SHELL
wget https://dl.grafana.com/oss/release/grafana-8.0.6.linux-amd64.tar.gz
tar -zxf grafana-8.0.6.linux-amd64.tar.gz
```

**2.** 在`vi ./conf/defaults.ini`中添加配置

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

```Plain
nohup ./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

### 仪表板

#### 仪表板配置

通过上一步配置的地址`http://grafana_host:8000`登录Grafana，使用默认用户名和密码（即admin, admin）。

**1.** 添加数据源。

配置路径：`配置-->数据源-->添加数据源-->Prometheus`

数据源配置说明

![8.10.2-2](../assets/8.10.2-2.png)

* 名称：数据源的名称，可以自定义，例如`starrocks_monitor`
* URL：Prometheus的Web地址，例如`http://prometheus_host:9090`
* 访问方式：选择Server方法，即Grafana所在的服务器访问Prometheus。其余选项保持默认。

点击底部的“保存并测试”，如果显示“数据源工作正常”，则表示数据源已经可用。

**2.** 添加仪表板。

下载仪表板。

> **注意**
> StarRocks v1.19.0和v2.4.0版本中的指标名称已更改。您必须根据您的StarRocks版本下载相应的仪表板模板：
* [v1.19.0之前版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview.json)
* [v1.19.0至v2.4.0（不包括）版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-19.json)
* [v2.4.0及之后版本的仪表板模板](http://starrocks-thirdparty.oss-cn-zhangjiakou.aliyuncs.com/StarRocks-Overview-24-new.json)

仪表板模板会不定期更新。

确认数据源可用后，点击`+`号添加新的Dashboard，这里我们使用上面下载的StarRocks Dashboard模板。转到`导入`->`上传Json文件`加载下载的json文件。

加载后，您可以为仪表板命名。默认名称为`StarRocks Overview`。然后选择`starrocks_monitor`作为数据源。点击`导入`完成导入。之后，您应该能看到仪表板。

#### 仪表板说明

为您的仪表板添加说明。每个版本的说明都应进行更新。

**1.** 顶部栏

![8.10.2-3](../assets/8.10.2-3.png)

左上角显示仪表板名称。右上角显示当前时间范围。使用下拉菜单选择不同的时间范围并指定页面刷新间隔。`cluster_name`：Prometheus配置文件中每个作业的`job_name`，代表一个StarRocks集群。您可以选择一个集群，并在图表中查看其监控信息。

* `fe_master`：集群的领导节点。
* `fe_instance`：对应集群的所有前端节点。选择后可查看图表中的监控信息。
* `be_instance`：对应集群的所有后端节点。选择后可查看图表中的监控信息。
* `interval`：部分图表显示与监控项相关的间隔。间隔时间可自定义（注意：15秒间隔可能导致部分图表无法显示）。

**2.** 行

![8.10.2-4](../assets/8.10.2-4.png)

在Grafana中，`Row`是图表的集合。您可以点击以折叠一个`Row`。当前Dashboard包含以下`Rows`：

* `Overview`：显示所有StarRocks集群的概览。
* `Cluster Overview`：显示所选集群的概览。
* `Query Statistic`：监控所选集群的查询统计。
* `Jobs`：监控导入作业。
* `Transaction`：监控事务。
* `FE JVM`：监控所选前端的JVM。
* `BE`：显示所选集群的后端。
* `BE Task`：显示所选集群后端的任务。

**3.** 典型图表分为以下几个部分。

![8.10.2-5](../assets/8.10.2-5.png)

* 将鼠标悬停在左上角的`i`图标上可查看图表说明。
* 点击下方的图例可以单独查看某个项目。再次点击可显示所有项目。
* 在图表中拖动选择特定的时间范围。
* 所选集群的名称显示在标题的`[]`中。
* 数值可能对应左侧或右侧的Y轴，可以通过图例末尾的`-right`来区分。
* 点击图表名称可以编辑名称。
```markdown
* 值可能对应左侧Y轴或右侧Y轴，可通过图例末尾的-right来区分。
* 点击图表名称可以编辑名称。

### 其他

如果您需要在自己的Prometheus系统中访问监控数据，可以通过以下接口进行访问。

* FE: fe_host:fe_http_port/metrics
* BE: be_host:be_web_server_port/metrics

如果需要JSON格式，请访问以下接口。

* FE: fe_host:fe_http_port/metrics?type=json
* BE: be_host:be_web_server_port/metrics?type=json
```