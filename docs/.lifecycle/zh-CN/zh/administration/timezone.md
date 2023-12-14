---
displayed_sidebar: "中文"
---

# 设置时区

本文介绍了如何设置时区以及时区设置的影响。

## 设置会话/全局时区

您可以通过 `time_zone` 参数设置 StarRocks 的时区，并指定其生效范围是会话级还是全局级。

- 如指定会话级时区，执行 `SET time_zone = 'xxx';`。不同会话可以指定不同的时区，如果断开与 FE 的连接，时区设置将会失效。
- 如指定全局时区，执行 `SET global time_zone = 'xxx';`。FE 会将该时区设置持久化，即使与 FE 的连接断开后该设置仍然有效。

> **说明**
>
> 在导入数据之前，需要确保 StarRocks 的全局时区和部署 FE 机器的时区一致。否则，导入 DATE 类型的数据之后可能会出现问题。`system_time_zone` 参数表示部署 FE 机器的时区。当机器启动时，该机器的时区会自动被设置为该参数的值，并且不能手动更改。

### 时区格式

时区值不区分大小写，支持以下格式：

| **格式**     | **示例**                                                     |
| ------------ | ------------------------------------------------------------ |
| UTC 偏移量   | `SET time_zone = '+10:00';` `SET global time_zone = '-6:00';` |
| 标准时区名称 | `SET time_zone = 'Asia/Shanghai';` `SET global time_zone = 'America/Los_Angeles';` |

有关时区值的更多格式说明，请参见 [tz 数据库时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。

> **说明**
>
> 关于时区缩写，StarRocks 仅支持 CST 的格式，它会将 `CST` 解释为标准时区 `Asia/Shanghai`。

### 默认时区

`time_zone` 参数的默认值是 `Asia/Shanghai`。

## 查看时区设置

要查看当前的时区设置，执行以下命令：

```Plain_Text
 SHOW VARIABLES LIKE '%time_zone%';
```

## 时区设置的影响

- 时区设置会影响 SHOW LOAD 和 SHOW BACKENDS 语句返回的时间值，但不影响 CREATE TABLE 语句中 DATE 或 DATETIME 类型的分区列的 `LESS THAN` 子句指定的值，以及 DATE 或 DATETIME 类型的数据。
- 以下函数会受到时区设置的影响：
  - **from_unixtime**：给定一个 UTC 时间戳，返回指定时区的日期和时间。例如，如果全局时区设置为 `Asia/Shanghai`，`select FROM_UNIXTIME(0);` 将返回 `1970-01-01 08:00:00`。
  - **unix_timestamp**：给定一个指定时区的日期和时间，返回 UTC 时间戳。例如，如果全局时区设置为 `Asia/Shanghai`，`select UNIX_TIMESTAMP('1970-01-01 08:00:00');` 将返回 `0`。
  - **curtime**：返回指定时区的当前时间。例如，如果某时区当前时间为 16:34:05，`select CURTIME();` 将返回 `16:34:05`。
  - **now**：返回指定时区的当前日期和时间。例如，如果某时区的当前日期和时间为 2021-02-11 16:34:13，`select NOW();` 将返回 `2021-02-11 16:34:13`。
  - **convert_tz**：将一个日期和时间从一个时区转换到另一个时区。例如，`select CONVERT_TZ('2021-08-01 11:11:11', 'Asia/Shanghai', 'America/Los_Angeles');` 将返回 `2021-07-31 20:11:11`。