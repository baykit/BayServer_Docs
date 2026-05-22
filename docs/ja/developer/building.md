# ビルド方法

BayServer をソースからビルドしたい場合の手順。各言語版で違いがあります。

## 共通: ソース取得

GitHub にホストされています:

```bash
git clone https://github.com/baykit/BayServer_Java.git    # Java 版
git clone https://github.com/baykit/BayServer_Ruby.git    # Ruby 版
git clone https://github.com/baykit/BayServer_Python.git  # Python 版
git clone https://github.com/baykit/BayServer_PHP.git     # PHP 版
git clone https://github.com/baykit/BayServer_TypeScript.git
git clone https://github.com/baykit/BayServer_Go.git
git clone https://github.com/baykit/BayServer_C.git
```

## Java 版

```bash
cd BayServer_Java
bash build.sh
```

`build.sh` は内部で:

1. 各モジュールの `pom.xml.template` から `pom.xml` を envsubst で生成
2. `mvn package` で全モジュールをビルド
3. `dist/` 配下に配布可能な tarball を作成

ビルドに必要なもの:

- **JDK 1.8 以降** (= 機能によっては 21 以降推奨)
- **Maven 3.x**
- **HTTP/3** を有効化するなら追加で **`libcroute`** ([HTTP/3 ガイド](../guide/http3.md))

開発時に特定モジュールだけ build したい場合:

```bash
cd modules/bayserver-core
mvn package -DskipTests
```

## Ruby 版

```bash
cd BayServer_Ruby
gem build bayserver.gemspec
gem install bayserver-*.gem
```

または bundler を使う場合:

```bash
bundle install
bundle exec rake build
```

## Python 版

```bash
cd BayServer_Python
pip install build
python -m build
pip install dist/bayserver-*.whl
```

## PHP 版

```bash
cd BayServer_PHP
composer install --no-dev
```

`composer install` した時点で動作可能になります (= ビルドステップ無し)。

## TypeScript 版

```bash
cd BayServer_TypeScript
npm install
npm run build
```

`tsc` で `.ts` → `.js` にトランスパイル。配布は `npm publish` で。

## C 版

```bash
cd BayServer_C
make
```

`Makefile` ベース。出力は `bin/bayserver`。クロスコンパイルする場合は `CC=<toolchain>` を指定。

## CI

各リポジトリの `.github/workflows/` に GitHub Actions の build & test が定義されています。PR 時に自動で回るので、ローカルでビルドが通れば PR 出して OK。

---

## 関連

- [コーディングスタイル](coding-style.md)
- [新しい Docker を書く](adding-a-docker.md)
- [内部実装の地図](internals.md)
