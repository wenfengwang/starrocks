---
displayed_sidebar: Chinese
---

# hll_empty

## 機能

空の HLL 列を生成し、INSERT やデータのインポート時にデフォルト値を補充するために使用します。

## 文法

```Haskell
HLL_EMPTY()
```

## パラメータ説明

なし

## 戻り値の説明

空の HLL を返します。

## 例

例1：INSERT 時にデフォルト値を補充。

```plain text
insert into hllDemo(k1,v1) values(10,hll_empty());
```

例2：データをインポートする際にデフォルト値を補充。

```plain text
curl --location-trusted -u <username>:<password> \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```
