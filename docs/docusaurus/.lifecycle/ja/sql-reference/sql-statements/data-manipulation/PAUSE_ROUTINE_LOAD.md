---
displayed_sidebar: "Japanese"
---

# ルーチンロードの一時停止

## 説明

この文はルーチンロードジョブを一時停止しますが、このジョブを終了しません。[RESUME ROUTINE LOAD](./RESUME_ROUTINE_LOAD.md)を実行して再開できます。ロードジョブが一時停止されたら、[SHOW ROUTINE LOAD](./SHOW_ROUTINE_LOAD.md)と[ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md)を実行して一時停止されたロードジョブの情報を表示および変更できます。

> **注意**
>
> データがロードされるテーブルにLOAD_PRIV権限を持つユーザーしか、そのテーブルのルーチンロードジョブを一時停止する権限を持っていません。

## 構文

```SQL
PAUSE ROUTINE LOAD FOR <db_name>.<job_name>;
```

## パラメータ

| パラメータ | 必須     | 説明                                                      |
| ---------- | -------- | ---------------------------------------------------------- |
| db_name    |          | ルーチンロードジョブを一時停止するデータベースの名前。           |
| job_name   | ✅        | ルーチンロードジョブの名前。テーブルには複数のルーチンロードジョブが存在する場合があります。識別可能な情報を使用して意味のあるルーチンロードジョブ名を設定することをお勧めします。例えば、Kafkaトピック名やロードジョブを作成した時刻などを使用して、複数のルーチンロードジョブを区別するためです。データベース内でルーチンロードジョブの名前は一意でなければなりません。 |

## 例

データベース`example_db`で、ルーチンロードジョブ`example_tbl1_ordertest1`を一時停止します。

```sql
PAUSE ROUTINE LOAD FOR example_db.example_tbl1_ordertest1;
```