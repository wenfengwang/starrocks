```markdown
---
displayed_sidebar: "Japanese"
---

# パイプの中断または再開

## 説明

パイプを中断または再開します：

- ロードジョブが進行中（つまり`RUNNING`状態）の場合、ジョブのための`SUSPEND`パイプはジョブを中断します。
- ロードジョブでエラーが発生した場合、ジョブのための`RESUME`パイプではエラーのあるジョブを実行し続けます。

## 構文

```SQL
ALTER PIPE [ IF EXISTS ] <pipe_name> { SUSPEND | RESUME [ IF SUSPENDED ] }
```

## パラメータ

### pipe_name

パイプの名前です。

## 例

### パイプを中断する

データベース`mydatabase`内で`RUNNING`状態にある`user_behavior_replica`という名前のパイプを中断します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica SUSPEND;
```

パイプを問い合わせるために[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用すると、その状態が`SUSPEND`に変わったことがわかります。

### パイプを再開する

データベース`mydatabase`内で`user_behavior_replica`という名前のパイプを再開します：

```SQL
USE mydatabase;
ALTER PIPE user_behavior_replica RESUME;
```

パイプを問い合わせるために[SHOW PIPES](../../../sql-reference/sql-statements/data-manipulation/SHOW_PIPES.md)を使用すると、その状態が`RUNNING`に変わったことがわかります。
```