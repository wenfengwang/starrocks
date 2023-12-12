---
displayed_sidebar: "Japanese"
---

# 管理者 チェック タブレット

## 説明

このステートメントは、一連のタブレットをチェックするために使用されます。

構文:

```sql
ADMIN CHECK TABLE (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

注意:

1. タブレットIDとPROPERTIESの「type」プロパティは指定する必要があります。

2. 現在、「type」は次のみサポートしています：

   一貫性: タブレットのレプリカの一貫性をチェックします。このコマンドは非同期です。送信後、StarRocksは対応するタブレットの一貫性をチェックし始めます。最終結果は、「/statistic」のSHOW PROCの結果のInconsistentTabletNum列に表示されます。

## 例

1. 指定されたタブレットのレプリカの一貫性をチェック

    ```sql
    ADMIN CHECK TABLET (10000, 10001)
    PROPERTIES("type" = "consistency");
    ```