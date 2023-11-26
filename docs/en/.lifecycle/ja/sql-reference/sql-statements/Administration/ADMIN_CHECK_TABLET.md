---
displayed_sidebar: "Japanese"
---

# ADMIN CHECK TABLET（タブレットのチェック）

## 説明

このステートメントは、一連のタブレットをチェックするために使用されます。

構文:

```sql
ADMIN CHECK TABLE (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注意:

1. タブレットIDとPROPERTIESの「type」プロパティを指定する必要があります。

2. 現在、「type」は以下のみサポートしています:

   Consistency: タブレットのレプリカの整合性をチェックします。このコマンドは非同期です。送信後、StarRocksは対応するタブレット間の整合性をチェックし始めます。最終結果は、SHOW PROC "/statistic"のInconsistentTabletNum列に表示されます。

## 例

1. 指定されたタブレットのレプリカの整合性をチェックする

    ```sql
    ADMIN CHECK TABLET (10000, 10001)
    PROPERTIES("type" = "consistency");
    ```
