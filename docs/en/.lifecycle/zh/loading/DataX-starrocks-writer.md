---
displayed_sidebar: English
---

# DataX Writer

## 介绍

StarRocksWriter 插件允许将数据写入 StarRocks 的目标表。具体来说，StarRocksWriter 通过 [Stream Load](./StreamLoad.md) 以 CSV 或 JSON 格式将数据导入 StarRocks，并在内部缓存并批量导入 `reader` 读取的数据到 StarRocks，以获得更好的写入性能。整体数据流程为 `source -> Reader -> DataX channel -> Writer -> StarRocks`。

[下载插件](https://github.com/StarRocks/DataX/releases)

请前往 `https://github.com/alibaba/DataX` 下载 DataX 完整包，并将 starrockswriter 插件放入 `datax/plugin/writer/` 目录下。

使用以下命令进行测试：
`python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json`

## 功能说明

### 配置示例

这是一个配置文件示例，用于从 MySQL 读取数据并将其加载到 StarRocks。

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

* **username**

*   描述：StarRocks 数据库的用户名

*   必填：是

*   默认值：无

* **password**

*   描述：StarRocks 数据库的密码

*   必填：是

*   默认值：无

* **database**

*   描述：StarRocks 表所在的数据库名称。

*   必填：是

*   默认值：无

* **table**

*   描述：StarRocks 表的名称。

*   必填：是

*   默认值：无

* **loadUrl**

*   描述：StarRocks FE 进行 Stream Load 的地址，可以是多个 FE 地址，格式为 `fe_ip:fe_http_port`。

*   必填：是

*   默认值：无

* **column**

  * 描述：目标表中**需要写入数据的字段**，字段之间用逗号分隔。例如："column": ["id", "name", "age"]。
        > **column 配置项必须指定，不能留空。**
注意：我们强烈建议不要留空，因为当目标表的列数、类型等发生变化时，您的作业可能会运行不正确或失败。配置项的顺序必须与 reader 中的 querySQL 或 column 的顺序一致。

* 必填：是

* 默认值：无

* **preSql**

* 描述：在向目标表写入数据之前将执行的标准 SQL 语句。

* 必填：否

* 默认值：无

* **jdbcUrl**

*   描述：用于执行 `preSql` 和 `postSql` 的目标数据库的 JDBC 连接信息。

*   必填：否

* 默认值：无

* **loadProps**

*   描述：StreamLoad 的请求参数，详细信息请参考 StreamLoad 介绍页面。

*   必填：否

*   默认值：无

## 类型转换

默认情况下，传入的数据会转换为字符串，以 `\t` 作为列分隔符，`\n` 作为行分隔符，形成用于 StreamLoad 导入的 csv 文件。要更改列分隔符，请正确配置 `loadProps`。

```json
"loadProps": {
    "column_separator": "\\x01",
    "row_delimiter": "\\x02" 
}
```

要将导入格式更改为 `json`，请正确配置 `loadProps`。

```json
"loadProps": {
    "format": "json",
    "strip_outer_array": true
}
```

> `json` 格式是为了让 Writer 以 JSON 格式导入数据到 StarRocks。

## 关于时区

如果源数据库位于其他时区，执行 datax.py 时，在命令行后添加以下参数

```json
"-Duser.timezone=xx"
```

例如，如果 DataX 导入 Postgres 数据且源数据库位于 UTC 时区，启动时添加参数 `-Duser.timezone=GMT+0`。