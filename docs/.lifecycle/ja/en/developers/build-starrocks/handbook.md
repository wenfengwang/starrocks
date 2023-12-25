---
displayed_sidebar: English
---

# StarRocksのビルド方法

一般的に、次のコマンドを実行するだけでStarRocksをビルドできます。

```
./build.sh
```

このコマンドは最初に、すべてのサードパーティの依存関係が準備できているかどうかをチェックします。依存関係がすべて準備できていれば、StarRocksの`Backend`と`Frontend`をビルドします。

このコマンドが正常に実行されると、生成されたバイナリファイルは`output`ディレクトリに配置されます。

## FE/BEを別々にビルドする

FEとBEを毎回ビルドする必要はありません。別々にビルドすることができます。
例えば、BEだけをビルドするには
```
./build.sh --be
```

そして、FEだけをビルドするには
```
./build.sh --fe
```

# ユニットテストの実行方法

BEとFEのユニットテストは別々に行われます。一般的に、BEのテストは以下のコマンドで実行できます。
```
./run-be-ut.sh
```

FEのテストを実行するには
```
./run-fe-ut.sh
```

## コマンドラインでBEのユニットテストを実行する方法

現在、BEのユニットテストを実行するにはいくつかの依存関係が必要で、`./run-be-ut.sh`はそれを補助します。しかし、これは十分に柔軟ではありません。コマンドラインでユニットテストを実行したい場合は、以下を実行します。

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test
```

StarRocks BackendのユニットテストはGoogle Testをベースに構築されているので、特定のテストだけを実行するためのフィルターを指定できます。例えば、MapColumnに関連するテストのみを実行したい場合は、以下を実行します。

```
UDF_RUNTIME_DIR=./ STARROCKS_HOME=./ LD_LIBRARY_PATH=/usr/lib/jvm/java-18-openjdk-amd64/lib/server ./be/ut_build_ASAN/test/starrocks_test --gtest_filter="*MapColumn*"
```


# ビルドオプション

## Clangでビルドする

`clang`を使用してStarRocksをビルドすることもできます。

```
CC=clang CXX=clang++ ./build.sh --be
```

すると、ビルドメッセージに以下のようなメッセージが表示されます。

```
-- compiler Clang version 14.0.0
```

## 異なるリンカーでビルドする

デフォルトのリンカーは遅いため、開発者はリンク速度を上げるために異なるリンカーを指定できます。
例えば、LLVMベースのリンカーである`lld`を使用することができます。

まず、`lld`をインストールする必要があります。

```
sudo apt install lld
```

その後、使用したいリンカーでSTARROCKS_LINKER環境変数を設定します。
例えば：

```
STARROCKS_LINKER=lld ./build.sh --be
```

## 異なるタイプでビルドする

異なるBUILD_TYPE変数を使用してStarRocksをビルドすることができ、デフォルトのBUILD_TYPEは`RELEASE`です。例えば、`ASAN`タイプでStarRocksをビルドするには
```
BUILD_TYPE=ASAN ./build.sh --be
```
