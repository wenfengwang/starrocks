```yaml
---
displayed_sidebar: "Japanese"
---

# hll_empty（HLL_EMPTY）

## 説明

データを挿入またはロードする際に、デフォルトの値を補完するために空の HLL 列を生成します。

## 構文

```Haskell
HLL_EMPTY()
```

## 戻り値

空の HLL を返します。

## 例

データを挿入する際にデフォルトの値を補完します。

```plain text
insert into hllDemo(k1,v1) values(10,hll_empty());
```

データをロードする際にデフォルトの値を補完します。

```plain text
curl --location-trusted -u <username>:<password> \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```