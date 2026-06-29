# インストール (Version 3 以前)

!!! warning "凍結ページ — Version 3 以前専用"
    このページは **BayServer Version 3 以前** のインストール手順（ダウンロード版 / パッケージ版の二分法）です。**今後の編集は行いません**。Version 4 以降のインストールは [インストール](../getting-started/install.md) を参照してください。

BayServer は **ダウンロード版** と **パッケージ版** の 2 通りで配布されています。どちらも同じ機能を提供しますが、配布形態と更新方法が違います。

## 動作環境

| 版 | 必要な処理系 |
|---|---|
| BayServer for Java | JDK 1.8 以降 (OpenJDK 含む) |
| BayServer for Ruby | Ruby 2.7.6 以降 |
| BayServer for Python | Python 3.7 以降 |
| BayServer for PHP | PHP 7.4 以降 |
| BayServer for TypeScript (Node.js) | Node.js v16.19.0 以降 |
| BayServer for Go | バイナリ配布 |

上記のバージョンは動作確認済みのものです。これより古いバージョンでも動く可能性はありますが、サポート対象外です。

## ダウンロード版 vs パッケージ版

| 版 | ダウンロード版 | パッケージ版 |
|---|:---:|:---:|
| Java | ✓ | △ (※) |
| Ruby | ✓ | ✓ (gem) |
| Python | ✓ | ✓ (pip) |
| PHP | ✓ | ✓ (composer) |
| TypeScript | ✓ | ✓ (npm) |
| Go | ✓ | ✕ |

※ Java のパッケージ版は当時利用していた Maven リポジトリのサービス終了に伴い、現在は 2.x のみ取得可能。

パッケージ版の提供は **2.2.0 以降** です。

## ダウンロード版

どの言語の版でもインストール手順は基本的に同じです。

### ダウンロードと展開

[配布ページ](https://baykit.yokohama/download/) からアーカイブを取得し展開します。

=== "Java"

    ```bash
    jar xf BayServer_Java-X.Y.Z.jar
    ```

=== "Ruby / Python / PHP / TypeScript"

    ```bash
    tar zxf BayServer_<Lang>-X.Y.Z.tgz
    ```

展開して出来たディレクトリが **「BayServer ホーム」** です。以降の起動コマンドはすべてこのディレクトリを基準に実行します。

### 起動

```bash
cd BayServer_<Lang>-X.Y.Z
chmod +x bin/bayserver.sh        # Unix 系のみ初回 1 回
bin/bayserver.sh -start
```

Windows では:

```
bin\bayserver -start
```

`-start` は省略可能。`-daemon` を付けるとデーモンモードで起動 (= コンソールから切り離して動く)。

```bash
bin/bayserver.sh -start -daemon
```

デーモンモードではコンソールにログが出ないため、ログを見たい場合は `.plan` ファイルの Log Docker に `redirectFile` パラメータを設定してファイル出力に切り替えてください。

### 動作確認

BayServer はデフォルトで **2020 (HTTP)** と **2024 (HTTPS)** で待ち受けます。

ブラウザで開いて画面が出れば成功:

- `http://localhost:2020/`
- `https://localhost:2024/`

### 停止

起動したターミナルなら `Ctrl-C`。別のターミナルから停止する場合は:

```bash
bin/bayserver.sh -stop
```

## パッケージ版

各言語の標準パッケージマネージャから取得します。

### Maven (Java)

BayServer ホーム用のディレクトリを作って `pom.xml` を配置:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>testGroup</groupId>
  <artifactId>testArtifact</artifactId>
  <version>1.0</version>
  <dependencies>
    <dependency>
      <groupId>yokohama.baykit</groupId>
      <artifactId>bayserver</artifactId>
      <version>2.2.0</version>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>3.0.0</version>
      </plugin>
    </plugins>
  </build>
</project>
```

```bash
mvn exec:java -Dexec.mainClass="yokohama.baykit.bayserver.BayServer" -Dexec.args="-init"
mvn exec:java -Dexec.mainClass="yokohama.baykit.bayserver.BayServer"
```

`-init` でホーム初期化、引数なしで起動です。

### Gem (Ruby)

```bash
gem install bayserver
mkdir bhome && cd bhome
bayserver -init
bayserver
```

### pip (Python)

```bash
pip install bayserver
mkdir bhome && cd bhome
bayserver -init
bayserver
```

### Composer (PHP)

```bash
mkdir bhome && cd bhome
composer require baykit/bayserver
bayserver -init       # vendor/bin/bayserver
bayserver
```

### npm (TypeScript / Node.js)

```bash
npm install -g @baykit/bayserver
bayserver -init
bayserver
```

## ダウンロード版とパッケージ版の違い

| 観点 | ダウンロード版 | パッケージ版 |
|---|---|---|
| 配布 | 1 ファイル (tar/jar) | 言語標準のレジストリ経由 |
| 更新 | 再ダウンロード | パッケージマネージャで `update` |
| ホームディレクトリ | 展開された場所そのもの | 別途作って `-init` で初期化 |
| 依存解決 | 自己完結 | 標準的な依存ツリーに乗る |
| 推奨用途 | スタンドアロン運用、検証 | アプリの一部に組込む場合 |

---

## 次のステップ

- [Hello World](../getting-started/hello-world.md) — 最初の `.plan` を書いて自分のコンテンツを配信する
- [`.plan` 文法](../reference/index.md) — 設定ファイルの完全な書き方
- [言語別](../languages/index.md) — 各実装の機能対応
