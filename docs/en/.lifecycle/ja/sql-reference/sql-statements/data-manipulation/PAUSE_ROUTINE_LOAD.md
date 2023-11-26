---
displayed_sidebar: "Japanese"
---

# ルーチンロードの一時停止

## 説明

このステートメントは、ルーチンロードジョブを一時停止しますが、ジョブを終了しません。再開するには、[RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md) を実行します。ロードジョブが一時停止された後、[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md) と [ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md) を実行して、一時停止されたロードジョブに関する情報を表示および変更することができます。

> **注意**
>
> データがロードされるテーブルに対してLOAD_PRIV権限を持つユーザーのみが、このテーブル上のルーチンロードジョブを一時停止する権限を持ちます。

## 構文

```SQL
PAUSE ROUTINE LOAD FOR <db_name>.<job_name>;
```

## パラメータ

| パラメータ | 必須 | 説明                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| db_name   |      | ルーチンロードジョブを一時停止するデータベースの名前。                 |
| job_name  | ✅    | ルーチンロードジョブの名前。テーブルには複数のルーチンロードジョブがある場合がありますが、識別可能な情報を使用して意味のあるルーチンロードジョブ名を設定することをお勧めします。たとえば、Kafkaトピック名やロードジョブを作成した時刻などです。同じデータベース内でルーチンロードジョブの名前は一意である必要があります。 |

## 例

データベース `example_db` のルーチンロードジョブ `example_tbl1_ordertest1` を一時停止します。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```
