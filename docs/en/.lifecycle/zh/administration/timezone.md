---
displayed_sidebar: English
---

# 配置时区

本主题介绍如何配置时区以及时区设置的影响。

## 配置会话级时区或全局时区

您可以使用 `time_zone` 参数为 StarRocks 集群配置会话级时区或全局时区。

- 要配置会话级时区，请执行命令 `SET time_zone = 'xxx';`。您可以为不同的会话配置不同的时区。如果与 FE 断开连接，时区设置将失效。
- 要配置全局时区，请执行命令 `SET global time_zone = 'xxx';`。时区设置将被持久化在 FE 中，并且即使您与 FE 断开连接也依然有效。

> **注意**
> 在将数据加载到 StarRocks 之前，请将 StarRocks 集群的全局时区修改为与 `system_time_zone` 参数的值相同。否则，数据加载后，DATE 类型的数据可能会不正确。`system_time_zone` 参数指的是托管 FE 的机器的时区。当机器启动时，机器的时区会被记录为此参数的值。您无法手动配置此参数。

### 时区格式

`time_zone` 参数的值不区分大小写。参数的值可以是以下格式之一。

|**格式**|**示例**|
|---|---|
|UTC 偏移|`SET time_zone = '+10:00';` `SET global time_zone = '-6:00';`|
|时区名称|`SET time_zone = 'Asia/Shanghai';` `SET global time_zone = 'America/Los_Angeles';`|

有关时区格式的更多信息，请参见 [tz 数据库时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。

> **注意**
> 除了 CST 之外，不支持时区缩写。如果将 `time_zone` 的值设置为 `CST`，StarRocks 会将 `CST` 转换为 `Asia/Shanghai`。

### 默认时区

`time_zone` 参数的默认值是 `Asia/Shanghai`。

## 查看时区设置

要查看时区设置，请运行以下命令。

```plaintext
 SHOW VARIABLES LIKE '%time_zone%';
```

## 时区设置的影响

- 时区设置会影响 SHOW LOAD 和 SHOW BACKENDS 语句返回的时间值。但是，设置不会影响 CREATE TABLE 语句中指定的分区列为 DATE 或 DATETIME 类型时，LESS THAN 子句中指定的值。设置也不会影响 DATE 和 DATETIME 类型的数据。
- 时区设置影响以下函数的显示和存储：
  - **from_unixtime**：根据指定的 UTC 时间戳返回指定时区的日期和时间。例如，如果您的 StarRocks 集群的全局时区是 `Asia/Shanghai`，`select FROM_UNIXTIME(0);` 返回 `1970-01-01 08:00:00`。
  - **unix_timestamp**：根据指定时区的日期和时间返回 UTC 时间戳。例如，如果您的 StarRocks 集群的全局时区是 `Asia/Shanghai`，`select UNIX_TIMESTAMP('1970-01-01 08:00:00');` 返回 `0`。
  - **curtime**：返回指定时区的当前时间。例如，如果指定时区的当前时间是 16:34:05，`select CURTIME();` 返回 `16:34:05`。
  - **now**：返回指定时区的当前日期和时间。例如，如果指定时区的当前日期和时间是 2021-02-11 16:34:13，`select NOW();` 返回 `2021-02-11 16:34:13`。
  - **convert_tz**：将日期和时间从一个时区转换到另一个时区。例如，`select CONVERT_TZ('2021-08-01 11:11:11', 'Asia/Shanghai', 'America/Los_Angeles');` 返回 `2021-07-31 20:11:11`。