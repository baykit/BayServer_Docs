# PHP 版 BayServer

BayServer for PHP は PHP 7.4 以降で動作します。PHP アプリケーション (= WordPress / Laravel / 等) を直接実行できます。

## インストール

```bash
mkdir bhome && cd bhome
composer require baykit/bayserver
vendor/bin/bayserver -init
vendor/bin/bayserver
```

より詳しい手順・起動オプションは [インストール](../getting-started/install.md#php) を参照。

## PHP ファイルを動かす

`.plan` で `*.php` の Club Docker に `php` ハンドラを指定:

```
[city *]
    [town /]
        location www/root
        index    index.php
        [club *.php]
            docker php
```

`www/root/index.php` などにスクリプトを置けば、ブラウザから実行できます。

## WordPress を動かす

WordPress の場合は専用の WordPress Docker を使うと URL リライト規則が自動適用されます:

```
[city wp.example.com]
    [town /]
        location /srv/wordpress
        [club *]
            docker wordpress
```

詳細: [WordPress を動かす](../guide/wordpress.md)。

## マルチコア / シングルコアモード

PHP 版は **Windows プラットフォームでは multiCore off** が必須 (= Cygwin / MinGW では動く可能性あり)。Linux / macOS では multiCore on で問題なく動作します。

```
[harbor]
    multiCore on    # Linux / macOS
    grandAgents 4
```

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| `php: command not found` | PHP の installation を確認 (`php --version`) |
| `.php` が PHP 実行されず表示される | Club Docker の `*.php` ルール確認、`docker php` の行があるか |

---

## 関連

- [インストール](../getting-started/install.md)
- [言語実装別 比較表](comparison.md)
- [WordPress を動かす](../guide/wordpress.md)
