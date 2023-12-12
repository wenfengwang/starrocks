```yaml
---
displayed_sidebar: "Japanese"
---

# ブローカを表示

## 説明

このステートメントは、現在存在するブローカを表示するために使用されます。

構文:

```sql
SHOW BROKER
```

ノート:

1. LastStartTime は、最新のBE起動時間を示します。
2. LastHeartbeat は、最新のハートビートを示します。
3. Alive は、ノードが生存しているかどうかを示します。
4. ErrMsg は、ハートビートが失敗した場合にエラーメッセージを表示するために使用されます。
```