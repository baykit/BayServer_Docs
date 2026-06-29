# Ruby 版 BayServer

BayServer for Ruby は Ruby 2.7.6 以降で動作します。**Rack** サーバを内蔵しているため、Rails / Sinatra / Roda などの Rack アプリをそのまま動かせます。

## インストール

```bash
gem install bayserver
mkdir bhome && cd bhome
bayserver -init
bayserver
```

より詳しい手順・起動オプションは [インストール](../getting-started/install.md#ruby) を参照。

## Rack を使う

### Rack アプリの準備

Rack アプリのルートディレクトリに `config.ru` (= `Rackup` ファイル) を置きます。これが Rack の標準起動ファイルです。

```ruby
# config.ru
require './app'
run MyApp.new
```

```ruby
# app.rb
class MyApp
  def call(env)
    [200, {'Content-Type' => 'text/plain'}, ['Hello, Rack!']]
  end
end
```

### `.plan` 設定例

```
[city *]
    [town /]
        location www/myapp
        [club *]
            docker rack
```

`town` の `location` を Rack アプリ (= `config.ru` がある) ディレクトリに向け、`club *` で全リクエストを Rack ハンドラに渡します。

### Rails アプリの場合

Rails の `config.ru` をそのまま使えます:

```
[city *]
    [town /]
        location /path/to/my-rails-app
        [club *]
            docker rack
```

事前に Rails アプリ側で `bundle install` 等を済ませておいてください。

### Sinatra / Roda

同様に、`config.ru` で Rack インスタンスを返せばどんな Rack アプリでも動きます。

## マルチコアモード / シングルコアモード

`[harbor] multiCore on` でマルチコアモード、`off` でシングルコアモード (= スレッド数 1)。Rack アプリの thread-safety に応じて選択:

```
[harbor]
    multiCore on        # スレッドセーフな Rack アプリ
    grandAgents 4       # スレッド数 = CPU コア数程度
```

## HTTP/3 サポート

Ruby 版は HTTP/3 サーバ機能を **未サポート** (= 2026 時点)。HTTP/2 (TLS) は対応しています。

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| `gem install bayserver` 失敗 | Ruby バージョンが 2.7.6 以上か確認 |
| Rack アプリが 500 を返す | `config.ru` がアプリのルートにあるか確認、依存 gem を `bundle install` |
| Rails で asset が出ない | `RAILS_ENV=production rails assets:precompile` を実行済みか確認 |

---

## 関連

- [インストール](../getting-started/install.md)
- [言語実装別 比較表](comparison.md)
- [設計ファイル (.plan) リファレンス](../reference/plan-reference.md)
