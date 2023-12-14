---
displayed_sidebar: "中文"
---

# 取消下线

## 功能

该语句用于撤销一个节点下线操作。

> **注意**
>
> 仅管理员使用！

## 语法

```sql
CANCEL DECOMMISSION BACKEND "主机:心跳端口"[,"主机:心跳端口"...]
```

## 示例

取消两个节点的下线操作:

```sql
CANCEL DECOMMISSION BACKEND "主机1:端口", "主机2:端口";
```