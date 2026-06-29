# Python 版 BayServer

BayServer for Python は Python 3.7 以降で動作します。**WSGI** サーバを内蔵しているため、Django / Flask / Bottle などの WSGI アプリをそのまま動かせます。

## インストール

```bash
pip install bayserver
mkdir bhome && cd bhome
bayserver -init
bayserver
```

より詳しい手順・起動オプションは [インストール](../getting-started/install.md#python) を参照。

## WSGI を使う

### WSGI アプリの準備

WSGI 仕様に従ったコールアブル (= 関数 or クラス) を `application` として公開します。

```python
# app.py
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return [b'Hello, WSGI!']
```

### `.plan` 設定例

```
[city *]
    [town /]
        location www/myapp
        [club *]
            docker wsgi
            module app
            application application
```

| パラメータ | 意味 |
|---|---|
| `module` | Python モジュール名 (= ファイル名から `.py` を除いたもの) |
| `application` | コールアブルの名前 (デフォルト `application`) |

### Django アプリ

```
[city *]
    [town /]
        location /path/to/my-django-project
        [club *]
            docker wsgi
            module mysite.wsgi
            application application
```

Django プロジェクトのルートに `manage.py` がある位置を `location` に指定します。`mysite/wsgi.py` の `application` が WSGI エントリポイントです。

### Flask アプリ

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

## HTTP/3 サポート

Python 版は HTTP/3 サーバ機能を **サポート** しています。ただし外部ライブラリ (= `aioquic`) の導入が必要です。

```bash
pip install aioquic
```

詳細: [HTTP/3 を有効化する](../guide/http3.md)。

## マルチコア / シングルコア

```
[harbor]
    multiCore on        # 推奨。WSGI アプリ側のスレッドセーフ性を確認
    grandAgents 4
```

Django のような ORM ベースのアプリは、コネクションプール周りで thread-safety に注意が必要です。

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| `ImportError: No module named ...` | アプリディレクトリ (= `location`) と `module` 指定が正しいか確認 |
| Django で static ファイルが出ない | `collectstatic` 済みか / `STATIC_ROOT` 設定確認 |

---

## 関連

- [インストール](../getting-started/install.md)
- [言語実装別 比較表](comparison.md)
- [HTTP/3 を有効化する](../guide/http3.md)
