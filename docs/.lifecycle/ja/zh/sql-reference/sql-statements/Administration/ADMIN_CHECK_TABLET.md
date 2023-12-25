---
displayed_sidebar: Chinese
---

# 管理者チェックタブレット

## 機能

この文は、特定のチェック操作を一連のタブレットに対して実行するために使用されます。

:::tip

この操作には SYSTEM レベルの OPERATE 権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```sql
ADMIN CHECK TABLET (tablet_id1, tablet_id2, ...)
PROPERTIES("type" = "...")
```

説明：

1. タブレットIDリストとPROPERTIES内のtype属性を指定する必要があります。
2. 現在、typeは以下のみをサポートしています：

     **consistency**: タブレットのレプリカデータの一貫性をチェックします。このコマンドは非同期コマンドで、送信後、StarRocksは対応するタブレットの一貫性チェックジョブを開始します。最終結果は `SHOW PROC "/statistic";` の結果にある InconsistentTabletNum 列に反映されます。

## 例

1. 指定されたタブレット群に対してレプリカデータの一貫性チェックを行う

    ```sql
    ADMIN CHECK TABLET (10000, 10001)
    PROPERTIES("type" = "consistency");
    ```
