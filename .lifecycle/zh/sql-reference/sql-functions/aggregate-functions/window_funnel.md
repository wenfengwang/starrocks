---
displayed_sidebar: English
---

# 窗口漏斗

## 描述

该函数在滑动窗口中寻找事件链，并计算事件链中最大的连续事件数量。这个函数通常用于分析转化率，自 v2.3 版本起提供支持。

此函数遵循以下规则运作：

- 它从事件链的第一个事件开始计数。如果找到了第一个事件，事件计数器就会设为 1，并且滑动窗口随之开始。如果没有找到第一个事件，则返回 0。

- 在滑动窗口内，如果事件链中的事件依次发生，则计数器会递增。一旦超出滑动窗口范围，事件计数器就不再增加。

- 如果有多条事件链符合给定条件，将返回最长的那一条事件链。

## 语法

```Plain
BIGINT window_funnel(BIGINT window, DATE|DATETIME time, INT mode, array[cond1, cond2, ..., condN])
```

## 参数

- window：滑动窗口的长度。支持的数据类型为 BIGINT。单位依据时间参数而定。如果时间数据类型为 DATE，则单位为天；如果时间数据类型为 DATETIME，则单位为秒。

- time：含有时间戳的列。支持 DATE 和 DATETIME 类型。

- mode：事件链过滤的模式。支持的数据类型为 INT。取值范围：0、1、2。
  - 0 是默认值，代表常规漏斗计算。
  - 1 代表去重模式（DEDUPLICATION），即过滤后的事件链中不能有重复事件。假设数组参数为 [event_type = 'A', event_type = 'B', event_type = 'C', event_type = 'D']，原始事件链为“A-B-C-B-D”。重复的事件 B 被去除，过滤后的事件链变为“A-B-C”。
  - 2 代表固定模式（FIXED），即过滤后的事件链中不能出现打乱预设顺序的事件。若使用上述数组参数，原始事件链为“A-B-D-C”。由于事件 D 打乱了顺序，过滤后的事件链变为“A-B”。
  - 4 代表递增模式（INCREASE），意味着过滤后的事件必须具有严格递增的时间戳。重复的时间戳会打断事件链。该模式自 2.5 版本起支持。

- array：定义的事件链，必须是一个数组。

## 返回值

返回 BIGINT 类型的值。

## 例子

**示例 1**：基于 `uid` 计算最大连续事件数量。滑动窗口设置为 1800 秒，过滤模式为 `0`。

本例使用的是按 uid 排序的 action 表。

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

- uid = 1 匹配到的事件链为“浏览-点击-下单-支付”，返回结果为 4。“浏览”事件的最后一次发生时间（2020-01-02 11:00:00）不满足条件，因此不被计入。

- uid = 2 的事件链没有从“浏览”这一第一个事件开始，返回结果为 0。

- uid = 3 匹配到的事件链为“浏览”，返回结果为 1。由于“点击”事件超出了 1800 秒的时间窗口，因此不被计入。

- uid = 4 匹配到的事件链为“浏览-点击”，返回结果为 2。

- uid = 5 匹配到的事件链为“浏览-点击”，返回结果为 2。“订单”事件（2020-01-02 11:10:00）不属于事件链，因此不被计入。

- uid = 6 匹配到的事件链为“浏览-点击-下单”，返回结果为 3。

**示例 2**：计算最大连续事件数量，基于 `uid`。滑动窗口设置为 1800 秒，使用过滤模式 `0` 和 `1`。

本例使用的是按时间排序的 action1 表。

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

对于 uid = 1，由于使用了模式 0，“点击”事件（2020-01-02 11:29:50）虽然是重复的，但仍被计入，因此返回结果为 4。

更改为模式 1 并重新执行语句。

```Plaintext
+------+-------+
| uid  | level |
+------+-------+
| 1    |     3 |
| 2    |     1 |
+------+-------+
2 rows in set (0.05 sec)
```

去重后的最长事件链为“浏览-点击-下单”，返回结果为 3。

**示例 3**：计算最大连续事件数量，基于 `uid`。滑动窗口设置为 1900 秒，并且使用过滤模式 `0` 和 `2`。

本例使用的是按时间排序的 action2 表。

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

由于使用了模式 0，对于 uid = 1，“支付”事件（2020-01-02 11:30:00）不会中断事件链，返回结果为 3。

更改为模式 2 并重新执行语句。

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

由于“支付”事件打断了事件链，事件计数器停止，返回结果为 2。过滤后的事件链为“浏览-点击”。

**示例 4**：基于 `uid` 计算最大连续事件数量。滑动窗口设置为 1900 秒，使用过滤模式 `0` 和 `4`。

本例使用的是按时间排序的 action3 表。

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

对于 uid = 1 和 uid = 2，返回结果为 3。

更改为模式 4 并重新执行语句。

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

对于 uid = 2，由于使用了严格递增的模式 4，“点击”事件与“浏览”事件同时发生，因此“点击”和“下单”事件都不被计入，返回结果为 1。

## 关键词

窗口漏斗, 漏斗分析, window_funnel
