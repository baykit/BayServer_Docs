# インストール

BayServer のインストール方法を **言語実装ごと** にまとめます。BayServer は言語によって自然な配布形態が異なります:

- **Java / Go** … バイナリ配布（jar / 単一バイナリの tarball）
- **Ruby / Python / PHP / TypeScript** … 各言語のパッケージマネージャ（gem / pip / composer / npm）

使いたい言語版のセクションへ進んでください。起動・停止の操作は全言語共通で、末尾の [起動・停止（共通）](#起動停止共通) にまとめています。

## 動作環境

| 言語版 | 必要な処理系 | 既定の入手方法 |
|---|---|---|
| Java | JDK 1.8 以降 (OpenJDK 含む) | バイナリ (jar) |
| Go | 不要（ネイティブ単一バイナリ） | バイナリ (tar) |
| Ruby | Ruby 2.7.6 以降 | gem |
| Python | Python 3.7 以降 | pip |
| PHP | PHP 7.4 以降 | composer |
| TypeScript (Node.js) | Node.js v16.19.0 以降 | npm |

上記のバージョンは動作確認済みのものです。これより古くても動く可能性はありますが、サポート対象外です。

---

## Java

JDK さえあれば動作します。[配布ページ](https://baykit.yokohama/download/) から jar を取得し、展開して起動します。

```bash
jar xf BayServer_Java-X.Y.Z.jar
cd BayServer_Java-X.Y.Z
chmod +x bin/bayserver.sh        # Unix 系のみ初回 1 回
bin/bayserver.sh -start
```

展開して出来たディレクトリが **BayServer ホーム** です。

JVM オプションは環境変数 `BSERV_OPT` で渡せます:

```bash
BSERV_OPT="-Xmx2g -XX:+UseG1GC" bin/bayserver.sh -start
```

## Go

ネイティブビルドの単一バイナリ実装です。処理系のインストールは不要で、[配布ページ](https://baykit.yokohama/download/) からバイナリの tarball を取得し、展開してそのまま実行します。

```bash
tar zxf BayServer_Go-X.Y.Z.tgz
cd BayServer_Go-X.Y.Z
./bin/bayserver -start
```

展開して出来たディレクトリが **BayServer ホーム** です。

ソースからビルドする場合は Go ツールチェーンで:

```bash
go build ./cmd/bayserver
```

!!! note
    Go 版にパッケージマネージャ経由の配布はありません（配布バイナリ または ソースビルド）。

## Ruby

RubyGems から取得します。

```bash
gem install bayserver
mkdir bhome && cd bhome
bayserver -init        # BayServer ホームを初期化
bayserver              # 起動
```

`gem install` でコマンドを導入し、空ディレクトリを作って `-init` でホームを初期化、引数なしで起動します。

## Python

pip から取得します。

```bash
pip install bayserver
mkdir bhome && cd bhome
bayserver -init        # BayServer ホームを初期化
bayserver              # 起動
```

## PHP

Composer から取得します。実行ファイルは `vendor/bin/` 配下に入ります。

```bash
mkdir bhome && cd bhome
composer require baykit/bayserver
vendor/bin/bayserver -init     # BayServer ホームを初期化
vendor/bin/bayserver           # 起動
```

## TypeScript (Node.js)

npm からグローバルインストールします。

```bash
npm install -g @baykit/bayserver
bayserver -init        # BayServer ホームを初期化
bayserver              # 起動
```

---

## 起動・停止（共通）

インストール方法に関わらず、起動オプションと動作確認の手順は共通です。起動コマンド名だけが版によって異なります:

| 版 | 起動コマンド |
|---|---|
| Java | `bin/bayserver.sh`（BayServer ホーム内） |
| Go | `./bin/bayserver`（BayServer ホーム内） |
| Ruby / Python / TypeScript | `bayserver` |
| PHP | `vendor/bin/bayserver` |

以下では起動コマンドを `bayserver` と表記します。

### 起動オプション

- `-start` … 起動（省略可）
- `-daemon` … デーモンモード（コンソールから切り離して起動）
- `-init` … BayServer ホームの初期化（gem / pip / composer / npm でインストールした場合に使用）

```bash
bayserver -start -daemon
```

デーモンモードではコンソールにログが出ません。ログを見たい場合は `.plan` ファイルの Log Docker に `redirectFile` パラメータを設定し、ファイル出力に切り替えてください。

### 動作確認

BayServer はデフォルトで **2020 (HTTP)** と **2024 (HTTPS)** で待ち受けます。ブラウザで開いて画面が出れば成功です:

- `http://localhost:2020/`
- `https://localhost:2024/`

### 停止

起動したターミナルなら `Ctrl-C`。別のターミナルから停止する場合は:

```bash
bayserver -stop
```

---

## 次のステップ

- [Hello World](hello-world.md) — 最初の `.plan` を書いて自分のコンテンツを配信する
- [設計ファイル (.plan) リファレンス](../reference/plan-reference.md) — 設定ファイルの完全な書き方
- [言語別](../languages/index.md) — 各実装の機能対応
