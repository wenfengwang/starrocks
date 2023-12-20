---
displayed_sidebar: English
---

# 删除用户

## 说明

删除指定的用户账号。

## 语法

```sql
 DROP USER '<user_identity>'

`user_identity`:

 user@'host'
user@['domain']
```

## 示例

删除用户 jack@'192.%'。

```sql
DROP USER 'jack'@'192.%'
```
