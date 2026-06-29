# php-fpm を使う

本番で PHP アプリを動かすなら、BayServer の前に置く形ではなく **BayServer から FCGI で php-fpm に転送する** 構成が一般的です。プロセスプールが永続化され、opcache が効くため高 RPS でも安定。

## 構成

```mermaid
graph LR
  Browser[ブラウザ] -->|HTTP/HTTPS| BayServer
  BayServer -->|FCGI| FPM[php-fpm]
  FPM --> App[PHP アプリ]
```

## 前提

- php-fpm 7.4 以降がインストール済
- BayServer は **任意の言語版** (= Java / Ruby / Python / TypeScript / Go 版どれでも可)

## php-fpm 側の設定

`/etc/php/8.x/fpm/pool.d/www.conf` で listen 設定。UDS (Unix Domain Socket) を推奨:

```
listen = /run/php/php8.x-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660
```

TCP で受けたい場合:

```
listen = 127.0.0.1:9000
```

`php-fpm` を起動:

```bash
sudo systemctl restart php8.x-fpm
sudo systemctl enable  php8.x-fpm
```

## BayServer 側の `.plan`

`.php` のリクエストを FCGI Warp で php-fpm に転送します。

### UDS 接続 (= 推奨、最速)

```
[city *]
    [town /]
        location /srv/myapp
        index    index.php
        [club *.php]
            docker fcgiWarp
            destCity :unix:/run/php/php8.x-fpm.sock
```

### TCP 接続

```
[club *.php]
    docker fcgiWarp
    destCity 127.0.0.1
    destPort 9000
```

## パラメータ

| パラメータ | 意味 |
|---|---|
| `docker` | `fcgiWarp` 固定 |
| `destCity` | php-fpm のホスト / UDS パス (= `:unix:/path/to/sock`) |
| `destPort` | TCP の場合のポート (デフォルト 80 なので必ず指定) |
| `maxShips` | 上流 (= php-fpm) との最大同時接続数 |
| `timeout` | 接続 / 読込タイムアウト (秒) |

## 動作確認

```bash
cat > /srv/myapp/index.php <<'EOF'
<?php echo "PHP: " . phpversion() . "\n"; ?>
EOF

bin/bayserver.sh -restart
curl -i http://localhost:8080/
# HTTP/1.1 200 OK
# X-Powered-By: PHP/8.x.y
# Content-Type: text/html; charset=UTF-8
#
# PHP: 8.x.y
```

`X-Powered-By: PHP/8.x` が見えれば、php-fpm 経由で実行されています。

## 静的アセットを高速化

PHP-FPM を経由せず、画像 / CSS / JS は BayServer から直接配信:

```
[city *]
    [town /uploads/]
        location /srv/myapp/uploads

    [town /assets/]
        location /srv/myapp/assets

    [town /]
        location /srv/myapp
        index    index.php
        [club *.php]
            docker fcgiWarp
            destCity :unix:/run/php/php8.x-fpm.sock
```

`town` の並び順は重要 (= 上から順にマッチング)。静的 town を先に書くこと。

## OPcache の確認

php-fpm 永続プロセスの強みは **OPcache** が効くこと。`/etc/php/8.x/fpm/php.ini`:

```ini
opcache.enable = 1
opcache.memory_consumption = 256
opcache.max_accelerated_files = 20000
opcache.validate_timestamps = 0    # = 本番のみ、dev では 1
```

`opcache.validate_timestamps = 0` にすると **ファイル変更を検知しない** ので、デプロイ後は `systemctl reload php8.x-fpm` でリロードが必要。

## プールサイズの tuning

`/etc/php/8.x/fpm/pool.d/www.conf`:

```
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 10
pm.max_requests = 1000
```

- **`pm.max_children`** = ピーク時の同時 PHP 実行数。CPU / メモリと相談
- **`pm.max_requests`** > 0 で N リクエスト後に worker をリサイクル (= メモリリーク対策)
- 高 RPS の場合は `pm = static` で固定数も検討

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| 502 Bad Gateway | php-fpm が起動しているか (`systemctl status php8.x-fpm`)、UDS パス / port が一致 |
| Permission denied (UDS) | `listen.owner` / `listen.group` / `listen.mode` 確認、BayServer ユーザがソケットを開けるか |
| 504 Gateway Timeout | php-fpm の `request_terminate_timeout`、BayServer の `timeout` を両方上げる |
| プロセスが詰まる | `pm.max_children` を上げる、`pm.max_requests` で reset 頻度を上げる |
| PHP が古いまま | `opcache.validate_timestamps = 0` なら `systemctl reload php8.x-fpm` |

---

## 関連

- [PHP を使う (= PhpCgi 方式)](php.md) — 開発・テスト向けの軽量方式
- [WordPress を動かす](wordpress.md) — WordPress 専用 Docker
- [WordPress 完全構成 (実例)](../examples/wordpress-full.md) — TLS + 静的アセット + Let's Encrypt 込み
- [リバースプロキシ](reverse-proxy.md) — Warp Docker 全般
- [Docker 種別 / FCGI Warp](../reference/docker-types.md)
