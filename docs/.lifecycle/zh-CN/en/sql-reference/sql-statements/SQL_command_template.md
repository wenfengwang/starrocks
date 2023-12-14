---
displayed_sidebar: "Chinese"
---

# SQL语句模板

> *此模板以* `*ADMIN SET REPLICA STATUS*` *作为示例，说明编写SQL命令主题的要求。*

- > *在正文中使用大写的命令和关键词。例如，“SELECT语句用于查询满足特定条件的记录。","您可以使用GROUP BY在此列中对数据进行分组。","LIMIT关键字指定可返回的记录的最大数量”。*

- > *如果在正文中需要引用参数或参数值，请用两个反引号（``）括起来，例如，* `*cachesize*`* 。*

## ADMIN SET REPLICA STATUS

> *主题标题。使用命令的英文名称作为主题标题。将命令的所有字母大写。确保使用正确的拼写。*

### 描述

指定一个tablet的复制状态。此命令用于手动设置tablet的复制状态为`bad`或`ok`。

> *此命令的作用。您可以添加相关描述或使用说明。*

### 语法

```SQL
ADMIN SET REPLICA STATUS

PROPERTIES (“key” = “value”，...);
```

> *此命令的语法。将语法置于代码块中。确保语法符合编码规范。*

- > *使用适当的换行和缩进。*

- > *在代码中不要使用中文字符，例如中文分号或逗号。*

- > *将SQL命令中的关键字大写。例如：*

```SQL
SELECT ta.x, count(ta.y) AS y, sum(tb.z) AS z

FROM (

    SELECT a AS x, b AS y

    FROM t) ta

    JOIN tb

        ON ta.x = tb.x

WHERE tb.a > 10

GROUP BY ta.x

ORDER BY ta.x, z

LIMIT 10
```

### 参数

`PROPERTIES`：每个属性必须是一个键值对。支持的属性：

- `tablet_id`：tablet的ID。此参数为必选项。
- `backend_id`：tablet的BE ID。此参数为必选项。
- `status`：复制的状态。此参数为必选项。有效值：`bad`和`ok`。值`ok`表示系统将自动修复一个tablet的replicas。如果将复制状态设置为`bad`，replicas可能立即被删除。执行此操作时要小心。如果您指定的tablet不存在或复制状态为`bad`，系统将忽略这些replicas。

> *对命令中的参数进行描述。*

- > *首选参数描述应包括参数含义，值格式，值范围，是否需要此参数以及其他必要的说明。* 

- > *您可以使用无序列表来组织参数描述。如果描述复杂，可以将信息组织为表格。表格可以包括以下列：参数名、值类型（可选）、示例值（可选）、参数描述。

### 使用注意事项（可选）

> *您可以*添加一些使用此命令的注释或注意事项。

### 示例

示例1：将tablet 10003的复制状态设置为`bad`，位于BE 10001上。

```SQL
ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
```

示例2：将tablet 10003的复制状态设置为`ok`，位于BE 10001上。

```SQL
ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
```

- > *提供使用此命令的示例，并解释每个示例的目的。*

- > *可以提供多个示例。*

- > *如果需要在示例中描述多个场景，可以在代码片段中为每个场景添加注释，以帮助用户快速区分它们。*