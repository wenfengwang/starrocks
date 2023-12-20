---
displayed_sidebar: English
---

# DataX 写入插件

## 介绍

StarRocksWriter 插件支持将数据写入 StarRocks 的目标表中。具体而言，StarRocksWriter 通过 [Stream Load](./StreamLoad.md) 功能将数据以 CSV 或 JSON 格式导入 StarRocks，并在内部对 `reader` 读取的数据进行缓存和批量导入，从而提高写入性能。整个数据流程是：`源数据 -> Reader -> DataX 通道 -> Writer -> StarRocks`。

[下载插件](https://github.com/StarRocks/DataX/releases)

请访问 https://github.com/alibaba/DataX 下载 DataX 的完整包，并将 starrockswriter 插件放置于 datax/plugin/writer/ 目录下。

使用以下命令进行测试：python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json

## 功能描述

### 配置示例

以下是一个配置文件示例，用于从 MySQL 读取数据并将其导入 StarRocks。

```json
{
    "job": {
        "setting": {
            "speed": {
                 "channel": 1
            },
            "errorLimit": {
                "record": 0,
                "percentage": 0
            }
        },
        "content": [
            {
                "reader": {
                    "name": "mysqlreader",
                    "parameter": {
                        "username": "xxxx",
                        "password": "xxxx",
                        "column": [ "k1", "k2", "v1", "v2" ],
                        "connection": [
                            {
                                "table": [ "table1", "table2" ],
                                "jdbcUrl": [
                                     "jdbc:mysql://127.0.0.1:3306/datax_test1"
                                ]
                            },
                            {
                                "table": [ "table3", "table4" ],
                                "jdbcUrl": [
                                     "jdbc:mysql://127.0.0.1:3306/datax_test2"
                                ]
                            }
                        ]
                    }
                },
               "writer": {
                    "name": "starrockswriter",
                    "parameter": {
                        "username": "xxxx",
                        "password": "xxxx",
                        "database": "xxxx",
                        "table": "xxxx",
                        "column": ["k1", "k2", "v1", "v2"],
                        "preSql": [],
                        "postSql": [], 
                        "jdbcUrl": "jdbc:mysql://172.28.17.100:9030/",
                        "loadUrl": ["172.28.17.100:8030", "172.28.17.100:8030"],
                        "loadProps": {}
                    }
                }
            }
        ]
    }
}

```

## StarRocksWriter 参数说明

* **用户名**

*   描述：StarRocks 数据库的用户名

*   是否必填：是

*   默认值：无

* **password**（密码）

*   描述：StarRocks 数据库的密码

*   是否必填：是

*   默认值：无

* **数据库**

*   描述：StarRocks 表所在的数据库名称

*   是否必填：是

*   默认值：无

* **table**（表）

*   描述：要写入的 StarRocks 表的名称

*   是否必填：是

*   默认值：无

* **loadUrl**（加载地址）

*   描述：用于 Stream Load 的 StarRocks FE 地址，可以是多个 FE 地址，格式为 fe_ip:fe_http_port

*   是否必填：是

*   默认值：无

* **column**（列）

  * 描述：需要**写入数据的目标表字段**，列之间以逗号分隔。例如："column": ["id", "name", "age"]。
        > **列配置项必须明确指定，不能留空。**
注意：我们强烈建议不要留空，因为目标表的列数、类型等变更时，可能会导致作业运行错误或失败。配置项的顺序必须与 Reader 中的 querySQL 或列的顺序相同。

* 是否必填：是

* 默认值：无

* **preSql**（预执行 SQL）

* 描述：在向目标表写入数据之前执行的标准 SQL 语句

* 是否必填：否

* 默认值：无

* **jdbcUrl**（JDBC 连接地址）

*   描述：用于执行 preSql 和 postSql 的目标数据库的 JDBC 连接信息

*   是否必填：否

* 默认值：无

* **loadProps**（加载属性）

*   描述：StreamLoad 的请求参数，详细信息请参考 StreamLoad 介绍页面

*   是否必填：否

*   默认值：无

## 类型转换

默认情况下，传入数据会被转换为字符串，以 't' 作为列分隔符，'n' 作为行分隔符，以形成用于 StreamLoad 导入的 CSV 文件。要更改列分隔符，请适当配置 loadProps。

```json
"loadProps": {
    "column_separator": "\\x01",
    "row_delimiter": "\\x02" 
}
```

要将导入格式更改为 JSON，请适当配置 loadProps。

```json
"loadProps": {
    "format": "json",
    "strip_outer_array": true
}
```

> JSON 格式用于以 JSON 格式将数据导入到 StarRocks 中。

## 关于时区

如果源数据库位于其他时区，执行 datax.py 时需在命令行后添加相应的时区参数。

```json
"-Duser.timezone=xx"
```

例如，如果 DataX 导入的 Postgrest 数据源位于 UTC 时区，启动时需添加参数 "-Duser.timezone=GMT+0"。
