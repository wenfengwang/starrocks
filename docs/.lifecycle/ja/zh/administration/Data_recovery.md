---
displayed_sidebar: Chinese
---

# データの復元

この文書では、StarRocksで削除されたデータを復元する方法について説明します。

StarRocksは、誤って削除されたデータベース、テーブル、パーティションのデータ復元をサポートしています。テーブルやデータベースを削除した後、StarRocksはすぐにデータを物理的に削除するわけではなく、一定期間（デフォルトは1日）Trashに保持します。管理者は誤って削除されたデータを復元することができます。

> 注意
>
> * 復元操作は、一定期間内に削除されたメタデータのみを復元できます。デフォルトは1日です。`catalog_trash_expire_second` パラメータを **fe.conf** ファイルで設定することで変更できます。
> * メタデータが削除された後、システムに同名同タイプの新しいメタデータが作成された場合、以前に削除されたメタデータは復元できません。

## データベースの復元

以下のコマンドでデータベースを復元します。

```sql
RECOVER DATABASE db_name;
```

以下の例では、`example_db` という名前のデータベースを復元します。

```sql
RECOVER DATABASE example_db;
```

## テーブルの復元

以下のコマンドでテーブルを復元します。

```sql
RECOVER TABLE [db_name.]table_name;
```

以下の例では、`example_db` データベース内の `example_tbl` という名前のテーブルを復元します。

```sql
RECOVER TABLE example_db.example_tbl;
```

## パーティションの復元

以下のコマンドでパーティションを復元します。

```sql
RECOVER PARTITION partition_name FROM [db_name.]table_name;
```

以下の例では、`example_db` データベースの `example_tbl` テーブルにある `p1` という名前のパーティションを復元します。

```sql
RECOVER PARTITION p1 FROM example_db.example_tbl;
```
