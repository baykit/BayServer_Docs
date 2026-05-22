# Let's Encrypt で証明書を取得する

[Let's Encrypt](https://letsencrypt.org/) は無料で自動取得・自動更新できる SSL/TLS 証明書サービスです。BayServer と組み合わせて公開サイトを HTTPS 化できます。

## 前提

- 公開可能なドメイン名 (= 例: `www.example.com`) を所有
- そのドメインの DNS A レコードがサーバの公開 IP を指している
- サーバの **80 番ポート** が外から繋がる (= HTTP-01 チャレンジで使う)

## 手順

### 1. certbot をインストール

```bash
sudo apt install certbot          # Debian/Ubuntu
sudo dnf install certbot          # RHEL/Fedora
```

### 2. BayServer の 80 番ポートを開ける

`.plan` に Port 80 を追加:

```
[port 80]
    [city *]
        [town /.well-known/acme-challenge/]
            location /var/www/letsencrypt/.well-known/acme-challenge
```

または certbot の standalone モードを使う場合は BayServer を一旦止める:

```bash
bin/bayserver.sh -stop
```

### 3. 証明書を取得

#### standalone モード (= 一時的に 80 番を certbot に渡す)

```bash
sudo certbot certonly --standalone -d www.example.com
```

#### webroot モード (= BayServer は止めずに継続)

```bash
sudo certbot certonly --webroot \
    -w /var/www/letsencrypt \
    -d www.example.com
```

成功すると `/etc/letsencrypt/live/www.example.com/` に証明書ファイル一式が配置されます:

| ファイル | 役割 |
|---|---|
| `privkey.pem` | 秘密鍵 |
| `cert.pem` | サーバ証明書 |
| `chain.pem` | 中間証明書 |
| `fullchain.pem` | サーバ証明書 + 中間 (= BayServer に渡すのはこれ) |

### 4. BayServer に組み込む

`.plan` の `[secure]` ブロックで参照:

```
[port 443]
    [secure]
        key  /etc/letsencrypt/live/www.example.com/privkey.pem
        cert /etc/letsencrypt/live/www.example.com/fullchain.pem
```

### 5. BayServer を再起動

```bash
bin/bayserver.sh -restart
```

`https://www.example.com/` でアクセスして、ブラウザが緑色の鍵を表示すれば成功。

### 6. 自動更新の設定

Let's Encrypt の証明書は 90 日で期限切れになるため自動更新を設定します。`/etc/cron.d/certbot` に:

```cron
0 3 * * * root certbot renew --quiet --post-hook "<BayServerホーム>/bin/bayserver.sh -restart"
```

`--post-hook` で更新成功時に BayServer を再起動して新証明書を再読み込みします。

## 権限問題への対処

`/etc/letsencrypt/` は root 所有のため、BayServer を一般ユーザで動かしていると証明書を読めない場合があります。対策:

1. BayServer を起動するユーザを `letsencrypt` グループ (もしくは適当なグループ) に追加し、`/etc/letsencrypt/` をそのグループで読めるよう設定
2. または `/etc/letsencrypt/live/www.example.com/` のシンボリックリンク先 (= `archive/`) のパーミッションを調整
3. 簡易には証明書ファイルを BayServer ホーム配下にコピー (= ただし更新時に同期する仕組みが必要)

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| certbot で 80 番が使えない | 既に BayServer 等が listen してないか、`ss -tln \| grep :80` |
| Permission denied で証明書読込失敗 | BayServer プロセスのユーザの権限確認 |
| 「証明書チェーンが不完全」と言われる | `cert.pem` ではなく `fullchain.pem` を使う |

---

## 関連

- [HTTPS / TLS の設定](https.md) — Secure Docker の基本
- [アクセス制限](access-control.md) — Permission Docker 連携
