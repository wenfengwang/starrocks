---
displayed_sidebar: English
---

# DataX 编写器

## 介绍

StarRocksWriter 插件允许将数据写入 StarRocks 的目标表。具体来说，StarRocksWriter 通过 [Stream Load](./StreamLoad.md) 将数据以 CSV 或 JSON 格式导入到 StarRocks，并在内部缓存并批量导入读取的数据 `reader` 到 StarRocks，以获得更好的写入性能。整体数据流为 `source -> Reader -> DataX channel -> Writer -> StarRocks`。

[下载该插件](https://github.com/StarRocks/DataX/releases)

请前往 `https://github.com/alibaba/DataX` 下载 DataX 的完整包，并将 starrockswriter 插件放入 `datax/plugin/writer/` 目录。

使用以下命令进行测试：
`python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json`

## 功能说明

### 示例配置

以下是一个配置文件，用于从 MySQL 读取数据并将其加载到 StarRocks。

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

## Starrockswriter 参数说明

* **用户名**

  * 描述：StarRocks 数据库的用户名

  * 必需：是

  * 默认值：无

* **密码**

  * 描述：StarRocks 数据库的密码

  * 必需：是

  * 默认值：无

* **数据库**

  * 描述：StarRocks 表的数据库名称。

  * 必需：是

  * 默认值：无

* **表**

  * 描述：StarRocks 表的表名。

  * 必需：是

  * 默认值：无

* **加载 Url**

  * 描述：用于流加载的 StarRocks FE 地址，可以是多个 FE 地址，形式为 `fe_ip:fe_http_port`.

  * 必需：是

  * 默认值：无

* **列**

  * 描述：目标表中**需要写入数据**的字段，列之间用逗号分隔。示例： “column”： [“id”、“姓名”、“年龄”].
    >**列配置项必须指定，不能留空。**
    >注意：我们强烈建议您不要将其留空，因为当您更改目标表的列数、类型等时，您的作业可能会运行不正确或失败。配置项的顺序必须与读取器中的 querySQL 或列的顺序相同。

* 必需：是

* 默认值：否

* **预SQL**

* 描述：在向目标表写入数据之前，将执行标准语句。

* 必需：否

* 默认值：否

* **jdbcUrl**

  * 描述：用于执行的目标数据库的JDBC连接信息 `preSql` 。 `postSql`
  
  * 必需：否

* 默认值：否

* **loadProps**

  * 描述：StreamLoad的请求参数，详见StreamLoad介绍页。

  * 必需：否

  * 默认值：否

## 类型转换

默认情况下，传入数据将转换为字符串，`t`并用作列分隔符和 `n` 行分隔符，以 `csv` 用于 StreamLoad 导入的样式文件。
要更改列分隔符，请正确配置 `loadProps` 。

```json
"loadProps": {
    "column_separator": "\\x01",
    "row_delimiter": "\\x02" 
}
```

要将导入格式更改为 `json`， `loadProps` 请正确配置。

```json
"loadProps": {
    "format": "json",
    "strip_outer_array": true
}
```

>  `json` 该格式是 writer 将数据以 JSON 格式导入到 StarRocks 的格式。

## 关于时区

如果源 tp 库在另一个时区，则在执行 datax.py 时，在命令行后添加以下参数

```json
"-Duser.timezone=xx"
```

例如，如果 DataX 导入 Postgrest 数据，并且源库是 UTC 时间，则在启动时添加参数 “-Duser.timezone=GMT+0”。
