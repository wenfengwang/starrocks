---
displayed_sidebar: "Japanese"
---

# ADMIN CANCEL REPAIR（管理者による修復のキャンセル）

## 説明

このステートメントは、高い優先度で指定されたテーブルまたはパーティションの修復をキャンセルするために使用されます。

## 構文

```sql
ADMIN CANCEL REPAIR TABLE テーブル名[ PARTITION (p1,...)]
```

注意
>
> このステートメントは、システムが指定されたテーブルまたはパーティションのシャーディングレプリカを高い優先度で修復しないことを示すだけです。デフォルトのスケジューリングにより、これらのコピーは修復されます。

## 例

1. 高い優先度での修復をキャンセルする

    ```sql
    ADMIN CANCEL REPAIR TABLE tbl PARTITION(p1);
    ```
