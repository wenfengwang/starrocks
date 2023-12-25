---
displayed_sidebar: Chinese
---

# STRING

## 説明

文字列型で、最大長は65533バイトです。

## 例

テーブル作成時にフィールドタイプとしてSTRINGを指定します。

```sql
CREATE TABLE stringDemo (
    pk INT COMMENT "range [-2147483648, 2147483647]",
    us_detail STRING COMMENT "上限値65533バイト"
) ENGINE=OLAP 
DUPLICATE KEY(pk)
COMMENT "OLAP"
DISTRIBUTED BY HASH(pk);
```

テーブルが正常に作成された後、`desc <table_name>;` を実行してテーブル情報を確認すると、STRING型が `VARCHAR(65533)` と表示されます。
