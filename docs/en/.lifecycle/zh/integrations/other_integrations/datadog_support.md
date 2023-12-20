---
displayed_sidebar: English
---

# 将 Datadog 与 StarRocks 集成

本主题介绍如何将 StarRocks 集群与 [Datadog](https://www.datadoghq.com/)（一个监控和安全平台）集成。

## 先决条件

在开始之前，您必须在实例上安装以下软件：

- [Datadog Agent](https://docs.datadoghq.com/getting_started/agent/)
- Python

> **注意**
> 当您第一次安装 Datadog Agent 时，Python 也会作为依赖项被安装。我们建议您在以下步骤中使用这个 Python 环境。

## 准备 StarRocks 源代码

由于 Datadog 尚未提供 StarRocks 的集成套件，因此您需要使用源代码来集成它们。

1. 启动终端，导航到您具有读写访问权限的本地目录，然后运行以下命令为 StarRocks 源代码创建专用目录。

   ```shell
   mkdir -p starrocks
   ```

2. 使用以下命令或在 [GitHub](https://github.com/StarRocks/starrocks/tags) 上下载 StarRocks 源码包到您创建的目录中。

   ```shell
   cd starrocks
   # 用 StarRocks 的实际版本号替换 <starrocks_ver>，例如："2.5.2"。
   wget https://github.com/StarRocks/starrocks/archive/refs/tags/<starrocks_ver>.tar.gz
   ```

3. 提取包中的文件。

   ```shell
   # 用 StarRocks 的实际版本号替换 <starrocks_ver>，例如："2.5.2"。
   tar -xzvf <starrocks_ver>.tar.gz --strip-components 1
   ```

## 安装和配置 FE 集成套件

1. 使用源代码安装 FE 的 Datadog 集成套件。

   ```shell
   /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_fe
   ```

2. 创建 FE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml**。

   ```shell
   sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_fe.d
   sudo cp contrib/datadog-connector/starrocks_fe/datadog_checks/starrocks_fe/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml
   ```

3. 修改 FE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml**。

   一些重要配置项的示例：

   | 配置 | 示例 | 说明 |
|---|---|---|
   | fe_metric_url | `http://localhost:8030/metrics` | 用于访问 StarRocks FE 指标的 URL。|
   | metrics | `- starrocks_fe_*` | 要在 FE 上监控的指标。可以使用通配符 `*` 来匹配配置项。|

## 安装和配置 BE 集成套件

1. 使用源代码安装 BE 的 Datadog 集成套件。

   ```shell
   /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_be
   ```

2. 创建 BE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml**。

   ```shell
   sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_be.d
   sudo cp contrib/datadog-connector/starrocks_be/datadog_checks/starrocks_be/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml
   ```

3. 修改 BE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml**。

   一些重要配置项的示例：

   | 配置 | 示例 | 说明 |
|---|---|---|
   | be_metric_url | `http://localhost:8040/metrics` | 用于访问 StarRocks BE 指标的 URL。|
   | metrics | `- starrocks_be_*` | BE 上要监控的指标。可以使用通配符 `*` 来匹配配置项。|

## 重新启动 Datadog Agent

重新启动 Datadog Agent 以使配置生效。

```shell
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

## 验证集成

有关验证集成的说明，请参阅 [Datadog 应用程序](https://docs.datadoghq.com/getting_started/application/)。

## 卸载集成套件

当您不再需要集成套件时，可以将其卸载。

- 要卸载 FE 集成套件，请运行以下命令：

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-fe
  ```

- 要卸载 BE 集成套件，请运行以下命令：

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-be
  ```