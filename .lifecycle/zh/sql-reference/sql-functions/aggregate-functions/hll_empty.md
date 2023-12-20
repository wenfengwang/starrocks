---
displayed_sidebar: English
---

# hll_empty

## 描述

在插入或加载数据时生成一个空的HLL列，用于补充默认值。

## 语法

```Haskell
HLL_EMPTY()
```

## 返回值

返回一个空的HLL。

## 示例

在插入数据时用以补充默认值。

```plain
insert into hllDemo(k1,v1) values(10,hll_empty());
```

在加载数据时用以补充默认值。

```plain
curl --location-trusted -u <username>:<password> \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```
