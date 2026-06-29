# WSGI を使う

Python 版 BayServer は **WSGI サーバを内蔵** しているため、Django / Flask / Bottle / FastAPI (= WSGI モード) などの WSGI アプリをそのまま動かせます。Gunicorn / uWSGI の代替として使えます。

!!! note "Python 版限定"
    WSGI を直接動かせるのは **Python 版 BayServer のみ**。他言語版から HTTP Warp で外部の Gunicorn 等にプロキシする構成は [リバースプロキシ](reverse-proxy.md) を参照。

## 前提

- Python 版 BayServer ([インストール](../getting-started/install.md))
- WSGI 仕様 (PEP 3333) に準拠した callable を持つ Python アプリ

## 最小構成

```
[city *]
    [town /]
        location /path/to/my-wsgi-app
        [club *]
            docker wsgi
            module app
            application application
```

| パラメータ | 意味 |
|---|---|
| `module` | Python モジュール名 (= ファイル名から `.py` を除いたもの) |
| `application` | WSGI コールアブルの変数名 (デフォルト `application`) |

## 最小の WSGI アプリ

`/path/to/my-wsgi-app/app.py`:

```python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain; charset=utf-8')])
    return [b'Hello, WSGI!']
```

```bash
bin/bayserver.sh -restart
curl http://localhost:8080/
# Hello, WSGI!
```

## Django

Django プロジェクトは `<project>/wsgi.py` に `application` callable が自動生成されています。

```
[city *]
    [town /]
        location /path/to/mydjango
        [club *]
            docker wsgi
            module mysite.wsgi
            application application
```

| 項目 | 設定 |
|---|---|
| `location` | `manage.py` がある Django プロジェクトのルート |
| `module` | `mysite.wsgi` (= `mysite/wsgi.py`) |
| `application` | 通常 `application` (= Django のデフォルト) |

環境変数:

```bash
export DJANGO_SETTINGS_MODULE=mysite.settings
export PYTHONPATH=/path/to/mydjango
```

## Flask

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, Flask!'
```

```
[club *]
    docker wsgi
    module app
    application app
```

Flask の `Flask(__name__)` インスタンス (= `app`) がそのまま WSGI コールアブルです。

## 静的アセットを高速化

Django の `collectstatic` で集めた静的ファイルを **WSGI を経由せず** 直接配信:

```bash
python manage.py collectstatic
```

```
[city *]
    [town /static]
        location /path/to/mydjango/static

    [town /media]
        location /path/to/mydjango/media

    [town /]
        location /path/to/mydjango
        [club *]
            docker wsgi
            module mysite.wsgi
            application application
```

`STATIC_ROOT` / `MEDIA_ROOT` の Django 側設定と `town` の `location` を一致させる。

## マルチコアモード

```
[harbor]
    multiCore on
    grandAgents 4
```

WSGI アプリ側がスレッドセーフかつ DB コネクションプールを適切に持っていれば OK。Django ORM はスレッドセーフですが、`pool` サイズに注意:

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        # = pool は psycopg2/pgbouncer 側で管理
        'CONN_MAX_AGE': 60,
    }
}
```

## HTTPS

```
[port 443]
    enableH2 on
    [secure]
        key  /etc/letsencrypt/live/example.com/privkey.pem
        cert /etc/letsencrypt/live/example.com/fullchain.pem

[city *]
    [town /static]
        location /path/to/mydjango/static
    [town /]
        location /path/to/mydjango
        [club *]
            docker wsgi
            module mysite.wsgi
            application application
```

詳細: [HTTPS の設定](https.md)、[Let's Encrypt](letsencrypt.md)。

## ASGI (FastAPI / Starlette 等) について

BayServer Python 版は **WSGI のみ対応** (= 2026 時点)。FastAPI / Starlette など非同期フレームワーク (ASGI) を使いたい場合は:

1. アプリ側で `asgiref.WsgiToAsgi` / 逆方向の wrapper を使い WSGI 化、または
2. 外部の Uvicorn / Hypercorn を立てて BayServer から HTTP Warp で繋ぐ ([リバースプロキシ](reverse-proxy.md))

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| `ImportError: No module named 'mysite'` | `location` 設定、`PYTHONPATH` 設定確認 |
| 静的ファイルが 404 | `python manage.py collectstatic` 済か、`STATIC_ROOT` と `town location` が一致 |
| 「DJANGO_SETTINGS_MODULE not set」 | 環境変数 `DJANGO_SETTINGS_MODULE` 確認 |
| DB コネクションエラー | `psycopg2`/`mysqlclient` インストール確認、DB の `max_connections` |
| メモリ使用量が増え続ける | アプリ側のリークか、`multiCore on` でプロセス再起動が無い (= 定期再起動の運用検討) |

---

## 関連

- [Python 版 BayServer](../languages/python.md)
- [リバースプロキシ](reverse-proxy.md) — Gunicorn 等の外部 WSGI サーバを置く場合
- [HTTP/3 を有効化する](http3.md) — Python 版で H3 サーバを立てる
- [HTTPS / TLS の設定](https.md)
- [Docker 種別 / WSGI Docker](../reference/plan-reference.md)
