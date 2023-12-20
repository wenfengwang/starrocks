---
displayed_sidebar: English
---

# 展示常规加载任务

## 示例

1. 展示所有名为 test1 的常规导入任务（包括已停止或已取消的任务）。结果可能为一行或多行。

   ```sql
   SHOW ALL ROUTINE LOAD FOR test1;
   ```

2. 展示当前正在运行的名为 test1 的常规导入任务。

   ```sql
   SHOW ROUTINE LOAD FOR test1;
   ```

3. 展示 example_db 中的所有常规导入任务（包括已停止或已取消的任务）。结果可能为一行或多行。

   ```sql
   use example_db;
   SHOW ALL ROUTINE LOAD;
   ```

4. 展示 example_db 中所有正在运行的常规导入任务。

   ```sql
   use example_db;
   SHOW ROUTINE LOAD;
   ```

5. 展示 example_db 中当前正在运行的名为 test1 的常规导入任务。

   ```sql
   SHOW ROUTINE LOAD FOR example_db.test1;
   ```

6. 展示名为 test1 的所有常规导入任务（包括已停止或已取消的任务），结果为 example_db 中的一行或多行。

   ```sql
   SHOW ALL ROUTINE LOAD FOR example_db.test1;
   ```
