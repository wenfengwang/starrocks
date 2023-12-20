---
displayed_sidebar: English
---

# 删除数据库

## 描述

在 StarRocks 中删除一个数据库。

> **注意**
> 此操作需要对目标数据库具有**DROP**权限。

## 语法

```sql
DROP DATABASE [IF EXISTS] <db_name> [FORCE]
```

注意以下几点：

- 执行 DROP DATABASE 命令删除数据库后，您可以在指定的保留期限内（默认保留期为一天）使用[RECOVER](../data-definition/RECOVER.md)语句来恢复已删除的数据库，但是随数据库一起删除的管道（从 v3.2 版本开始支持）将无法恢复。
- 如果您执行 DROP DATABASE FORCE 命令来删除数据库，该数据库将会被直接删除，不会检查数据库内是否存在未完成的操作，且删除后无法恢复。通常不建议使用此操作。
- 删除数据库时，所有属于该数据库的管道（从 v3.2 版本开始支持）也会随之被删除。

## 示例

1. 删除名为 db_text 的数据库。

   ```sql
   DROP DATABASE db_test;
   ```

## 参考资料

- [CREATE DATABASE（创建数据库）](../data-definition/CREATE_DATABASE.md)
- [SHOW CREATE DATABASE](../data-manipulation/SHOW_CREATE_DATABASE.md)（显示创建数据库的语句）
- [使用指定数据库](../data-definition/USE.md)
- [DESC](../Utility/DESCRIBE.md)（描述表结构）
- [显示数据库列表](../data-manipulation/SHOW_DATABASES.md)
