---
displayed_sidebar: "Japanese"
---

# ルーチンロードの実行を一時停止

## 説明

このステートメントは、ルーチンロードのジョブを一時停止しますが、このジョブを終了しません。[RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md)を実行して再開できます。ロードジョブが一時停止された後は、[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)および[ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md)を実行して、一時停止されたロードジョブの情報を表示および変更することができます。

> **注意**
>
> ロードされるデータが含まれるテーブルのLOAD_PRIV権限を持つユーザーのみが、このテーブルのルーチンロードジョブを一時停止する権限を持っています。

## 構文

```SQL
PAUSE ROUTINE LOAD FOR <db_name>.<job_name>;
```

## パラメータ

| パラメータ | 必須     | 説明                                                         |
| ---------- | -------- | ------------------------------------------------------------ |
| db_name    |          | ルーチンロードジョブの一時停止を行いたいデータベースの名前。   |
| job_name   | ✅       | ルーチンロードジョブの名前。1つのテーブルには複数のルーチンロードジョブがある場合がありますが、例えばKafkaトピック名やロードジョブを作成した時刻などの識別可能な情報を使用して意味のあるルーチンロードジョブ名を設定することを推奨します。ルーチンロードジョブの名前は、同じデータベース内で一意である必要があります。 |

## 例

データベース`example_db`でルーチンロードジョブ`example_tbl1_ordertest1`を一時停止します。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```