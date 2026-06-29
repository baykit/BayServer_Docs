# リバースプロキシ

BayServer をリバースプロキシとして使うことで、別のサーバ (= Tomcat / PHP-FPM / 別 BayServer / 任意の HTTP サーバ) の前段に立てます。

BayServer の世界観で言うと、プロキシは「特定の観光地に行こうとしたら、別の都市の観光地へワープしてしまった」状態。これを担当する Docker が **Warp Docker** です。

## Warp Docker の種類

| Docker | 上流プロトコル |
|---|---|
| **HTTP Warp** (`httpWarp`) | HTTP/1.1 (TLS 可) |
| **AJP Warp** (`ajpWarp`) | Apache JServ Protocol — Tomcat 等 |
| **FCGI Warp** (`fcgiWarp`) | Fast CGI — PHP-FPM、uWSGI 等 |

## 基本例: `/wordpress` を別ポートに転送

```
[city *]
    [town /wordpress]
        [club *]
            docker httpWarp
            destCity localhost
            destPort 8080
            destTown /wordpress
```

`http://your-server/wordpress/*` のリクエストが `http://localhost:8080/wordpress/*` にプロキシされます。

## 共通パラメータ

| パラメータ | 値 / 意味 |
|---|---|
| `docker` | `httpWarp` / `ajpWarp` / `fcgiWarp` |
| `destCity` | 上流のホスト名。UNIX ドメインソケットは `:unix:/path/to/sock.sock` |
| `destPort` | 上流のポート (HTTP デフォルト 80) |
| `destTown` | 上流のパス |
| `maxShips` | 上流との最大接続数 |
| `timeout` | 接続 / 読込のタイムアウト (秒) |

## HTTP Warp 固有

| パラメータ | 値 / 意味 |
|---|---|
| `secure` | 上流が TLS なら `on`。デフォルト `false` |

!!! note
    HTTP Warp が使うプロトコルは **HTTP/1.1 のみ**。HTTP/2 / HTTP/3 は使えません。

## 典型的な構成パターン

### 1. BayServer フロント + Tomcat (AJP)

Java Servlet コンテナである Tomcat に AJP で接続:

```
[city *]
    [town /app]
        [club *]
            docker ajpWarp
            destCity localhost
            destPort 8009
            destTown /app
```

### 2. BayServer フロント + PHP-FPM (FCGI)

PHP-FPM を FCGI で動かして PHP アプリを実行:

```
[city *]
    [town /]
        location www/wp
        [club *.php]
            docker fcgiWarp
            destCity localhost
            destPort 9000
```

### 3. Apache フロント + BayServer バック (AJP)

Apache の `mod_proxy_ajp` で BayServer に転送する場合は、BayServer 側を AJP Port で開きます:

```
[port 8009]
    docker ajp

[city *]
    [town /]
        location www/root
        [club *.do]
            docker servlet
```

### 4. UNIX ドメインソケット経由

ローカル接続なら UDS の方が高速:

```
[club *]
    docker httpWarp
    destCity :unix:/var/run/myapp.sock
    destTown /
```

### 5. 複数バックエンド (= パスで振り分け)

```
[city *]
    [town /api]
        [club *]
            docker httpWarp
            destCity api.internal
            destPort 8001
            destTown /

    [town /admin]
        [club *]
            docker httpWarp
            destCity admin.internal
            destPort 8002
            destTown /
```

## 静的ファイル + 動的プロキシの併用

`town` 単位でハンドラを変えられます。同じ `city` 内で:

```
[city *]
    [town /]
        location www/static
        index    index.html

    [town /api]
        [club *]
            docker httpWarp
            destCity backend.internal
            destPort 8080
            destTown /api
```

`/` は静的ファイル、`/api/*` だけ上流にプロキシ。

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| 502 Bad Gateway | 上流が起動しているか、`destPort` が正しいか |
| プロキシ先で IP が `127.0.0.1` に見える | 上流側で `X-Forwarded-For` ヘッダを参照 |
| パスが二重になる | `destTown` の指定確認 (= 上流側のパス) |
| Tomcat が動かない | AJP コネクタの設定確認 (= `secretRequired="false"` 等) |

---

## 関連

- [Docker 種別 / Warp Docker](../reference/plan-reference.md)
- [HTTPS / TLS の設定](https.md) — TLS フロント + プロキシ構成
- [設計ファイル (.plan) リファレンス](../reference/plan-reference.md)
