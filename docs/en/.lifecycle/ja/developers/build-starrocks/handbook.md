---
displayed_sidebar: "Japanese"
---

# StarRocksのビルド方法

一般的には、以下のコマンドを実行するだけでStarRocksをビルドすることができます。

```
./build.sh
```

このコマンドは、まずすべてのサードパーティの依存関係が準備されているかどうかをチェックします。すべての依存関係が準備されている場合、StarRocksの`Backend`と`Frontend`をビルドします。

このコマンドが正常に実行されると、生成されたバイナリは`output`ディレクトリに保存されます。

## FE/BEを個別にビルドする方法

毎回FEとBEの両方をビルドする必要はありません。個別にビルドすることもできます。
例えば、BEのみをビルドする場合は次のようにします。

```
./build.sh --be
```

そして、FEのみをビルドする場合は次のようにします。

```
./build.sh --fe
```

# ユニットテストの実行方法

BEとFEのユニットテストは別々に実行することができます。一般的には、BEのテストは次のコマンドで実行します。

```
./run-be-ut.sh
```

FEのテストは次のコマンドで実行します。

```
./run-fe-ut.sh
```

## コマンドラインでBEのUTを実行する方法

現在、BEのUTはいくつかの依存関係が必要であり、`./run-be-ut.sh`がそれをサポートしています。しかし、柔軟性に欠ける場合もあります。コマンドラインでUTを実行したい場合は、次のコマンドを実行します。

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks Backend UTはgoogle-testの上に構築されているため、フィルタを渡して一部のUTのみを実行することができます。例えば、MapColumnに関連するテストのみを実行したい場合は、次のコマンドを実行します。

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```

# ビルドオプション

## clangでビルドする方法

`clang`を使用してStarRocksをビルドすることもできます。

```
CC=clang CXX=clang++ ./build.sh --be
```

ビルドメッセージには、次のような類似のメッセージが表示されます。

```
-- compiler Clang version 14.0.0
```

## 異なるリンカーでビルドする方法

デフォルトのリンカーは遅いため、開発者は異なるリンカーを指定してリンク速度を向上させることができます。
例えば、LLVMベースのリンカーである`lld`を使用することができます。

まず、`lld`をインストールする必要があります。

```
sudo apt install lld
```

次に、環境変数STARROCKS_LINKERを使用したいリンカーで設定します。
例えば：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 異なるタイプでビルドする方法

異なるBUILD_TYPE変数を使用して、異なるタイプでStarRocksをビルドすることができます。デフォルトのBUILD_TYPEは`RELEASE`です。例えば、`ASAN`タイプでStarRocksをビルドする場合は次のようにします。

```
BUILD_TYPE=ASAN ./build.sh --be
```
