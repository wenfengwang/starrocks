---
displayed_sidebar: English
---

# SQL 语句模板

> *此模板以* `*ADMIN SET REPLICA STATUS*` *作为示例，说明编写 SQL 命令主题的要求。*

- > *在运行文本中将命令和关键字大写。例如，“SELECT语句用于查询满足特定条件的记录”、“您可以使用GROUP BY对该列中的数据进行分组”、“LIMIT关键字指定可返回的最大记录数”。*

- > *如果需要在运行文本中引用参数或参数值，请用两个反引号（``）括起来，例如* `*cachesize*`*。*

## ADMIN SET REPLICA STATUS

> *主题标题。使用英文命令名称作为主题标题。将命令名称中的所有字母大写。请确保使用正确的拼写。*

### 描述

指定平板电脑的副本状态。此命令用于手动设置平板电脑的副本状态为 `bad` 或 `ok`。

> *此命令的作用。您可以添加相关描述或使用说明。*

### 语法

```SQL
ADMIN SET REPLICA STATUS

PROPERTIES ("key" = "value", ...);
```

> *此命令的语法。将语法放在代码块中。确保语法符合编码规范。*

- > *使用适当的换行和缩进。*

- > *不要在代码中使用中文字符，如中文分号或逗号。*

- > *将 SQL 命令中的关键字大写。例如：*

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

`PROPERTIES`：每个属性必须是键值对。支持的属性：

- `tablet_id`：平板电脑的 ID。此参数是必需的。
- `backend_id`：平板电脑的 BE ID。此参数是必需的。
- `status`：副本的状态。此参数是必需的。有效值： `bad` 和 `ok`。值 `ok` 表示系统会自动修复平板电脑的副本。如果副本状态设置为 `bad`，则可能会立即删除副本。执行此操作时要小心。如果指定的平板电脑不存在或副本状态为 `bad`，系统将忽略这些副本。

> *命令中的参数说明。*

- > *首选参数描述必须包括参数含义、值格式、值范围、此参数是否为必填项，以及其他必要的备注。*

- > *您可以使用无序列表来组织参数描述。如果描述较复杂，可以将信息组织为表格。表格可以包括以下列：参数名称、值类型（可选）、示例值（可选）、参数描述。*

### 使用说明（可选）

> *您可以* *添加一些使用此命令的注释或注意事项。*

### 例子

示例 1：将 BE 10001 上的平板电脑 10003 的副本状态设置为 `bad`。

```SQL
ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");
```

示例 2：将 BE 10001 上的平板电脑 10003 的副本状态设置为 `ok`。

```SQL
ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "ok");
```

- > *提供使用此命令的示例，并解释每个示例的目的。*

- > *您可以提供多个示例。*

- > *如果需要在示例中描述多个场景，请在代码片段中为每个场景添加注释，以帮助用户快速区分它们。*