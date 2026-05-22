# HTTPS / TLS の設定

BayServer の SSL/TLS 化は **Secure Docker** を Port Docker にネストするだけです。

```
[port 2024]
    [secure]
        key  cert/server.key
        cert cert/server.crt
```

## クイックスタート: 既存のオレオレ証明書を使う

BayServer に同梱されている開発用証明書がそのまま使えます。`bayserver.sh -start` した直後、`https://localhost:2024/` にアクセスすれば証明書警告とともにアクセス可能です。

本番ではちゃんとした証明書を使ってください ([Let's Encrypt 連携](letsencrypt.md))。

## Secure Docker の制約

- **AJP / FCGI Port では使えません** (= HTTP Port 専用)
- 1 つの Port に対して 1 つの Secure Docker

## 自分でオレオレ証明書を作る

開発・テスト用に独自証明書を作る手順:

### 1. 秘密鍵を生成

```bash
openssl genrsa -out oreore.key 2048
```

### 2. 証明書署名要求 (CSR) を作成

```bash
openssl req -new -key oreore.key -out oreore.csr
```

対話的に組織名・メアド等を聞かれます (開発用なら全部 `Ore` でも可)。

### 3. 自己署名する

```bash
openssl x509 -req -days 365 -in oreore.csr -signkey oreore.key -out oreore.crt
```

### 4. BayServer に配置

```bash
mv oreore.key oreore.crt <BayServerホーム>/cert/
```

### 5. `.plan` に書く

```
[port 2024]
    [secure]
        key  cert/oreore.key
        cert cert/oreore.crt
```

### 6. 起動して確認

```bash
bin/bayserver.sh -restart
```

`https://localhost:2024/` にアクセス。ブラウザは「信頼できない証明書」警告を出しますが、開発用なら無視して進めば動作確認できます。

## ちゃんとした証明書を使う

オレオレ証明書ではブラウザが警告を出すので、公開サーバには信頼できる認証局 (CA) の証明書を使います。

- **無料** — [Let's Encrypt](letsencrypt.md) (= certbot で取得)
- **有料** — 各種商用認証局

詳細手順: [Let's Encrypt で証明書を取得する](letsencrypt.md)。

## 鍵ストア (Java 版) を使う

Java 版では `key` + `cert` ファイル方式の代わりに **PKCS12 / JKS** 鍵ストアを使えます。

### PKCS12 ストアを作る

```bash
openssl pkcs12 -export \
    -in oreore.crt \
    -inkey oreore.key \
    -out keystore.p12 \
    -name myalias \
    -password pass:changeit
```

### JKS ストアを作る

PKCS12 → JKS に変換:

```bash
keytool -importkeystore \
    -srckeystore keystore.p12 \
    -srcstoretype PKCS12 \
    -srcstorepass changeit \
    -destkeystore keystore.jks \
    -deststoretype JKS \
    -deststorepass changeit
```

### `.plan` での参照

```
[port 2024]
    [secure]
        keyStore     cert/keystore.jks
        keyStorePass changeit
```

## TLS バージョン / 暗号スイートの制御

詳細は [Docker 種別 / Secure Docker](../reference/docker-types-v3.md) を参照。

## HTTP/2 / HTTP/3 を有効化

TLS 化したポートで HTTP/2 / HTTP/3 を併用:

```
[port 2024]
    enableH2 on
    enableH3 on
    [secure]
        key  cert/server.key
        cert cert/server.crt
```

ALPN で対応プロトコルがネゴシエートされます。詳細: [HTTP/3 を有効化する](http3.md)。

## ファイアウォール

クラウド VPS などで `iptables` / セキュリティグループが効いている場合、HTTPS 用ポート (= 443 を本番では使う) を開ける必要があります。

```bash
sudo ufw allow 443/tcp
```

---

## 関連

- [Let's Encrypt で証明書を取得する](letsencrypt.md)
- [HTTP/3 を有効化する](http3.md)
- [Docker 種別 / Secure Docker](../reference/docker-types-v3.md)
