---
displayed_sidebar: "Japanese"
---

# 日付

## 説明

DATEタイプ。現在の値の範囲は['0000-01-01', '9999-12-31']で、デフォルトのフォーマットは `YYYY-MM-DD` です。

## 例

例1：テーブルを作成する際に、列をDATEタイプとして指定します。

```SQL
CREATE TABLE dateDemo (
    pk INT COMMENT "範囲 [-2147483648, 2147483647]",
    make_time DATE NOT NULL COMMENT "YYYY-MM-DD"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk)
```

例2：DATETIME値をDATE値に変換します。

```sql
mysql> SELECT DATE('2003-12-31 01:02:03');
-> '2003-12-31'
```

詳細は、[date](../../sql-functions/date-time-functions/date.md) 関数を参照してください。