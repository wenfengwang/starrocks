---
displayed_sidebar: "Japanese"
---

# SHOW ROUTINE LOAD（ルーチンのロードを表示する）

## 例

1. 名前がtest1のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブも含む）を表示します。結果は1つ以上の行です。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. 名前がtest1の現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. example_db内のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブも含む）を表示します。結果は1つ以上の行です。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. example_db内のすべての実行中のルーチンインポートジョブを表示します。

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. example_db内の名前がtest1の現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. example_db内の名前がtest1のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブも含む）を表示します。結果は1つ以上の行です。

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```
