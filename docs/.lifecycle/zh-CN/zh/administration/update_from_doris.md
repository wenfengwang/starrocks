---
displayed_sidebar: "中文"
---

# 升级 Apache Doris 至 StarRocks

本文章档介绍如何将 Apache Doris 升级为 StarRocks。

当前仅支持从 Apache Doris 0.13.15（不包含）之前的版本升级至 StarRocks。如需将 Apache Doris 0.13.15 版本升级至 StarRocks，请联系官方人员以获得源代码修改的帮助。目前还不支持将 Apache Doris 0.14 或更高版本升级至 StarRocks。

> 注意事项：
>
> * 由于 StarRocks 支持 BE 向后兼容 FE，强烈建议**先升级 BE 节点，再升级 FE 节点**。升级操作顺序错误可能会导致新旧 FE 和 BE 节点之间不兼容，并可能导致 BE 节点停止服务。
> * 请在进行跨版本升级时加以谨慎。如果必须进行跨版本升级，请先在测试环境中验证无误后再在生产环境中进行。从 Apache Doris 升级至 Starrocks-2.x 版本需要事先启用 CBO，因而需要先将 Apache Doris 升级至 StarRocks 1.19.x 版本。您可以在[官网](https://www.mirrorship.cn/zh-CN/download)下载 1.19.x 版本的安装包。

## 检查升级环境

1. 使用 MySQL 客户端查看现有集群的信息。

    ```SQL
    SHOW FRONTENDS;
    SHOW BACKENDS;
    SHOW BROKER;
    ```

    记录下 FE、BE 的数量、IP 地址、版本等信息，以及 FE 的 Leader、Follower、Observer 等详细情况。

2. 确认安装路径。

    以下示例假定原 Apache Doris 安装在 `/home/doris/doris/` 目录，而 Starrocks 新安装在 `/home/starrocks/starrocks/` 目录。为避免误操作，后续步骤将使用绝对路径。请根据实际情况来调整路径。

    > 注意：在某些情况下，Apache Doris 的安装路径可能是 `/disk1/doris`。

3. 查看 BE 的配置文件。

    查看 **be.conf** 配置文件中有关 `default_rowset_type` 选项的设置：

    * 如果此项设为 `ALPHA`，表示所有数据均使用 segmentV1 格式，您需要更改它为 BETA，并进行后续检查和转换。
    * 如果此项设为 `BETA`，表示数据已采用 segmentV2 格式，但可能仍有部分 Tablet 或 Rowset 使用 segmentV1 格式，因此需要进行后续检查和转换。

4. 测试 SQL。

    ```SQL
    show databases;
    use {one_db};
    show tables;
    show data;
    select count(*) from {one_table};
    ```

## 转换数据格式

转化流程可能会很耗时。如果数据中存在大量的 segmentV1 格式数据，建议提前进行转化。

1. 检查文件格式。

    a. 下载并安装文件格式检查工具。

    ```bash
    # 您也可以通过 git clone 或直接从当前地址下载。
    wget http://Starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/show_segment_status.tar.gz
    tar -zxvf show_segment_status.tar.gz
    ```

    b. 修改安装路径下的 **conf** 文件，根据实际情况配置以下项目。

    ```conf
    [cluster]
    fe_host =10.0.2.170
    query_port =9030
    user = root
    query_pwd = ****

    # 以下配置为可选
    # 选择一个数据库
    db_names =              // 填写数据库名称
    # 选择一个表
    table_names =           // 填写表名称
    # 选择一个 BE，当值为 0 时表示所有 BE
    be_id = 0
    ```

    c. 完成设置后，运行检测脚本检查文件格式。

    ```bash
    python show_segment_status.py
    ```

    d. 检查工具输出的 `rowset_count` 数据的两个值是否相等。若不相等，证明存在使用 segmentV1 格式的表格，需要对其数据进行转换。

2. 转换数据格式。

    a. 将数据格式转换为 segmentV1 的表。

    ```SQL
    ALTER TABLE table_name SET ("storage_format" = "v2");
    ```

    b. 查看转换状态。当 `status` 字段值为 `FINISHED` 时，表示格式转换已完成。

    ```sql
    SHOW ALTER TABLE column;
    ```

    c. 再次运行文件格式检查工具，查看状态。如果 `storage_format` 已成功设置为 `V2`，但仍有 segmentV1 格式的数据，可通过以下方式进一步检查并转换：

      i.  分别对所有表进行查询，获取表格的元数据链接。

    ```sql
    SHOW TABLET FROM table_name;
    ```

      ii. 通过元数据链接获取 Tablet 元数据。

    示例：

    ```shell
    wget http://172.26.92.139:8640/api/meta/header/11010/691984191
    ```

      iii. 检查本地保存的元数据 JSON 文件，确认其中的 `rowset_type` 值。如果是 `ALPHA_ROWSET`，表明该数据采用 segmentV1 格式，需要转换。

      iv. 如果还有 segmentV1 格式数据需要转换，可参照以下示例操作。

    示例：

    ```SQL
    ALTER TABLE dwd_user_tradetype_d
    ADD TEMPORARY PARTITION p09
    VALUES [('2020-09-01'), ('2020-10-01')]
    ("replication_num" = "3")
    DISTRIBUTED BY HASH(`dt`, `c`, `city`, `trade_hour`);

    INSERT INTO dwd_user_tradetype_d TEMPORARY partition(p09)
    select * from dwd_user_tradetype_d partition(p202009);

    ALTER TABLE dwd_user_tradetype_d
    REPLACE PARTITION (p202009) WITH TEMPORARY PARTITION (p09);
    ```

## 升级 BE 节点

遵循**先升级 BE，后升级 FE**的原则进行升级。

> 注意：BE 升级采用逐台升级的方法。在当前机器升级完毕后，需间隔一些时间（建议间隔为一天），确保当前机器升级无误后，再升级其他机器。

1. 下载并解压 StarRocks 安装包，并将安装路径改为 **Starrocks**。

    ```bash
    cd ~
    tar xzf Starrocks-EE-1.19.6/file/Starrocks-1.19.6.tar.gz
    mv Starrocks-1.19.6/ Starrocks
    ```

2. 对比并拷贝原有 **conf/be.conf** 内容到新的 BE **conf/be.conf** 中。

    ```bash
    # 对比并编辑配置
    vimdiff /home/doris/Starrocks/be/conf/be.conf /home/doris/doris/be/conf/be.conf

    # 将以下配置拷贝至新 BE 配置文件中，建议使用原有数据目录。
    storage_root_path =     // 原有数据目录
    ```

3. 检查系统是否启用 Supervisor 来启动 BE，并且关闭 BE。

    ```bash
    # 检查原有进程（palo_be）。
    ps aux | grep palo_be
    # 检查 doris 账户是否运行着 Supervisor 进程。
    ps aux | grep supervisor
    ```

    * 如果已部署并使用 Supervisor 启动 BE，则需要通过 Supervisor 发送命令来关闭 BE。

    ```bash```
    cd /home/doris/doris/be && ./control.sh stop && cd -
```

* 如果没有部署 Supervisor，您需要手动关闭 BE。

```bash
sh /home/doris/doris/be/bin/stop_be.sh
```

4. 查看 BE 进程状态，确认节点 `Alive` 为 `false`，并且 `LastStartTime` 为最新时间。

```sql
SHOW BACKENDS;
```

确认 BE 进程已不存在。

```bash
ps aux | grep be
ps aux | grep supervisor
```

5. 启动新的 BE 节点。

```bash
sh /home/doris/Starrocks/be/bin/start_be.sh --daemon
```

6. 检查升级结果。

* 查看新 BE 进程状态。

```bash
ps aux | grep be
ps aux | grep supervisor
```

* 通过 MySQL 客户端查看节点 `Alive` 状态。

```sql
SHOW BACKENDS;
```

* 查看 **be.out**，检查是否有异常日志。
* 查看 **be.INFO**，检查 heartbeat 是否正常。
* 查看 **be.WARN**， 检查是否有异常日志。

> 注意：成功升级 2 个 BE 节点后，您可以通过 `SHOW FRONTENDS;` 查看 `ReplayedJournalId` 是否在增长，以检测导入是否存在问题。

## 升级 FE 节点

成功升级 BE 节点后，您可以继续升级 FE 节点。

> 注意：升级 FE 遵循**先升级 Observer，再升级 Follower，最后升级 Leader** 的顺序。

1. 修改 FE 源码（可选，如果您无需从 Apache Doris 0.13.15 版本升级，可以跳过此步骤）。

a. 下载源码 patch。

```bash
wget "http://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/upgrade_from_apache_0.13.15.patch"
```

b. 合入源码 patch。

* 通过 Git 合入。

```bash
git apply --reject upgrade_from_apache_0.13.15.patch
```

* 如果本地代码不在 Git 环境中，您也可以根据 patch 的内容手动合入。

c. 编译 FE 模块。

```bash
./build.sh --fe --clean
```

2. 通过 MySQL 客户端确定各 FE 节点的 Leader 和 Follower 身份。

```sql
SHOW FRONTENDS;
```

如果 `IsMaster` 为 `true`，代表该节点为 Leader，否则为 Follower 或 Observer。

3. 检查系统是否使用 Supervisor 启动的 FE，并关闭 FE。

```bash
# 查看原有进程。
ps aux | grep fe
# 检查 Doris 账户下是否有 Supervisor 进程
ps aux | grep supervisor
```

* 如果已部署并使用 Supervisor 启动 FE，您需要通过 Supervisor 发送命令关闭 FE。

```bash
cd /home/doris/doris/fe && ./control.sh stop && cd -
```

* 如果没有部署 Supervisor，您需要手动关闭 BE。

```bash
sh /home/doris/doris/fe/bin/stop_fe.sh
```

4. 查看 FE 进程状态，确认节点 `Alive` 为 `false`。

```sql
SHOW FRONTENDS;
```

确认 BE 进程已不存在。

```bash
ps aux | grep fe
ps aux | grep supervisor
```

5. 升级 Follower 或者 Leader 之前，务必确保备份元数据。您可以使用升级的日期做备份时间。

```bash
cp -r /home/doris/doris/fe/palo-meta /home/doris/doris/fe/doris-meta
```

6. 比较并拷贝原有 **conf/fe.conf** 的内容到新 FE 的 **conf/fe.conf** 中。

```bash
# 比较并修改和拷贝。
vimdiff /home/doris/Starrocks/fe/conf/fe.conf /home/doris/doris/fe/conf/fe.conf

# 修改此行配置到新 FE 中，在原 doris 目录，新的 meta 文件（后面会拷贝）。
meta_dir = /home/doris/doris/fe/doris-meta
# 维持原有java堆大小等信息。
JAVA_OPTS="-Xmx8192m
```

7. 启动新的 FE 节点。

```bash
sh /home/doris/Starrocks/fe/bin/start_fe.sh --daemon
```

8. 检查升级结果。

* 查看新 FE 进程状态。

```bash
ps aux | grep be
ps aux | grep supervisor
```

* 通过 MySQL 客户端查看节点 `Alive` 状态。

```sql
SHOW BACKENDS;
```

* 查看 **fe.out** 或 **fe.log**，检查是否有异常日志。
* 如果 **fe.log** 始终是 UNKNOWN 状态，没有变成 Follower、Observer，说明升级出现问题。

> 注意：如果您通过修改源码升级，需要等元数据产生新 image 之后（既 **meta/image** 目录下有新的 **image.xxx** 文件生成），再将 **fe/lib** 路径替换回新的安装路径。

## 回滚方案

从 Apache Doris 升级的 StarRocks 暂时不支持回滚。建议您在测试环境验证测试成功后再升级至生产环境。

如果遇到无法解决的问题，您可以添加下方企业微信寻求帮助。

![二维码](../assets/8.3.1.png)