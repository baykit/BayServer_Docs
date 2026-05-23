# ガイド

特定の課題を解くための How-to ガイド集。**機能単位**の使い方を扱います。アプリケーション完成形のレシピは [実例集](../examples/index.md) を参照。

## アプリケーション実行

- [PHP を使う](php.md) — `php-cgi` を呼び出す軽量方式
- [php-fpm を使う](php-fpm.md) — 本番向けの永続プロセスプール + FCGI
- [WordPress を動かす](wordpress.md) — WordPress 専用 Docker と PHP-FPM 構成
- [Servlet を使う](servlet.md) — Java 版に内蔵された Servlet コンテナ
- [Rails を使う](rails.md) — Ruby 版に内蔵された Rack サーバ
- [WSGI を使う](wsgi.md) — Python 版に内蔵された WSGI サーバ (Django / Flask)

## ネットワーク / プロトコル

- [リバースプロキシ](reverse-proxy.md) — Warp Docker で別サーバの前段に立つ
- [HTTPS / TLS の設定](https.md) — Secure Docker、自己署名証明書、JKS 鍵ストア
- [Let's Encrypt で証明書を取得する](letsencrypt.md) — 無料証明書の取得と自動更新
- [HTTP/3 を有効化する](http3.md) — Java 版 / Python 版での H3 サーバ

## 運用

- [アクセス制限](access-control.md) — Permission Docker で IP / Basic 認証
- [Apache / Nginx / Tomcat からの移行](migration.md) — 設定対応表 + 並行運用パターン
