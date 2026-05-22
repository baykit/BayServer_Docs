# HTTP/3 を有効化する

HTTP/3 は QUIC (UDP ベース) 上で動く新しい HTTP プロトコルです。BayServer は **Java 版** と **Python 版** で HTTP/3 サーバ機能をサポートしています。

| 版 | HTTP/3 サーバ | 備考 |
|---|:---:|---|
| Java | ✓ | ネイティブライブラリ `libcroute` 等が必要 |
| Python | ✓ | `aioquic` パッケージが必要 |
| Ruby | ✕ | 未対応 |
| PHP | ✕ | 未対応 |
| TypeScript | ✕ | 未対応 |

## 前提

- **TLS 必須** (= QUIC は TLS 1.3 が前提)
- 有効な証明書 ([HTTPS / TLS の設定](https.md), [Let's Encrypt](letsencrypt.md))
- UDP 用ポートの開放 (= HTTPS と同じポート番号で UDP も)

## 設定

`.plan` の `[port]` ブロックで `enableH3 on` を追加:

```
[port 443]
    enableH2 on
    enableH3 on
    [secure]
        key  /etc/letsencrypt/live/www.example.com/privkey.pem
        cert /etc/letsencrypt/live/www.example.com/fullchain.pem
```

ブラウザは初回 TCP 接続後に `Alt-Svc` ヘッダで H3 を発見し、以後 H3 で通信します。

## Java 版で HTTP/3 を使う

Java 版 BayServer は **`libcroute`** (= QUIC native library) を使用します。

### libcroute のビルド

ソースから:

```bash
git clone https://github.com/baykit/croute.git
cd croute
make
```

ビルドした `libcroute.so` (Linux) または `libcroute.dylib` (macOS) を BayServer ホームの `lib/` 配下に配置:

```bash
cp croute/target/libcroute.so <BayServerホーム>/lib/
```

### Java 起動オプション

```bash
BSERV_OPT="-Djava.library.path=lib" bin/bayserver.sh -start
```

## Python 版で HTTP/3 を使う

Python 版 BayServer は **`aioquic`** を使用します。

```bash
pip install aioquic
```

それだけ。BayServer 側の `enableH3 on` を有効化して起動するだけで HTTP/3 がリッスンします。

## ファイアウォール / NAT

HTTP/3 は **UDP** なので、TCP しか開けていないファイアウォールでは通りません。

```bash
# Ubuntu / Debian の ufw
sudo ufw allow 443/udp

# RHEL / firewalld
sudo firewall-cmd --add-port=443/udp --permanent
sudo firewall-cmd --reload
```

クラウド (AWS / GCP / Azure) のセキュリティグループでも UDP 443 を許可。

## 動作確認

Chrome / Firefox の開発ツール (Network タブの Protocol カラム) で `h3` と表示されれば成功。または `curl` で:

```bash
curl --http3 https://www.example.com/
```

(`curl` が HTTP/3 サポートビルドである必要あり)

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| H3 ネゴシエートされない | UDP 443 がファイアウォール / NAT で通っているか |
| Java 版で `UnsatisfiedLinkError` | `libcroute` のパス・JVM オプション `-Djava.library.path` 確認 |
| Python 版で `aioquic` ImportError | `pip install aioquic` |
| `Alt-Svc` が出ない | `enableH3 on` が `[port]` ブロックに書かれているか |

---

## 関連

- [HTTPS / TLS の設定](https.md)
- [Let's Encrypt で証明書を取得する](letsencrypt.md)
- [言語実装別 比較表](../languages/comparison.md)
