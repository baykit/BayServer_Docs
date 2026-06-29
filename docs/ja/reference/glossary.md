# 用語集

BayServer の **港湾比喩** を中心とした主要用語の早見表。各用語の詳細は関連ページへリンクしています。

## 設定 (= `.plan`) で使う用語

| 用語 | 比喩 | 役割 | 詳細 |
|---|---|---|---|
| **harbor** | 港湾 | サーバ全体の取りまとめ (= タイムアウト、スレッド数、I/O 多重化) | [Docker / Harbor](plan-reference.md#harbor-docker) |
| **port** | 港 | リッスンポート (TCP / UDP)、プロトコル設定 | [Docker / Port](plan-reference.md#port-docker) |
| **secure** | 通信暗号 | TLS / SSL の鍵と証明書 | [Docker / Secure](plan-reference.md#secure-docker) / [HTTPS ガイド](../guide/https.md) |
| **permission** | 入国審査 | IP / ホスト制限、Basic 認証 | [Docker / Permission](plan-reference.md#permission-docker) / [アクセス制限](../guide/access-control.md) |
| **city** | 都市 | バーチャルホスト | [Docker / City](plan-reference.md#city-docker) |
| **town** | 街区 | URL パス区画 | [Docker / Town](plan-reference.md#town-docker) |
| **club** | クラブ | 拡張子 / パスパターンのハンドラ (= file / cgi / php / servlet / warp) | [Docker / Club](plan-reference.md#club-docker) |
| **log** | 入国記録 | アクセスログ | [Docker / Log](plan-reference.md#log-docker) |
| **reroute** | 行先変更 | URL Rewriting | [Docker / Reroute](plan-reference.md#reroute-docker) |
| **trouble** | 問題対処 | HTTP エラー時の代替挙動 | [Docker / Trouble](plan-reference.md#trouble-docker) |
| **barge** | 艀 (= はしけ) | プロセス内 LRU メモリキャッシュ | [Multiplexer / Barge](../architecture/multiplexer.md) |

## 処理モデルの用語

| 用語 | 比喩 | 実体 | 詳細 |
|---|---|---|---|
| **ship** | 船舶 | 1 TCP 接続 | [Tour/Ship/Agent](../architecture/tour-ship-agent.md) |
| **tour** | 旅 | 1 リクエスト/レスポンス | 同上 |
| **agent (grand agent)** | 港湾従事者 | ワーカースレッド | 同上 |
| **multiplexer** | 多重化エンジン | epoll / kqueue 等の I/O 多重化 | [Multiplexer](../architecture/multiplexer.md) |

## ship のサブ種

| 用語 | 役割 |
|---|---|
| **InboundShip** | クライアントからの接続 (= 普通の受信) |
| **WarpShip** | 上流サーバへの接続 (= リバースプロキシ時) |

## Multiplexer の種類

| 値 | 内部実装 | 主な用途 |
|---|---|---|
| **spider** | epoll (Linux) / kqueue (macOS) | デフォルト推奨、最高性能 |
| **spin** | スピンロック | 超低レイテンシ用途 |
| **pigeon** | 言語固有の async I/O | Java の AsynchronousSocketChannel 等 |
| **taxi** | スレッドプール | ファイル送信等の重い I/O |
| **train** | キュー + ワーカ | 順序保証が必要な処理 |
| **job** | OS スレッド (= 1 接続 1 スレッド) | レガシー / デバッグ |

## ソース読解時の追加用語

ソースを読むと出てくる、`.plan` には登場しない概念:

| 用語 | 意味 |
|---|---|
| **Letter** | Multiplexer 間で渡されるイベントオブジェクト |
| **Rudder** | Socket / File / Pipe の handle 抽象 |
| **Transporter** | I/O 読み書きを抽象化したクラス |
| **Store** | プールされたオブジェクト (= ObjectStore / TourStore / CommandStore 等) |
| **Direct Boarding** | sendfile() を使ったユーザ空間を経由しないファイル送信経路 |
| **MemCargo** | Barge にキャッシュされた個別ファイルの中身 |

## 略語 / 関連プロトコル

| 略語 | 意味 |
|---|---|
| **AJP** | Apache JServ Protocol — Tomcat 等が使う |
| **FCGI** | Fast CGI — PHP-FPM / uWSGI 等が使う |
| **WSGI** | Web Server Gateway Interface — Python の Web アプリ標準 (PEP 3333) |
| **Rack** | Ruby の Web アプリ標準インタフェース |
| **PSGI** | Perl の Web アプリ標準 |
| **HSTS** | HTTP Strict Transport Security |
| **HPACK** | HTTP/2 のヘッダ圧縮 (RFC 7541) |
| **QPACK** | HTTP/3 のヘッダ圧縮 (RFC 9204) |
| **ALPN** | Application-Layer Protocol Negotiation — TLS 上でプロトコル合意 |
| **C10K** | 1 台のサーバで 1 万クライアントを捌く問題 |

---

## 関連

- [BayServer とは](../about.md) — 港湾比喩の由来
- [設計ファイル (.plan) リファレンス](plan-reference.md)
- [Tour / Ship / Agent](../architecture/tour-ship-agent.md)
