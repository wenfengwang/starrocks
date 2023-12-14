---
displayed_sidebar: "Chinese"
---

# window_funnel

## 描述

在滑动窗口中搜索事件链，并计算事件链中连续事件的最大数量。此函数通常用于分析转化率。自 v2.3 版本起，它得到支持。

此函数按照以下规则工作：

- 它从事件链中的第一个事件开始计数。如果找到第一个事件，事件计数器设置为 1，并且滑动窗口开始。如果未找到第一个事件，则返回 0。

- 在滑动窗口中，如果事件链中的事件按顺序发生，则计数器递增。如果滑动窗口已超出，则不再递增事件计数器。

- 如果多个事件链匹配指定条件，则返回最长的事件链。

## 语法

```Plain
BIGINT window_funnel(BIGINT window, DATE|DATETIME time, INT mode, array[cond1, cond2, ..., condN])
```

## 参数

- `window`：滑动窗口的长度。支持的数据类型为 BIGINT。单位取决于 `time` 参数。如果 `time` 的数据类型为 DATE，则单位为天。如果 `time` 的数据类型为 DATETIME，则单位为秒。

- `time`：包含时间戳的列。支持 DATE 和 DATETIME 数据类型。

- `mode`：事件链被过滤的模式。支持的数据类型为 INT。取值范围：0、1、2。
  - `0` 是默认值，表示常规漏斗计算。
  - `1` 表示 `DEDUPLICATION` 模式，即过滤的事件链不能有重复事件。假设 `array` 参数为 `[event_type = 'A', event_type = 'B', event_type = 'C', event_type = 'D']`，原始事件链为 "A-B-C-B-D"。事件 B 重复，过滤后的事件链为 "A-B-C"。
  - `2` 表示 `FIXED` 模式，即过滤的事件链不能有打乱指定顺序的事件。假设使用前述的 `array` 参数，原始事件链为 "A-B-D-C"。事件 D 打乱了顺序，过滤后的事件链为 "A-B"。
  - `4` 表示 `INCREASE` 模式，即过滤的事件必须具有严格增加的时间戳。重复的时间戳会打乱事件链。此模式自 v2.5 版本起得到支持。

- `array`：定义的事件链。必须为数组。

## 返回值

返回 BIGINT 类型的值。

## 示例

**示例 1**：基于 `uid` 计算连续事件的最大数量。滑动窗口为 1800 秒，过滤模式为 `0`。

此示例使用 `action` 表，其中的数据已按 `uid` 排序。

```Plaintext
mysql> select * from action;
+------+------------+---------------------+
| uid  | event_type | time                |
+------+------------+---------------------+
| 1    | Browse     | 2020-01-02 11:00:00 |
| 1    | Click      | 2020-01-02 11:10:00 |
| 1    | Order      | 2020-01-02 11:20:00 |
| 1    | Pay        | 2020-01-02 11:30:00 |
| 1    | Browse     | 2020-01-02 11:00:00 |
| 2    | Order      | 2020-01-02 11:00:00 |
| 2    | Pay        | 2020-01-02 11:10:00 |
| 3    | Browse     | 2020-01-02 11:20:00 |
| 3    | Click      | 2020-01-02 12:00:00 |
| 4    | Browse     | 2020-01-02 11:50:00 |
| 4    | Click      | 2020-01-02 12:00:00 |
| 5    | Browse     | 2020-01-02 11:50:00 |
| 5    | Click      | 2020-01-02 12:00:00 |
| 5    | Order      | 2020-01-02 11:10:00 |
| 6    | Browse     | 2020-01-02 11:50:00 |
| 6    | Click      | 2020-01-02 12:00:00 |
| 6    | Order      | 2020-01-02 12:10:00 |
+------+------------+---------------------+
17 rows in set (0.01 sec)
```

执行以下语句：

```Plaintext
select uid,
       window_funnel(1800,time,0,[event_type='Browse', event_type='Click', 
        event_type='Order', event_type='Pay']) AS level
from action
group by uid
order by uid; 
+------+-------+
| uid  | level |
+------+-------+
| 1    |     4 |
| 2    |     0 |
| 3    |     1 |
| 4    |     2 |
| 5    |     2 |
| 6    |     3 |
+------+-------+
```

结果描述：

- `uid = 1` 的匹配事件链为 "Browse-Click-Order-Pay"，返回 `4`。最后一个 "Browse" 事件的时间（2020-01-02 11:00:00）不符合条件，因此未计算在内。

- `uid = 2` 的事件链未从第一个 "Browse" 事件开始，返回 `0`。

- `uid = 3` 的匹配事件链为 "Browse"，返回 `1`。"Click" 事件超出 1800 秒时间窗口，未计算在内。

- `uid = 4` 的匹配事件链为 "Browse-Click"，返回 `2`。

- `uid = 5` 的匹配事件链为 "Browse-Click"，返回 `2`。"Order" 事件（2020-01-02 11:10:00）不属于事件链，未计算在内。

- `uid = 6` 的匹配事件链为 "Browse-Click-Order"，返回 `3`。

**示例 2**：基于 `uid` 计算连续事件的最大数量。滑动窗口为 1800 秒，使用过滤模式 `0` 和 `1`。

此示例使用 `action1` 表，其中的数据已按 `time` 排序。

```Plaintext
mysql> select * from action1 order by time;
+------+------------+---------------------+ 
| uid  | event_type | time                |     
+------+------------+---------------------+
| 1    | Browse     | 2020-01-02 11:00:00 |
| 2    | Browse     | 2020-01-02 11:00:01 |
| 1    | Click      | 2020-01-02 11:10:00 |
| 1    | Order      | 2020-01-02 11:29:00 |
| 1    | Click      | 2020-01-02 11:29:50 |
| 1    | Pay        | 2020-01-02 11:30:00 |
| 1    | Click      | 2020-01-02 11:40:00 |
+------+------------+---------------------+
7 rows in set (0.03 sec)
```

执行以下语句：

```Plaintext
select uid,
       window_funnel(1800,time,0,[event_type='Browse', 
        event_type='Click', event_type='Order', event_type='Pay']) AS level
from action1
group by uid
order by uid;
+------+-------+
| uid  | level |
+------+-------+
| 1    |     4 |
| 2    |     1 |
+------+-------+
2 rows in set (0.02 sec)
```

对于 `uid = 1`，"Click" 事件（2020-01-02 11:29:50）是重复事件，但仍然计算在内，因为使用了模式 `0`。因此，返回 `4`。

将 `mode` 更改为 `1`，再次执行该语句。

```Plaintext
+------+-------+
| uid  | level |
+------+-------+
| 1    |     3 |
| 2    |     1 |
+------+-------+
2 rows in set (0.05 sec)
```

去重后筛选得到的最长事件链为 "Browse-Click-Order"，返回 `3`。

**示例 3**：基于 `uid` 计算连续事件的最大数量。滑动窗口为 1900 秒，使用过滤模式 `0` 和 `2`。

此示例使用 `action2` 表，其中的数据已按 `time` 排序。

```Plaintext
```plaintext
mysql> select * from action2 order by time;
+------+------------+---------------------+
| uid  | event_type | time                |
+------+------------+---------------------+
| 1    | 浏览       | 2020-01-02 11:00:00 |
| 2    | 浏览       | 2020-01-02 11:00:01 |
| 1    | 点击       | 2020-01-02 11:10:00 |
| 1    | 付款       | 2020-01-02 11:30:00 |
| 1    | 下单       | 2020-01-02 11:31:00 |
+------+------------+---------------------+
5 rows in set (0.01 sec)

```

执行以下语句：

```plaintext
select uid,
       window_funnel(1900,time,0,[event_type='浏览', event_type='点击', 
        event_type='下单', event_type='付款']) AS level
from action2
group by uid
order by uid;
+------+-------+
| uid  | level |
+------+-------+
| 1    |     3 |
| 2    |     1 |
+------+-------+

2 rows in set (0.02 sec)

```

对于 `uid = 1`，返回 `3`，因为使用了 `mode 0`，且“付款”事件（2020-01-02 11:30:00）没有中断事件链。

将 `mode` 更改为 `2`，再次执行该语句。

```plaintext
select uid,
       window_funnel(1900,time,2,[event_type='浏览', event_type='点击', 
        event_type='下单', event_type='付款']) AS level
from action2
group by uid
order by uid;
+------+-------+
| uid  | level |
+------+-------+
| 1    |     2 |

| 2    |     1 |

+------+-------+

2 rows in set (0.06 sec)

```

返回 `2`，因为“付款”事件中断了事件链，事件计数停止。过滤后的事件链为“浏览-点击”。

**示例 4**：基于 `uid` 计算连续事件的最大数量。滑动窗口为 1900 秒，使用过滤模式 `0` 和 `4`。

此示例使用表 `action3`，其中数据按 `time` 排序。

```plaintext
select * from action3 order by time;
+------+------------+---------------------+
| uid  | event_type | time                |
+------+------------+---------------------+
| 1    | 浏览       | 2020-01-02 11:00:00 |

| 1    | 点击       | 2020-01-02 11:00:01 |

| 2    | 浏览       | 2020-01-02 11:00:03 |
| 1    | 下单       | 2020-01-02 11:00:31 |
| 2    | 点击       | 2020-01-02 11:00:03 |
| 2    | 下单       | 2020-01-02 11:01:03 |
+------+------------+---------------------+
3 rows in set (0.02 sec)
```

执行以下语句：

```plaintext
select uid,
       window_funnel(1900,time,0,[event_type='浏览', event_type='点击',
        event_type='下单']) AS level

from action3

group by uid

order by uid;
+------+-------+
| uid  | level |
+------+-------+
|    1 |     3 |
|    2 |     3 |
+------+-------+
```

`uid = 1` 和 `uid = 2` 的结果均为 `3`。

将 `mode` 更改为 `4`，再次执行该语句。

```plaintext
select uid,

       window_funnel(1900,time,4,[event_type='浏览', event_type='点击',

        event_type='下单']) AS level

from action3