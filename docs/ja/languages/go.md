# Go 版 BayServer

BayServer for Go は **epoll / kqueue ベースのノンブロッキング I/O** を採用したネイティブビルドの実装です。配布形態は単一バイナリ。

## インストール

配布バイナリ を使うのが基本:

```bash
# 配布バイナリを展開
tar zxf BayServer_Go-X.Y.Z.tgz
cd BayServer_Go-X.Y.Z
./bin/bayserver -start
```

ソースからビルドする場合:

```bash
git clone https://github.com/baykit/BayServer_Go.git
cd BayServer_Go
go build ./cmd/bayserver
```

## 主な特徴

- **epoll / kqueue ベース** — Linux では epoll、macOS / BSD では kqueue で I/O 多重化
- **単一バイナリ** — Go のクロスコンパイル特性を活かして展開即実行
- **軽量** — 起動が速い、メモリフットプリント小

## 使用例

基本構成は他の言語版と同じ:

```
[harbor]
    charset UTF-8
    grandAgents 4

[port 8080]

[city *]
    [town /]
        location www/root
        index    index.html
```

リバースプロキシ:

```
[city *]
    [town /api]
        [club *]
            docker httpWarp
            destCity backend.internal
            destPort 8080
            destTown /api
```

## マルチコアモード

```
[harbor]
    multiCore on
    grandAgents 4
```

`GOMAXPROCS` で利用 CPU 数を明示することも可能:

```bash
GOMAXPROCS=8 ./bin/bayserver -start
```

## 制約

- 内蔵アプリ実行エンジン (Servlet / Rack / WSGI 相当) は **無し** (= リバプロ用途が中心)
- HTTP/3 サーバ機能は未対応 (= 2026 時点)
- パッケージマネージャ経由の配布は無し (= 配布バイナリ or ソースビルドのみ)

## 用途

- リバースプロキシ専用サーバ (= フロント担当)
- 静的サイトホスティング
- Go アプリの隣に立てる Web サーバ (= デプロイバンドルの統一)
- 組込・コンテナ最適化 (= 単一バイナリで Docker イメージ最小化)

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| ビルド失敗 | Go バージョン確認 (`go version`)、go.mod 確認 |
| バイナリが起動しない | アーキテクチャ一致 (= `file ./bin/bayserver` で確認) |
| `GOMAXPROCS` が効かない | 環境変数の伝播確認 (= `systemd` の `Environment=` 等) |

---

## 関連

- [インストール](../getting-started/install.md)
- [言語実装別 比較表](comparison.md)
- [リバースプロキシ](../guide/reverse-proxy.md)
