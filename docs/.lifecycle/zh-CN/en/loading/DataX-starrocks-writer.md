---
displayed_sidebar: "Chinese"
---

# DataX writer

## 介绍

StarRocksWriter插件允许将数据写入StarRocks的目标表。具体来说，StarRocksWriter通过[Stream Load](./StreamLoad.md)以CSV或JSON格式将数据导入StarRocks，并在内部缓存并批量导入`reader`读取的数据到StarRocks，以获得更好的写入性能。整体数据流为`source -> Reader -> DataX channel -> Writer -> StarRocks`。

[下载该插件](https://github.com/StarRocks/DataX/releases)

请前往 `https://github.com/alibaba/DataX` 下载完整的DataX包，并将starrockswriter插件放入 `datax/plugin/writer/` 目录。

使用以下命令进行测试:
`python datax.py --jvm="-Xms6G -Xmx6G" --loglevel=debug job.json`

## 功能描述

### 示例配置

以下是用于从MySQL读取数据并加载到StarRocks的配置文件。

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

## Starrockswriter 参数描述

* **username**

  * 描述: StarRocks数据库的用户名

  * 必需: 是

  * 默认值: 无

* **password**

  * 描述: StarRocks数据库的密码

  * 必需: 是

  * 默认: 无

* **database**

  * 描述: StarRocks表的数据库名称

  * 必需: 是

  * 默认: 无

* **table**

  * 描述: StarRocks表的表名

  * 必需: 是

  * 默认: 无

* **loadUrl**

  * 描述: StarRocks FE的流加载地址，可以是多个FE地址，格式为 `fe_ip:fe_http_port`。

  * 必需: 是

  * 默认值: 无

* **column**

  * 描述: **需要写入数据的** 目标表的字段，以逗号分隔的列。示例: "column": ["id", "name", "age"]。
    >**必须指定column配置项，不能留空。**
    >注意: 我们强烈不建议将其留空，因为当您更改目标表的列数、类型等时，您的作业可能会运行不正确或失败。配置项必须与reader中的querySQL或column的顺序相同。

* 必需: 是

* 默认值: 无

* **preSql**

* 描述: 在将数据写入目标表之前将执行的标准语句。

* 必需: 否

* 默认: 无

* **jdbcUrl**

  * 描述: 执行`preSql`和`postSql`的目标数据库的JDBC连接信息。
  
  * 必需: 否

* 默认: 无

* **loadProps**

  * 描述: StreamLoad的请求参数，详情请参阅StreamLoad介绍页面。

  * 必需: 否

  * 默认值: 无

## 类型转换

默认情况下，传入数据将转换为字符串，以 `t` 作为列分隔符，`n` 作为行分隔符，形成 `csv` 文件以进行StreamLoad导入。
要更改列分隔符，请正确配置 `loadProps`。

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

> `json` 格式用于writer以JSON格式将数据导入StarRocks。

## 关于时区

如果源tp库位于另一个时区，执行datax.py时，在命令行后添加以下参数

```json
"-Duser.timezone=xx"
```

例如，如果DataX导入Postgrest数据，并且源库是以UTC时间，则在启动时添加参数 "-Duser.timezone=GMT+0"。