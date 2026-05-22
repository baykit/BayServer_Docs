# Hello World

BayServer をインストールしたら、まずは自分のコンテンツを配信してみましょう。

## デフォルト構成の確認

[インストール](install.md) 直後、BayServer ホームには既にサンプルの `.plan` (= 設計ファイル) と簡単な静的コンテンツが入っています。`bin/bayserver.sh -start` で起動すれば `http://localhost:2020/` でデフォルトページが表示されます。

`plan/bayserver.plan` を開くと、おおむね以下のような構造が見えます:

```
[harbor]
    charset UTF-8
    grandAgents 4

[port 2020]

[port 2024]
    [secure]
        key  cert/oreore.key
        cert cert/oreore.crt

[city *]
    [town /]
        location www/root
        index    index.html
```

## 自分のコンテンツに差し替える

BayServer ホーム配下の `www/root/` に置いた静的ファイル (HTML / 画像 / CSS / JS) が、そのままルートで配信されます。

```bash
echo '<h1>Hello, BayServer!</h1>' > www/root/index.html
bin/bayserver.sh -start
```

ブラウザで `http://localhost:2020/` を開くと「Hello, BayServer!」が見えます。

## 別ディレクトリを公開する

`town` の `location` を別パスに変えれば、任意のディレクトリを公開できます。

```
[city *]
    [town /]
        location /var/www/mysite
        index    index.html
```

ホスト名で分けたい場合は `city` を複数置きます:

```
[city www.example.com]
    [town /]
        location /srv/example/www

[city blog.example.com]
    [town /]
        location /srv/blog/www
```

## ポートを変える

`[port 2020]` を `[port 8080]` に書き換えれば、HTTP 待ち受けポートを変更できます。1024 未満のポートを使う場合は OS の特権が必要になるため (= Linux なら `setcap` か `sudo`)、開発時はそのまま 4 桁ポートを使うのが楽です。

## 設定変更を反映させる

`.plan` を編集したら BayServer を再起動して反映します:

```bash
bin/bayserver.sh -stop
bin/bayserver.sh -start
```

## 次のステップ

- [`.plan` 文法と Docker 種別](../reference/index.md) — 設定ファイルの全体像
- [HTTPS / TLS の設定](../guide/https.md) — ちゃんとした証明書で SSL 化する
- [リバースプロキシ](../guide/reverse-proxy.md) — バックの Tomcat / PHP-FPM 等に繋ぐ
- [言語別固有事項](../languages/index.md) — Servlet / Rack / WSGI 連携
