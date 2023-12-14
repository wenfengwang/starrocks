```markdown
---
displayed_sidebar: "Chinese"
---

# 恢復數據

本文介紹如何恢複 StarRocks 中被刪除的數據。

StarRocks 支持對誤刪除的數據庫、表、分區進行數據恢複，在刪除表或數據庫之後，StarRocks 不會立刻對數據進行物理刪除，而是將其保留在 Trash 一段時間（默認為 1 天）。管理員可以對誤刪除的數據進行恢複。

> 注意
>
> * 恢複操作僅能恢複一段時間內刪除的元信息。默認為 1 天。您可通過配置 **fe.conf** 文件中的 `catalog_trash_expire_second` 參數修改。
> * 如果元信息被刪除後，系統新建了同名同類型的元信息，則之前刪除的元信息無法被恢複。

## 恢複數據庫

通過以下命令恢複數據庫。

```sql
RECOVER DATABASE db_name;
```

以下示例恢複名為 `example_db` 的數據庫。

```sql
RECOVER DATABASE example_db;
```

## 恢複表

通過以下命令恢複表。

```sql
RECOVER TABLE [db_name.]table_name;
```

以下示例恢複 `example_db` 數據庫中名為 `example_tbl` 的表。

```sql
RECOVER TABLE example_db.example_tbl;
```

## 恢複分區

通過以下命令恢複分區。

```sql
RECOVER PARTITION partition_name FROM [db_name.]table_name;
```

以下示例恢復 `example_db` 數據庫的 `example_tbl` 表中名為 `p1` 的分區。

```sql
RECOVER PARTITION p1 FROM example_db.example_tbl;
```