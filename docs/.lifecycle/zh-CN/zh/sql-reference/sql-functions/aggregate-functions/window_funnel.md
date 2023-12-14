---
displayed_sidebar: "Chinese"
---

# window_funnel

## 功能

Searching for the list of events within a sliding time window, and calculating the maximum number of continuous events in the matching event chain. This function is a funnel function, and it is a common method of conversion analysis used to analyze the conversion rate of user behaviors at each stage.

This function follows the following rules:

- Start judgment from the first condition in the event chain. If the data contains events that meet the conditions, the counter is incremented by 1, and the time corresponding to this event is used as the start time of the sliding window. If no data that meets the first condition is found, 0 is returned.

- Within the sliding window, if the events in the event chain occur in sequence, the counter is incremented; if it exceeds the time window, the counter no longer increases.

- If there are multiple event chains that meet the conditions, the longest event chain is output.

This function is supported from version 2.3.

## 语法

```Haskell
BIGINT window_funnel(BIGINT window, DATE|DATETIME time, INT mode, array[cond1, cond2, ..., condN])
```

## 参数说明

- `window`：The size of the sliding window, type BIGINT. The unit depends on the `time` parameter. If the type of `time` is DATE, the window unit is in days; if the type of `time` is DATETIME, the window unit is in seconds.

- `time`: Column containing timestamps. Currently supports DATE and DATETIME types.

- `mode`: The filtering mode of the event chain, type INT. Value range: 0, 1, 2, 4.
  - The default value is `0`, which represents the most general funnel calculation.
  - When the mode is `1` (bit set to the 1st bit), it represents the `DEDUPLICATION` mode, meaning that the filtered event chain cannot have duplicate events. Assuming the `array` parameter is `[event_type='A', event_type='B', event_type='C', event_type='D']`, the original event chain is "A-B-C-B-D". Because event B is duplicated, the filtered event chain can only be "A-B-C".
  - When the mode is `2` (bit set to the 2nd bit), it represents the `FIXED` mode, meaning that the filtered event chain cannot have events that jump. Assuming the `array` parameter remains the same, the original event chain is "A-B-D-C", because event D jumps, the filtered event chain can only be "A-B".
  - When the mode is `4` (bit set to the 3rd bit), it represents the `INCREASE` mode, meaning that in the filtered event chain, the timestamps of continuous events must strictly increase. **This mode is supported from version 2.5.**

- `array`: The defined event chain, type ARRAY.

## 返回值说明

Returns a value of type BIGINT, the value is the maximum number of continuous events satisfying the conditions within the sliding window.

## 示例

Example 1: Select the maximum number of continuous events corresponding to different `uid`, with a window of 1800 seconds, and filtering mode of `0`.

Assuming there is a table `action`, with data sorted by `uid`:

```Plaintext
SELECT * FROM action;
+------+------------+---------------------+
| uid  | event_type | time                |
+------+------------+---------------------+
| 1    | browse     | 2020-01-02 11:00:00 |
| 1    | click      | 2020-01-02 11:10:00 |
| 1    | order      | 2020-01-02 11:20:00 |
| 1    | payment    | 2020-01-02 11:30:00 |
| 1    | browse     | 2020-01-02 11:00:00 |
| 2    | order      | 2020-01-02 11:00:00 |
| 2    | payment    | 2020-01-02 11:10:00 |
| 3    | browse     | 2020-01-02 11:20:00 |
| 3    | click      | 2020-01-02 12:00:00 |
| 4    | browse     | 2020-01-02 11:50:00 |
| 4    | click      | 2020-01-02 12:00:00 |
| 5    | browse     | 2020-01-02 11:50:00 |
| 5    | click      | 2020-01-02 12:00:00 |
| 5    | order      | 2020-01-02 11:10:00 |
| 6    | browse     | 2020-01-02 11:50:00 |
| 6    | click      | 2020-01-02 12:00:00 |
| 6    | order      | 2020-01-02 12:10:00 |
+------+------------+---------------------+
17 rows in set (0.01 sec)
```

Execute the following SQL statement to calculate the maximum number of continuous events:

```Plaintext
SELECT uid,
       window_funnel(1800,time,0,[event_type='browse', event_type='click', 
       event_type='order', event_type='payment'])
       AS level
FROM action
GROUP BY uid
ORDER BY uid; 
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

It can be seen that:

- For uid=1, the matching event chain is "browse-click-order-payment", output is 4, because the time of the last browse event does not meet the condition and is not included;

- For uid=2, the event chain does not start from the first event "browse", so the output is 0;

- For uid=3, the corresponding event chain is "browse", output is 1, because the "click" event exceeds the 1800s window and is not included;

- For uid=4, the corresponding event chain is "browse-click", output is 2;

- For uid=5, the event chain is "browse-click", output is 2, because the order time does not belong to this event chain and is not included;

- For uid=6, the event chain is "browse-click-order", output is 3.

Example 2: Select the maximum number of continuous events corresponding to different `uid`, with a window of 1800 seconds, and calculate the results for filtering modes `0` and `1` respectively.

Assuming there is a table `action1`, with data sorted by `time`:

```Plaintext
mysql> select * from action1 order by time;
+------+------------+---------------------+ 
| uid  | event_type | time                |     
+------+------------+---------------------+
| 1    | browse     | 2020-01-02 11:00:00 |
| 2    | browse     | 2020-01-02 11:00:01 |
| 1    | click      | 2020-01-02 11:10:00 |
| 1    | order      | 2020-01-02 11:29:00 |
| 1    | click      | 2020-01-02 11:29:50 |
| 1    | payment    | 2020-01-02 11:30:00 |
| 1    | click      | 2020-01-02 11:40:00 |
+------+------------+---------------------+
7 rows in set (0.03 sec)
```

Execute the following SQL statement to calculate the maximum number of continuous events:

```Plaintext
SELECT uid,
```plaintext
window_funnel(1800,time,0,[event_type='浏览', 
        event_type='点击', event_type='下单', event_type='支付'])
        AS level
FROM action1
GROUP BY uid
ORDER BY uid;

+------+-------+
| uid  | level |
+------+-------+
| 1    |     4 |
| 2    |     1 |
+------+-------+
2 rows in set (0.02 sec)
```

可以看到，对于 uid=1，即使“点击事件 (2020-01-02 11:29:50) ”已经重复出现，但是依然计入，最终输出 `4`，因为使用了模式 `0`。

将 `mode` 改为 `1`，进行去重，再次执行 SQL:

```plaintext
+------+-------+
| uid  | level |
+------+-------+
| 1 | 3 |
| 2 | 1 |
+------+-------+
2 rows in set (0.05 sec)
```

可以看到输出为 `3`，去重后筛选出的最长事件链为“浏览-点击-下单”。

示例三：筛选出 `uid` 对应的最大连续事件数，窗口为1900s，分别计算筛选模式为 `0` 和 `2` 的结果。

假设有表 `action2`，数据以 `time`排序：

```plaintext
mysql> select * from action2 order by time;
+------+------------+---------------------+
| uid  | event_type | time                |
+------+------------+---------------------+
| 1    | 浏览       | 2020-01-02 11:00:00 |
| 2    | 浏览       | 2020-01-02 11:00:01 |
| 1    | 点击       | 2020-01-02 11:10:00 |
| 1    | 支付       | 2020-01-02 11:30:00 |
| 1    | 下单       | 2020-01-02 11:31:00 |
+------+------------+---------------------+
5 rows in set (0.01 sec)
```

执行如下 SQL 语句：

```plaintext
SELECT uid,
       window_funnel(1900,time,0,[event_type='浏览', event_type='点击', 
        event_type='下单', event_type='支付'])
        AS level
FROM action2
GROUP BY uid
ORDER BY uid;
+------+-------+
| uid  | level |
+------+-------+
| 1    |     3 |
| 2    |     1 |
+------+-------+
2 rows in set (0.02 sec)
```

可以看到对于 uid=1，输出为 `3`，因为使用了模式 `0`，所以“支付 (2020-01-02 11:30:00)” 这一跳跃的事件并没有阻断筛选出的事件链。

将 `mode` 改为 `2`，再次执行 SQL：

```plaintext
SELECT uid,
       window_funnel(1900,time,2,[event_type='浏览', event_type='点击', 
        event_type='下单', event_type='支付'])
        AS level
FROM action2
GROUP BY uid
ORDER BY uid;
+------+-------+
| uid  | level |
+------+-------+
| 1    |     2 |
| 2    |     1 |
+------+-------+
2 rows in set (0.06 sec)
```

输出为 `2`，因为“支付”事件跳跃，停止计数，此时筛选出的最大事件链是“浏览-点击”。

示例四：筛选出 `uid` 对应的最大连续事件数，窗口为 1900s，分别计算筛选模式为 `0`（时间戳不需要严格递增）和 `4`（时间戳需要严格递增）的结果。

假设有表 `action3`，数据以 `time` 排序：

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

执行如下 SQL 语句：

```plaintext
SELECT uid,
       window_funnel(1900,time,0,[event_type='浏览', event_type='点击',
        event_type='下单'])
        AS level
FROM action3
GROUP BY uid
ORDER BY uid;
+------+-------+
| uid  | level |
+------+-------+
|    1 |     3 |
|    2 |     3 |
+------+-------+
```

对于 uid=1 和 2，输出均为 3。

将 `mode` 改为 `4`，再次执行 SQL：

```plaintext
SELECT uid, window_funnel(1900,time,4,[event_type='浏览', event_type='点击',
        event_type='下单'])
        AS level
FROM action3
GROUP BY uid
ORDER BY uid;
+------+-------+
| uid  | level |
+------+-------+
|    1 |     3 |
|    2 |     1 |
+------+-------+
1 row in set (0.02 sec)
```

对于 uid=2，输出为 1，因为指定了时间戳严格递增，该用户的“点击”和“浏览”发生在同一秒，因此“浏览”及其后行为均被忽略。

## Keywords

漏斗，漏斗函数，转化率，funnel