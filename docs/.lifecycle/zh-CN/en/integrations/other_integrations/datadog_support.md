---
displayed_sidebar: "Chinese"
---

# 将 Datadog 与 StarRocks 集成

本主题描述了如何将您的 StarRocks 集群与 [Datadog](https://www.datadoghq.com/) 集成，Datadog 是一个监控和安全平台。

## 先决条件

开始之前，您必须在实例上安装以下软件：

- [Datadog Agent](https://docs.datadoghq.com/getting_started/agent/)
- Python

> **注意**
>
> 首次安装 Datadog Agent 时，Python 也会作为一个依赖项安装。我们建议您在接下来的步骤中使用此 Python。

## 准备 StarRocks 源代码

由于目前 Datadog 尚未为 StarRocks 提供集成工具包，您需要使用源代码进行集成。

1. 打开终端，切换到一个本地目录，该目录需要具有读写权限，并运行以下命令以创建一个专用目录来存放 StarRocks 源代码。

    ```shell
    mkdir -p starrocks
    ```

2. 使用以下命令或[GitHub](https://github.com/StarRocks/starrocks/tags)上的命令将 StarRocks 源代码包下载到您创建的目录中。

    ```shell
    cd starrocks
    # 将 <starrocks_ver> 替换为 StarRocks 的实际版本，例如："2.5.2"。
    wget https://github.com/StarRocks/starrocks/archive/refs/tags/<starrocks_ver>.tar.gz
    ```

3. 解压缩包中的文件。

    ```shell
    # 将 <starrocks_ver> 替换为 StarRocks 的实际版本，例如："2.5.2"。
    tar -xzvf <starrocks_ver>.tar.gz --strip-components 1
    ```

## 安装和配置 FE 集成工具包

1. 使用源代码安装 FE 的 Datadog 集成工具包。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_fe
    ```

2. 创建 FE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml**。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_fe.d
    sudo cp contrib/datadog-connector/starrocks_fe/datadog_checks/starrocks_fe/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml
    ```

3. 修改 FE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml**。

    以下是一些重要配置项的示例：

    | **配置项** | **示例** | **说明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | fe_metric_url | `http://localhost:8030/metrics` | 用于访问 StarRocks FE 指标的 URL。 |
    | metrics | `- starrocks_fe_*` | 要在 FE 上监控的指标。您可以使用通配符 `*` 来匹配配置项。 |

## 安装和配置 BE 集成工具包

1. 使用源代码安装 BE 的 Datadog 集成工具包。

    ```shell
    /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_be
    ```

2. 创建 BE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml**。

    ```shell
    sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_be.d
    sudo cp contrib/datadog-connector/starrocks_be/datadog_checks/starrocks_be/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml
    ```

3. 修改 BE 集成配置文件 **/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml**。

    以下是一些重要配置项的示例：

    | **配置项** | **示例** | **说明** |
    | -------------------------------------- | ------------ | ------------------------------------------------------------ |
    | be_metric_url | `http://localhost:8040/metrics` | 用于访问 StarRocks BE 指标的 URL。 |
    | metrics | `- starrocks_be_*` | 要在 BE 上监控的指标。您可以使用通配符 `*` 来匹配配置项。 |

## 重新启动 Datadog Agent

重新启动 Datadog Agent 以使配置生效。

```shell
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

## 验证集成

有关验证集成的说明，请参阅[Datadog 应用程序](https://docs.datadoghq.com/getting_started/application/)。

## 卸载集成工具包

当您不再需要集成工具包时，可以对其进行卸载。

- 若要卸载 FE 集成工具包，请运行以下命令：

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-fe
  ```

- 若要卸载 BE 集成工具包，请运行以下命令：

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-be
  ```