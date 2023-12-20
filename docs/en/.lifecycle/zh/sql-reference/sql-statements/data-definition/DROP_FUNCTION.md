---
displayed_sidebar: English
---

# 删除函数

## 描述

删除自定义函数。只有当函数的名称和参数类型一致时，才能删除该函数。

只有自定义函数的所有者才有权限删除该函数。

### 语法

```sql
DROP FUNCTION function_name(arg_type [, ...])
```

### 参数

`function_name`：要删除的函数的名称。

`arg_type`：要删除的函数的参数类型。

## 示例

1. 删除一个函数。

   ```sql
   DROP FUNCTION my_add(INT, INT)
   ```