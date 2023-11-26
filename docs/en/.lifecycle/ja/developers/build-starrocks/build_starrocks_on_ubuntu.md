---
displayed_sidebar: "Japanese"
---

x86_64およびaarch64でのビルドをサポートします

### 必要条件

```
sudo apt-get update
```

```
sudo apt-get install automake binutils-dev bison byacc ccache flex libiberty-dev libtool maven zip python3 python-is-python3 -y
```

### コンパイラ

Ubuntuのバージョンが22.04以上の場合、以下のコマンドを実行してください
```
sudo apt-get install cmake gcc g++ default-jdk -y
```

Ubuntuのバージョンが22.04未満の場合、以下のツールとコンパイラのバージョンを確認してください

##### 1. GCC/G++

GCC/G++のバージョンは10.3以上である必要があります
```
gcc --version
g++ --version
```
GCC/G++をインストールしてください(https://gcc.gnu.org/releases.html)

##### 2. JDK

OpenJDKのバージョンは8以上である必要があります
```
java --version
```
OpenJDKをインストールしてください(https://openjdk.org/install)

##### 3. CMake

CMakeのバージョンは3.20.1以上である必要があります

```
cmake --version
```
CMakeをインストールしてください(https://cmake.org/download)


### コンパイル速度の向上

デフォルトのコンパイル並列度は**CPUコア数 / 4**です。
コンパイル速度を向上させたい場合は、並列度を改善することができます。

1. 32個のCPUコアを持っている場合、デフォルトの並列度は8です。

```
./build.sh
```

2. 32個のCPUコアを持っている場合、24個のコアを使用してコンパイルしたい場合。

```
./build.sh -j 24
```

### よくある質問

1. Ubuntu 20.04で`aws_cpp_sdk`のビルドに失敗しました。
```
Error: undefined reference to pthread_create
```
このエラーは、CMakeのバージョンが古いために発生します。CMakeのバージョンを少なくとも3.20.1にアップグレードしてください。
