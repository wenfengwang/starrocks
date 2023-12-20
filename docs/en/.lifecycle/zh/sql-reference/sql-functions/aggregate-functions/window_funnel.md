---
displayed_sidebar: English
---

# window_funnel

## 描述

在滑动窗口中搜索事件链，并计算事件链中连续事件的最大数量。该函数常用于分析转化率。从 v2.3 开始支持。

该函数根据以下规则工作：

- 它从事件链中的第一个事件开始计数。如果找到第一个事件，则事件计数器设置为 1，并且滑动窗口启动。如果未找到第一个事件，则返回 0。

- 在滑动窗口中，如果事件链中的事件按顺序发生，则计数器会递增。如果超出了滑动窗口，则事件计数器不再递增。

- 如果多个事件链符合指定条件，则返回最长的事件链。

## 语法

```Plain
BIGINT window_funnel(BIGINT window, DATE|DATETIME time, INT mode, array[cond1, cond2, ..., condN])
```

## 参数

- `window`：滑动窗口的长度。支持的数据类型是 BIGINT。单位取决于 `time` 参数。如果 `time` 的数据类型为 DATE，则单位为天。如果 `time` 的数据类型为 DATETIME，则单位为秒。

- `time`：包含时间戳的列。支持 DATE 和 DATETIME 类型。

- `mode`：事件链过滤的模式。支持的数据类型是 INT。值范围：0、1、2。
  - `0` 为默认值，表示普通漏斗计算。
  - `1` 表示 `DEDUPLICATION` 模式，即过滤后的事件链不能有重复的事件。假设 `array` 参数为 `[event_type = 'A', event_type = 'B', event_type = 'C', event_type = 'D']`，原始事件链为 “A-B-C-B-D”。重复事件 B，过滤后的事件链为 “A-B-C”。
  - `2` 表示 `FIXED` 模式，即过滤后的事件链不能有打乱指定顺序的事件。假设使用前面的 `array` 参数，原始事件链为 “A-B-D-C”。事件 D 中断序列，过滤后的事件链为 “A-B”。
  - `4` 表示 `INCREASE` 模式，这意味着过滤的事件必须具有严格递增的时间戳。重复的时间戳会破坏事件链。从 2.5 版本开始支持该模式。

- `array`：定义的事件链。它必须是一个数组。

## 返回值

返回 BIGINT 类型的值。

## 示例

**示例 1**：根据 `uid` 计算最大连续事件数。滑动窗口为 1800 秒，过滤模式为 `0`。

此示例使用表 `action`，其中数据按 `uid` 排序。

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

结果说明：

- `uid = 1` 的匹配事件链为 “Browse-Click-Order-Pay”，返回 4。最后一次 “Browse” 事件的时间（2020-01-02 11:00:00）不满足条件，不计算在内。

- `uid = 2` 的事件链不是从第一个事件 “Browse” 开始，返回 0。

- `uid = 3` 的匹配事件链是 “Browse”，返回 1。 “Click” 事件超过 1800 秒的时间窗口，不被统计在内。

- `uid = 4` 的匹配事件链是 “Browse-Click”，返回 2。

- `uid = 5` 的匹配事件链是 “Browse-Click”，返回 2。 “Order” 事件（2020-01-02 11:10:00）不属于事件链，不统计。

- `uid = 6` 的匹配事件链为 “Browse-Click-Order”，返回 3。

**示例 2**：根据 `uid` 计算最大连续事件数。滑动窗口为 1800 秒，采用过滤模式 `0` 和 `1`。

本示例使用表 `action1`，其中数据按 `time` 排序。

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

对于 `uid = 1`，“Click” 事件 (2020-01-02 11:29:50) 是重复事件，但由于使用模式 `0`，因此仍会被计数。因此，返回 4。

将 `mode` 更改为 `1` 并再次执行该语句。

```Plaintext
+------+-------+
| uid  | level |
+------+-------+
| 1    |     3 |
| 2    |     1 |
+------+-------+
2 rows in set (0.05 sec)
```

去重后过滤出的最长事件链为 “Browse-Click-Order”，返回 3。

**示例 3**：根据 `uid` 计算最大连续事件数。滑动窗口为 1900 秒，使用过滤模式 `0` 和 `2`。

本示例使用表 `action2`，其中数据按 `time` 排序。

```Plaintext
mysql> select * from action2 order by time;
+------+------------+---------------------+
| uid  | event_type | time                |
+------+------------+---------------------+
| 1    | Browse     | 2020-01-02 11:00:00 |
| 2    | Browse     | 2020-01-02 11:00:01 |
| 1    | Click      | 2020-01-02 11:10:00 |
| 1    | Pay        | 2020-01-02 11:30:00 |
| 1    | Order      | 2020-01-02 11:31:00 |
+------+------------+---------------------+
5 rows in set (0.01 sec)
```

执行以下语句：

```Plaintext
select uid,
       window_funnel(1900,time,0,[event_type='Browse', event_type='Click', 
        event_type='Order', event_type='Pay']) AS level
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

对于 `uid = 1`，返回 3，因为使用模式 `0` 并且 “Pay” 事件 (2020-01-02 11:30:00) 不会中断事件链。

将 `mode` 更改为 `2` 并再次执行该语句。

```Plaintext
select uid,
       window_funnel(1900,time,2,[event_type='Browse', event_type='Click', 
        event_type='Order', event_type='Pay']) AS level
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

返回 2 是因为 “Pay” 事件扰乱了事件链并且事件计数器停止。过滤后的事件链是 “Browse-Click”。

**示例 4**：根据 `uid` 计算最大连续事件数。滑动窗口为 1900 秒，使用过滤模式 `0` 和 `4`。

本示例使用表 `action3`，其中数据按 `time` 排序。

```Plaintext
select * from action3 order by time;
+------+------------+---------------------+
| uid  | event_type | time                |
+------+------------+---------------------+
| 1    | Browse     | 2020-01-02 11:00:00 |
| 1    | Click      | 2020-01-02 11:00:01 |
| 2    | Browse     | 2020-01-02 11:00:03 |
| 1    | Order      | 2020-01-02 11:00:31 |
| 2    | Click      | 2020-01-02 11:00:03 |
| 2    | Order      | 2020-01-02 11:01:03 |
+------+------------+---------------------+
3 rows in set (0.02 sec)
```

执行以下语句：

```Plaintext
select uid,
       window_funnel(1900,time,0,[event_type='Browse', event_type='Click',
        event_type='Order']) AS level
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

对于 `uid = 1` 和 `uid = 2`，返回 3。

将 `mode` 更改为 `4` 并再次执行该语句。

```Plaintext
select uid,
       window_funnel(1900,time,4,[event_type='Browse', event_type='Click',
        event_type='Order']) AS level
from action3
group by uid
order by uid;
+------+-------+
| uid  | level |
+------+-------+
|    1 |     3 |
|    2 |     1 |
+------+-------+
1 row in set (0.02 sec)
```

对于 `uid = 2`，返回 1，因为使用模式 `4`（严格递增）。 “Click” 与 “Browse” 同时发生。因此，“Click” 和 “Order” 不计算在内。

## 关键词

window_funnel, 漏斗, window_funnel