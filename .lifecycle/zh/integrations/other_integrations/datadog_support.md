---
displayed_sidebar: English
---

# 将Datadog与StarRocks集成

本主题旨在介绍如何将您的StarRocks集群与[Datadog](https://www.datadoghq.com/)（一款监控和安全平台）进行集成。

## 先决条件

在开始之前，您需要在实例上安装以下软件：

- [Datadog代理](https://docs.datadoghq.com/getting_started/agent/)
- Python

> **注意**
> 当您首次安装Datadog代理时，Python会作为依赖项一并安装。我们建议您在接下来的步骤中使用该`Python`环境。

## 准备StarRocks源代码

由于Datadog目前还没有提供StarRocks的集成工具包，您需要使用源代码来实现集成。

1. 打开终端，导航至您拥有读写权限的本地目录，然后执行以下命令，在该目录下为StarRocks源代码创建一个专用文件夹。

   ```shell
   mkdir -p starrocks
   ```

2. 通过以下命令或在[GitHub](https://github.com/StarRocks/starrocks/tags)上下载StarRocks的源代码包到您刚创建的目录中。

   ```shell
   cd starrocks
   # Replace <starrocks_ver> with the actual version of StarRocks, for example, "2.5.2".
   wget https://github.com/StarRocks/starrocks/archive/refs/tags/<starrocks_ver>.tar.gz
   ```

3. 解压缩该软件包中的文件。

   ```shell
   # Replace <starrocks_ver> with the actual version of StarRocks, for example, "2.5.2".
   tar -xzvf <starrocks_ver>.tar.gz --strip-components 1
   ```

## 安装和配置FE集成工具包

1. 使用源代码安装Datadog的FE集成工具包。

   ```shell
   /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_fe
   ```

2. 创建FE集成配置文件**/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml**。

   ```shell
   sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_fe.d
   sudo cp contrib/datadog-connector/starrocks_fe/datadog_checks/starrocks_fe/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml
   ```

3. 修改FE集成配置文件**/etc/datadog-agent/conf.d/starrocks_fe.d/conf.yaml**。

   以下是一些重要配置项的示例：

   |配置|示例|描述|
|---|---|---|
   |fe_metric_url|http://localhost:8030/metrics|用于访问 StarRocks FE 指标的 URL。|
   |metrics|- starrocks_fe_*|要在 FE 上监控的指标。可以使用通配符*来匹配配置项。|

## 安装和配置BE集成工具包

1. 使用源代码安装Datadog的BE集成工具包。

   ```shell
   /opt/datadog-agent/embedded/bin/pip install contrib/datadog-connector/starrocks_be
   ```

2. 创建BE集成配置文件**/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml**。

   ```shell
   sudo mkdir -p /etc/datadog-agent/conf.d/starrocks_be.d
   sudo cp contrib/datadog-connector/starrocks_be/datadog_checks/starrocks_be/data/conf.yaml.example /etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml
   ```

3. 修改BE集成配置文件**/etc/datadog-agent/conf.d/starrocks_be.d/conf.yaml**。

   以下是一些重要配置项的示例：

   |配置|示例|说明|
|---|---|---|
   |be_metric_url|http://localhost:8040/metrics|用于访问 StarRocks BE 指标的 URL。|
   |metrics|- starrocks_be_*|BE 上要监控的指标。可以使用通配符*来匹配配置项。|

## 重启Datadog代理

重启Datadog代理，以便新的配置生效。

```shell
sudo systemctl stop datadog-agent
sudo systemctl start datadog-agent
```

## 验证集成

有关验证集成是否成功的指南，请参见[Datadog应用程序](https://docs.datadoghq.com/getting_started/application/)。

## 卸载集成工具包

当您不再需要这些集成工具包时，可以进行卸载。

- 要卸载FE集成工具包，请执行以下命令：

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-fe
  ```

- 要卸载BE集成工具包，请执行以下命令：

  ```shell
  /opt/datadog-agent/embedded/bin/pip uninstall datadog-starrocks-be
  ```
