# PHP を使う

BayServer で PHP スクリプトを動かす方法は大きく **2 通り** あります:

| 方法 | 特徴 | 対応する Docker |
|---|---|---|
| **PHP-CGI を使う** | `php-cgi` バイナリを呼び出して `.php` を実行。最小構成、外部プロセス不要 | `docker php` (= PhpCgi Docker) |
| **PHP-FPM 経由** | 永続プロセスプール (= PHP-FPM) に FCGI でリクエストを転送。本番向け | `docker fcgiWarp` ([詳細](php-fpm.md)) |

このページでは **(1) PHP-CGI を使う** 方法を扱います。PHP-FPM 経由は [php-fpm を使う](php-fpm.md) を参照してください。

## 前提

- `php-cgi` バイナリが `PATH` 上にある (= `which php-cgi` で確認)
- PHP 7.4 以降

`php-cgi` が無い場合:

```bash
sudo apt install php-cgi          # Debian / Ubuntu
sudo dnf install php-cgi          # RHEL / Fedora
```

## 最小構成

```
[city *]
    [town /]
        location www/root
        index    index.php
        [club *.php]
            docker php
```

- `[club *.php]` で `.php` ファイルへのリクエストを **PhpCgi Docker** が処理
- リクエストごとに `php-cgi` プロセスが起動 → スクリプト実行 → 結果を返却

## 動作確認

```bash
cat > www/root/info.php <<'EOF'
<?php phpinfo(); ?>
EOF

bin/bayserver.sh -restart
curl http://localhost:8080/info.php
```

`phpinfo()` の出力が返ってくれば成功。

## パラメータ

| パラメータ | 意味 | デフォルト |
|---|---|---|
| `phpCgi` | `php-cgi` のフルパス | `php-cgi` (= PATH 検索) |
| `timeout` | PHP プロセスのタイムアウト (秒) | (実装依存) |

カスタム例:

```
[club *.php]
    docker php
    phpCgi /usr/local/bin/php-cgi
    timeout 30
```

## いつ PhpCgi を使い、いつ PHP-FPM 経由にするか

| 観点 | PhpCgi (= `docker php`) | PHP-FPM 経由 (= `docker fcgiWarp`) |
|---|---|---|
| プロセス起動コスト | 毎リクエスト fork+exec → **遅い** | プール済みプロセスを使い回し → 速い |
| メモリ使用 | リクエスト時のみ占有 | 常時占有 |
| opcache 効果 | 起動毎にリセット → ほぼ無効 | プロセス間で永続 → 効果大 |
| 設定の複雑さ | `.plan` だけ | + PHP-FPM 設定が必要 |
| 推奨用途 | 開発・テスト、低 RPS | **本番、高 RPS** |

開発時は PhpCgi で OK、本番は **PHP-FPM 経由を強く推奨** ([php-fpm ガイド](php-fpm.md))。

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| 「php-cgi: not found」 | `which php-cgi` で実体確認、必要なら `phpCgi` パラメータでフルパス指定 |
| 「No input file specified」 | `town` の `location` 設定確認 (= 物理ファイルパスとリクエスト URL の対応) |
| 文字化け | `harbor` の `charset UTF-8` 設定、PHP 側の `default_charset` 設定 |
| 動作が遅い | PHP-FPM に切替 ([php-fpm ガイド](php-fpm.md)) |

---

## 関連

- [php-fpm を使う](php-fpm.md) — 本番運用向け
- [WordPress を動かす](wordpress.md) — PHP + WordPress 専用 Docker
- [WordPress 完全構成 (実例)](../examples/wordpress-full.md)
- [PHP 版 BayServer](../languages/php.md)
- [Docker 種別 / PHP Docker](../reference/docker-types.md)
