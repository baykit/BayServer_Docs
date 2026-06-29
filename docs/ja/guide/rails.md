# Rails を使う

Ruby 版 BayServer は **Rack サーバを内蔵** しているため、Rails / Sinatra / Roda などの Rack 互換アプリをそのまま動かせます。Puma / Unicorn の代替として使えます。

!!! note "Ruby 版限定"
    Rack を直接動かせるのは **Ruby 版 BayServer のみ**。他言語版から HTTP Warp で外部の Puma 等にプロキシする構成は [リバースプロキシ](reverse-proxy.md) を参照。

## 前提

- Ruby 版 BayServer ([インストール](../getting-started/install.md))
- Rails アプリ (= `config.ru` がある)
- `bundle install` 済み、必要なら `assets:precompile` 済み

## 最小構成

```
[city *]
    [town /]
        location /path/to/my-rails-app
        [club *]
            docker rack
```

- `location` は Rails アプリのルート (= `config.ru` がある場所)
- `[club *]` で **すべてのリクエスト** を Rack ハンドラに渡す

## config.ru

Rails が自動生成する `config.ru` がそのまま使えます:

```ruby
require_relative "config/environment"
run Rails.application
```

## 環境変数

BayServer プロセスに Rails 用の環境変数を渡しておく:

```bash
export RAILS_ENV=production
export SECRET_KEY_BASE="..."
export DATABASE_URL="postgresql://user:pass@localhost/myapp"

bin/bayserver.sh -start
```

## 静的アセットを高速化

Rails の `public/assets/` (= precompiled assets) は **BayServer から直接配信** した方が爆速です。Rack を経由させない:

```
[city *]
    [town /assets]
        location /path/to/my-rails-app/public/assets

    [town /packs]
        location /path/to/my-rails-app/public/packs   # = Webpacker 使用時

    [town /]
        location /path/to/my-rails-app
        [club *]
            docker rack
```

`town` の **並び順が重要** (= 上から順にマッチング)。`/assets` を先に置くこと。

## マルチコアモード

Rack ハンドラがスレッドセーフなら multiCore on:

```
[harbor]
    multiCore on
    grandAgents 4
```

`config/database.yml` の `pool` 値は `grandAgents` × `(Rails の concurrency)` 以上を確保:

```yaml
production:
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
```

## HTTPS

```
[port 443]
    enableH2 on
    [secure]
        key  /etc/letsencrypt/live/example.com/privkey.pem
        cert /etc/letsencrypt/live/example.com/fullchain.pem

[city *]
    [town /assets]
        location /path/to/my-rails-app/public/assets

    [town /]
        location /path/to/my-rails-app
        [club *]
            docker rack
```

詳細: [Let's Encrypt](letsencrypt.md)、[HTTPS の設定](https.md)。

## デプロイのフロー

1. `bundle install --without development test`
2. `rake assets:precompile`
3. `rake db:migrate`
4. BayServer を再起動 (= 新しいコードを Rack ハンドラがロード)

```bash
cd /path/to/my-rails-app
RAILS_ENV=production bundle install --without development test
RAILS_ENV=production bundle exec rake assets:precompile
RAILS_ENV=production bundle exec rake db:migrate

cd <BayServerホーム>
bin/bayserver.sh -restart
```

## Reverse-Proxy 越しの HTTPS 判定

BayServer 自身が TLS 終端する構成なら不要ですが、別フロント (= ALB / CDN) を挟む場合、Rails の `config/environments/production.rb` で:

```ruby
config.force_ssl = true
config.action_dispatch.trusted_proxies = [
  IPAddr.new('10.0.0.0/8'),
  IPAddr.new('127.0.0.1')
]
```

## WebSocket / ActionCable

ActionCable は HTTP/1.1 Upgrade ベース。BayServer の Rack 経路でも動きますが、**本格運用では別途 ActionCable サーバを立てる** ことを推奨 (= Rails 本体と分離してスケール)。

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| `LoadError: cannot load such file -- config.ru` | `location` 設定が Rails アプリのルートを指しているか |
| 静的 assets が 404 | `rake assets:precompile` 済か、`public/assets/` の存在確認、town の並び順 |
| 「Missing template」 | `RAILS_ENV` 設定確認、precompile されているか |
| DB connection error | `database.yml` の pool 値、PostgreSQL / MySQL の `max_connections` |
| 起動が遅い | Rails の eager loading で時間がかかる (= 通常 30 秒〜数分)、待つ |

---

## 関連

- [Ruby 版 BayServer](../languages/ruby.md)
- [Rails 完全構成 (実例)](../examples/rails.md) — TLS + 静的 + Let's Encrypt 込み
- [リバースプロキシ](reverse-proxy.md) — Puma 等の外部 Rack サーバを置く場合
- [HTTPS / TLS の設定](https.md)
- [Docker 種別 / Rack Docker](../reference/plan-reference.md)
