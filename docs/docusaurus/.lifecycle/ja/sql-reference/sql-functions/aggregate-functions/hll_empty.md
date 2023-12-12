---
displayed_sidebar: "Japanese"
---

# hll_empty

## 説明

データの挿入や読み込み時にデフォルト値を補完するために空のHLLカラムを生成します。

## 構文

```Haskell
HLL_EMPTY()
```

## 戻り値

空のHLLを返します。

## 例

データの挿入時にデフォルト値を補完します。

```plain text
insert into hllDemo(k1,v1) values(10,hll_empty());
```

データの読み込み時にデフォルト値を補完します。

```plain text
curl --location-trusted -u <username>:<password> \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```