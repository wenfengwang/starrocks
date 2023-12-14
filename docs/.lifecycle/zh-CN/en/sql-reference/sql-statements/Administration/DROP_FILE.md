---
displayed_sidebar: "Chinese"
---

# 删除文件

您可以执行 DROP FILE 语句来删除文件。当您使用此语句来删除文件时，该文件将在前端（FE）内存和 Berkeley DB Java Edition（BDBJE）中均被删除。

## 语法

```SQL
DROP FILE "file_name" [FROM database]
[properties]
```

## 参数

| **参数**    | **必需** | **描述**                   |
| ----------- | -------- | -------------------------- |
| file_name   | 是       | 文件的名称。               |
| database    | 否       | 文件所属的数据库。         |
| properties  | 是       | 文件的属性。下表描述了属性的配置项。   |

**`properties`** **的配置项**

| **配置项** | **必需** | **描述**                |
| ---------- | -------- | ---------------------- |
| catalog    | 是       | 文件所属的类别。        |

## 示例

删除名为 **ca.pem** 的文件。

```SQL
DROP FILE "ca.pem" properties("catalog" = "kafka");
```