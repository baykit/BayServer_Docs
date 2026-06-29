# FAQ

よくある質問とその対処。

## 一般

### BayServer はどんな用途に向きますか?

- Servlet / Rack / WSGI アプリケーションの **フロントエンド**
- Apache / Nginx の代わりに **シンプルな設定** で運用したい場合
- リバースプロキシ + 静的配信 + 動的アプリ実行を **1 プロセス** にまとめたい場合

詳細: [なぜ BayServer か](architecture/why-bayserver.md)。

### Apache / Nginx と何が違いますか?

設定の簡単さ、外部依存ゼロ、`.plan` の比喩 (港湾) による直感性。実装も Nginx と同等以上の RPS が出ます。詳細: [なぜ BayServer か](architecture/why-bayserver.md)。

### どの言語版を選ぶべき?

| アプリの言語 | 推奨 BayServer |
|---|---|
| Java (Servlet/JSP) | Java 版 |
| Ruby (Rails/Sinatra) | Ruby 版 |
| Python (Django/Flask) | Python 版 |
| PHP (WordPress 等) | PHP 版 |
| Node.js / TypeScript | TypeScript 版 |
| 言語不問の静的サイト or 単純プロキシ | 任意の版 (= Java / Go が軽量) |

詳細: [言語実装別 比較表](languages/comparison.md)。

### 設定ファイルの拡張子はなぜ `.plan`?

BayServer の世界観で「設計ファイル (= 港湾の設計書)」と呼んでいるため。ini ファイルに似た書式で、Python 風のインデントで Docker ブロックの階層を表現します。詳細: [設計ファイル (.plan) リファレンス](reference/plan-reference.md)。

## 起動・設定

### 起動したけどブラウザでアクセスできない

確認:

1. **プロセスが起動しているか** — `ps aux | grep bayserver` で確認
2. **ポートが正しいか** — `.plan` の `[port N]` でブラウザの URL のポート番号と一致
3. **ファイアウォール** — Linux なら `ufw` / `firewalld`、クラウド VPS ならセキュリティグループで該当ポート許可
4. **listen address** — 全 IF にバインドされているか (`ss -tln` で確認)

### `.plan` を書き換えたが反映されない

BayServer は起動時に `.plan` を読み込むため、変更後は再起動が必要です:

```bash
bin/bayserver.sh -stop
bin/bayserver.sh -start
```

### `.plan` の Docker 階層が分からない

[設計ファイル (.plan) リファレンス / Docker 階層](reference/plan-reference.md#docker-階層) を参照。基本は:

```
harbor
port
    secure
    permission
city
    town
        club
            docker xxx
log
```

### 同じ city を 2 つ書きたい

`[city *]` は **1 つの city** として扱われ、後の宣言が前を上書きします。複数の `city` が必要なら異なる識別子を:

```
[city www.example.com]
    [town /] ...
[city blog.example.com]
    [town /] ...
[city *]               # 上記にマッチしない場合の fallback
    [town /] ...
```

## トラブルシューティング

### `.php` ファイルが PHP として実行されず、ソースが表示される

Club Docker が `*.php` に紐づいていない、または `docker php` 行が抜けている:

```
[city *]
    [town /]
        location www/root
        [club *.php]
            docker php
```

PHP-FPM を使う場合は `docker fcgiWarp` で:

```
[club *.php]
    docker fcgiWarp
    destCity localhost
    destPort 9000
```

### `.cgi` が 403 / 500 になる

CGI Docker の場合:

1. CGI スクリプトに **実行権限** があるか (`chmod +x`)
2. **shebang** (`#!/usr/bin/perl` 等) が正しい
3. `[club *.cgi]` の `interpreter` パラメータ確認

### 502 Bad Gateway (= リバースプロキシ時)

上流サーバが動いていないか、`destCity` / `destPort` が間違っています。

```bash
# 上流の動作確認
curl http://<destCity>:<destPort>/
```

### HTTPS で「証明書が信頼できない」警告

オレオレ証明書なら正常。実運用では [Let's Encrypt](guide/letsencrypt.md) で取得したものを使う。

### 急にアクセスが遅くなった

考えられる原因:

- **アクセスログ大量出力** — `logLevel info` で `redirectFile` なし → ログが BayServer ログに溢れる
- **maxShips 枯渇** — 同時接続が多すぎ、`maxShips` を上げる
- **GC のフルストップ** (Java) — `-Xmx` 設定確認 / G1GC を使う
- **上流サーバが遅い** — リバースプロキシ時、上流の応答待ちで詰まる

詳細: [パフォーマンス Tips](performance/tips.md)。

### Permission Docker が効かない

`admit` / `refuse` の **順序** に注意:

```
# refuse all で全部弾く → 個別 admit
[permission]
    refuse all
    admit 192.168.1.0/24     # ← 効かない (= 既に refuse all が先にマッチ)
```

正しくは:

```
[permission]
    admit 192.168.1.0/24
    refuse all
```

詳細: [アクセス制限](guide/access-control.md)。

### アクセスログが出ない

- Log Docker が `.plan` に書かれているか確認
- ログファイルの **書込権限** があるディレクトリか
- デーモンモード (`-daemon`) で起動した場合は標準出力ではなくファイル出力に切り替える必要あり

### Java 版で OOM (OutOfMemoryError)

`-Xmx` を上げる:

```bash
BSERV_OPT="-Xmx4g" bin/bayserver.sh -start
```

それでも OOM するなら、`maxShips` を下げて同時接続を絞る。

---

## ここに無い質問は

- GitHub Issues — [baykit org のリポジトリ](https://github.com/baykit) の Issues に投稿
- ベイキット公式 — [https://baykit.yokohama/support/](https://baykit.yokohama/support/)
