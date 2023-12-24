---
displayed_sidebar: English
---

# hll_empty

## 描述

在插入或加载数据时，生成一个空的 HLL 列，以补充默认值。

## 语法

```Haskell
HLL_EMPTY()
```

## 返回值

返回一个空的 HLL。

## 例子

补充默认值以插入数据。

```plain text
insert into hllDemo(k1,v1) values(10,hll_empty());
```

补充默认值以加载数据。

```plain text
curl --location-trusted -u <username>:<password> \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```
