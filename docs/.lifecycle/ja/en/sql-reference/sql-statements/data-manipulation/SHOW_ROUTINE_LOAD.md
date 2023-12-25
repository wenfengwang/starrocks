---
displayed_sidebar: English
---

# ROUTINE LOADの表示

## 例

1. 名前がtest1のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。結果は1行以上です。

    ```sql
    SHOW ALL ROUTINE LOAD FOR test1;
    ```

2. 名前がtest1の現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR test1;
    ```

3. example_dbにおけるすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。結果は1行以上です。

    ```sql
    use example_db;
    SHOW ALL ROUTINE LOAD;
    ```

4. example_dbにおける実行中のすべてのルーチンインポートジョブを表示します。

    ```sql
    use example_db;
    SHOW ROUTINE LOAD;
    ```

5. example_dbにおける名前がtest1の現在実行中のルーチンインポートジョブを表示します。

    ```sql
    SHOW ROUTINE LOAD FOR example_db.test1;
    ```

6. example_dbにおける名前がtest1のすべてのルーチンインポートジョブ（停止またはキャンセルされたジョブを含む）を表示します。結果は1行以上です。

    ```sql
    SHOW ALL ROUTINE LOAD FOR example_db.test1;
    ```
