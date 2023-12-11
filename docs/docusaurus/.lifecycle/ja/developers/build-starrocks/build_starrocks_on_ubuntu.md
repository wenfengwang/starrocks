---
displayed_sidebar: "Japanese"
---

x86_64およびaarch64でのサポートを構築

### 前提条件

```
sudo apt-get update
```

```
sudo apt-get install automake binutils-dev bison byacc ccache flex libiberty-dev libtool maven zip python3 python-is-python3 -y
```

### コンパイラー

Ubuntuのバージョンが22.04以上の場合、以下を実行できます
```
sudo apt-get install cmake gcc g++ default-jdk -y
```

Ubuntuのバージョンが22.04よりも低い場合。
以下のツールとコンパイラーのバージョンを確認してください

##### 1. GCC/G++

GCC/G++のバージョンは10.3以上である必要があります
```
gcc --version
g++ --version
```
GCC/G++をインストールする(https://gcc.gnu.org/releases.html)

##### 2. JDK

OpenJDKのバージョンは8以上である必要があります
```
java --version
```
OpenJdkをインストールする(https://openjdk.org/install)

##### 3. CMake

cmakeのバージョンは3.20.1以上である必要があります

```
cmake --version
```
CMakeをインストールする(https://cmake.org/download)


### コンパイル速度の向上

デフォルトのコンパイル並列性は **CPUコア数 / 4** に等しいです。
コンパイル速度を向上させたい場合、並列性を改善できます。

1. 32のCPUコアを持っている場合、デフォルトの並列性は8です。

```
./build.sh
```

2. 32のCPUコアを持っている場合、24のコアを使用してコンパイルしたい場合。

```
./build.sh -j 24
```

### よくある質問

1. Ubuntu 20.04で `aws_cpp_sdk`をビルドできませんでした。
```
Error: undefined reference to pthread_create
```
このエラーは、より低いCMakeバージョンから発生します； CMakeバージョンを3.20.1以上にアップグレードできます