---
displayed_sidebar: Chinese
---

# イベント

`events` は Event Manager のイベントに関する情報を提供します。

`events` は以下のフィールドを提供します：

| フィールド           | 説明                                                         |
| -------------------- | ------------------------------------------------------------ |
| EVENT_CATALOG        | イベントが属するカタログの名前。この値は常に `def` です。    |
| EVENT_SCHEMA         | イベントが属するスキーマ（データベース）の名前。             |
| EVENT_NAME           | イベントの名前。                                             |
| DEFINER              | DEFINER 句で指定されたユーザー（通常はイベントを作成したユーザー）。 |
| TIME_ZONE            | イベントのタイムゾーンで、イベントのスケジューリングと実行時に適用されます。デフォルト値は `SYSTEM` です。 |
| EVENT_BODY           | イベントの DO 句で使用される言語。この値は常に `SQL` です。  |
| EVENT_DEFINITION     | イベントの DO 句で構成される SQL ステートメントのテキスト；つまり、このイベントが実行するステートメント。 |
| EVENT_TYPE           | イベントの繰り返しタイプで、`ONE TIME`（一回限り）または `RECURRING`（繰り返し）のいずれかです。 |
| EXECUTE_AT           | 一回限りのイベントについては、CREATE EVENT ステートメントの AT 句で指定された DATETIME 値、または最後にイベントを変更した ALTER EVENT ステートメントで指定された DATETIME 値です。この列に表示される値は、イベントの AT 句に含まれる任意の INTERVAL 値の加算または減算を反映しています。例えば、`CURRENT_DATETIME + INTERVAL '1:6' DAY_HOUR` でイベントを作成し、そのイベントが 2018-02-09 14:05:30 に作成された場合、この列に表示される値は '2018-02-10 20:05:30' になります。イベントのタイミングが EVERY 句によって決定される場合（つまり、イベントが繰り返しの場合）、この列の値は NULL です。 |
| INTERVAL_VALUE       | 繰り返しイベントの場合、イベントの実行間の待機間隔数。一回限りのイベントの場合、この値は常に NULL です。 |
| INTERVAL_FIELD       | 繰り返しイベントが次の繰り返しの前に待機する間隔の単位。一回限りのイベントの場合、この値は常に NULL です。 |
| SQL_MODE             | イベントが作成または変更された時の有効な SQL モード、およびイベント実行時に使用される SQL モード。 |
| STARTS               | 繰り返しイベントの開始日時。DATETIME 値として表示され、イベントに開始日時が定義されていない場合は NULL です。一回限りのイベントの場合、この列は常に NULL です。STARTS 句を含む繰り返しイベントが定義されている場合、この列には対応する DATETIME 値が含まれます。EXECUTE_AT 列と同様に、使用された任意の式が解析された値です。STARTS 句がイベントのタイミングに影響を与えない場合、この列は NULL です。 |
| ENDS                 | ENDS 句を含む繰り返しイベントが定義されている場合、この列には対応する DATETIME 値が含まれます。EXECUTE_AT 列と同様に、使用された任意の式が解析された値です。ENDS 句がイベントのタイミングに影響を与えない場合、この列は NULL です。 |
| STATUS               | イベントの状態。`ENABLED`、`DISABLED`、または `SLAVESIDE_DISABLED` のいずれかです。`SLAVESIDE_DISABLED` はイベントがレプリケーションのソースとして機能する別の MySQL サーバーで作成され、現在の MySQL サーバーにレプリケートされたが、現在はレプリカ上で実行されていないことを意味します。 |
| ON_COMPLETION        | 有効な値：`PRESERVE` と `NOT PRESERVE`。                   |
| CREATED              | イベントが作成された日時。これは DATETIME 値です。         |
| LAST_ALTERED         | イベントが最後に変更された日時。これは DATETIME 値です。イベントが作成されてから変更されていない場合、この値は CREATED 値と同じです。 |
| LAST_EXECUTED        | イベントが最後に実行された日時。これは DATETIME 値です。イベントが一度も実行されていない場合、この列は NULL です。LAST_EXECUTED はイベントが実行を開始した時刻を示します。したがって、ENDS 列は LAST_EXECUTED よりも小さくなることはありません。 |
| EVENT_COMMENT        | イベントにコメントがある場合のコメントテキスト。コメントがない場合、この値は空です。 |
| ORIGINATOR           | イベントを作成した時の MySQL サーバーのサーバー ID；レプリケーションに使用されます。レプリケーションのソースで実行された場合、ALTER EVENT はこの値をそのステートメントが発生したサーバーのサーバー ID に更新することができます。デフォルト値は 0 です。 |
| CHARACTER_SET_CLIENT | イベントを作成した時の character_set_client システム変数のセッション値。 |
| COLLATION_CONNECTION | イベントを作成した時の collation_connection システム変数のセッション値。 |
| DATABASE_COLLATION   | イベントに関連付けられたデータベースの照合順序。           |
