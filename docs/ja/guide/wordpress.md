# WordPress を動かす

BayServer は WordPress 専用の **WordPress Docker** を持っています。WP の URL リライト規則を自動適用してくれるため、`.htaccess` 相当の設定を BayServer に書く必要がありません。

## 前提

- BayServer (PHP 版か、Java 版 + FCGI Warp で PHP-FPM 経由のいずれか)
- WordPress 本体 (= `wp-content/` 含む一式) をダウンロード済み
- MySQL / MariaDB 等のデータベース

## PHP 版 BayServer で動かす

PHP 版なら最もシンプル。WordPress Docker (= 内部で PHP Docker + URL リライト) を使います:

```
[city wp.example.com]
    [town /]
        location /srv/wordpress
        [club *]
            docker wordpress
```

`/srv/wordpress` に WordPress 本体を展開しておきます (= `index.php`, `wp-config.php`, `wp-content/` などがある)。

### WordPress Docker の効果

通常 WordPress では `.htaccess` で:

```
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
```

のようなリライト規則を書きますが、WordPress Docker はこれと同等の処理を内蔵しているため、追加の Reroute Docker は不要です。

## Java 版 BayServer + PHP-FPM で動かす

Java 版で PHP を直接動かせないため、PHP-FPM をバックエンドにして FCGI で繋ぎます。

### 構成

```
ブラウザ → BayServer for Java (HTTPS) → PHP-FPM (FCGI :9000) → WordPress
```

### `.plan` 設定

```
[city wp.example.com]
    [town /]
        location /srv/wordpress

        [reroute]
            docker wordpress    # URL リライトだけ適用

        [club *.php]
            docker fcgiWarp
            destCity localhost
            destPort 9000
```

`reroute wordpress` で URL リライト規則を適用、`*.php` のリクエストは PHP-FPM に転送します。

### PHP-FPM の設定

`/etc/php/8.x/fpm/pool.d/www.conf` あたりで `listen = 127.0.0.1:9000` または UNIX ソケット (= `listen = /run/php/php8.x-fpm.sock`) を設定。BayServer 側を:

```
[club *.php]
    docker fcgiWarp
    destCity :unix:/run/php/php8.x-fpm.sock
```

にすれば UDS 接続でさらに高速。

## wp-config.php

DB 接続情報を設定:

```php
define('DB_NAME',     'wordpress');
define('DB_USER',     'wpuser');
define('DB_PASSWORD', 'secret');
define('DB_HOST',     'localhost');
define('DB_CHARSET',  'utf8mb4');

define('WP_HOME',    'https://wp.example.com');
define('WP_SITEURL', 'https://wp.example.com');
```

リバースプロキシ越しの場合は HTTPS 判定が効くよう以下も追加:

```php
if (!empty($_SERVER['HTTP_X_FORWARDED_PROTO']) &&
    $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}
```

## 静的アセット (= 画像 / CSS / JS) の高速化

`wp-content/uploads/` 配下などの静的ファイルは PHP 経由せずに BayServer から直接配信した方が高速です。

```
[city wp.example.com]
    [town /wp-content/uploads/]
        location /srv/wordpress/wp-content/uploads

    [town /]
        location /srv/wordpress
        [reroute]
            docker wordpress
        [club *.php]
            docker fcgiWarp
            destCity :unix:/run/php/php8-fpm.sock
```

`/wp-content/uploads/` の `town` を先に書くことで、画像系のリクエストは静的配信に振り分けられます。

## TLS 化

`.plan` の Port を 443 + Secure Docker で TLS 化。詳細: [HTTPS](https.md)、[Let's Encrypt](letsencrypt.md)。

## 動作確認

```bash
curl -I https://wp.example.com/
# HTTP/1.1 200 OK
# X-Powered-By: PHP/8.x   (= PHP-FPM 経由なら出る)
```

WordPress 管理画面 (`/wp-admin/`) にログインできれば成功。

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| Permalink が 404 | WordPress Docker / Reroute Docker が効いていない |
| 502 Bad Gateway | PHP-FPM が起動しているか、`destPort` / UDS パス確認 |
| HTTPS リダイレクトループ | `wp-config.php` の `$_SERVER['HTTPS']` 設定確認 |
| アップロードが反映されない | `wp-content/uploads/` の権限確認 |

---

## 関連

- [リバースプロキシ](reverse-proxy.md) — FCGI Warp の使い方
- [HTTPS / TLS の設定](https.md)
- [Docker 種別 / WordPress Docker](../reference/plan-reference.md)
