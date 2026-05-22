# 内部実装の地図

BayServer のソースを読むときの入口。Java 版を例に説明しますが、他言語版もクラス名 / モジュール構成は揃えてあります。

## モジュール構成 (Java 版)

```
BayServer_Java/
├── modules/
│   ├── bayserver-core/              # 中核 (= GrandAgent, Tour, Ship, Multiplexer)
│   ├── bayserver-docker-http/       # HTTP/1, HTTP/2, HTTP/3
│   ├── bayserver-docker-ajp/        # AJP
│   ├── bayserver-docker-fcgi/       # FCGI
│   ├── bayserver-docker-cgi/        # CGI (= fork+exec モデル)
│   ├── bayserver-docker-servlet/    # Servlet コンテナ
│   ├── bayserver-docker-wordpress/  # WordPress Docker
│   └── bayserver/                   # ランチャ (= main + init.jar)
└── build.sh                         # 全モジュールビルド
```

## bayserver-core の主要クラス

### Boot / BayServer

`yokohama.baykit.bayserver.BayServer` がエントリポイント。`.plan` を読み込み、各 Docker を初期化、GrandAgent を起動します。

### GrandAgent

`yokohama.baykit.bayserver.agent.GrandAgent` がワーカースレッド。

```
GrandAgent.run() {
    while (running) {
        multiplexer.receive(letters)
        for (letter in letters) {
            letter.dispatch()
        }
    }
}
```

### Multiplexer

`yokohama.baykit.bayserver.agent.multiplexer.*`:

| クラス | 役割 |
|---|---|
| `SpiderMultiplexer` | epoll/kqueue ベース (= デフォルト) |
| `PigeonMultiplexer` | AsynchronousSocketChannel |
| `TaxiMultiplexer` | スレッドプール |
| `TrainMultiplexer` | キュー + コンシューマ |
| `JobMultiplexer` | OS スレッド 1 接続 1 スレッド |
| `SpinMultiplexer` | スピンロック |

### Ship

`yokohama.baykit.bayserver.ship.Ship` の派生:

| クラス | 役割 |
|---|---|
| `InboundShip` | クライアント接続 |
| `WarpShip` | 上流接続 (= プロキシ) |
| `H1InboundShip` / `H2InboundShip` / `H3InboundShip` | プロトコル別 |

### Tour

`yokohama.baykit.bayserver.tour.Tour` がリクエスト 1 つを表す。`tour.req` (= TourReq) と `tour.res` (= TourRes) でリクエスト/レスポンスを操作。

### Rudder

`yokohama.baykit.bayserver.rudder.Rudder` は Socket / File / Pipe の handle 抽象。Multiplexer はこれを介して I/O。

### Transporter

`yokohama.baykit.bayserver.agent.multiplexer.Transporter` は I/O 読み書きの抽象。`reqRead` / `reqWrite` / `reqClose` を持ちます。

### Store

オブジェクト pool 群。`yokohama.baykit.bayserver.util.ObjectStore` を基底に、各種特殊版があります:

- `TourStore` — Tour の pool
- `RudderStateStore` — Rudder state の pool
- `CommandStore` — protocol command の pool
- `ProtocolHandlerStore` — protocol handler の pool

## protocol module の構成

各 protocol (= HTTP / AJP / FCGI) は同じ抽象を実装:

```
xxx-core/
├── XxxPortDocker.java         # Port Docker (= 受信側)
├── XxxWarpDocker.java         # Warp Docker (= 送信側)
├── XxxInboundHandler.java     # protocol handler (= デコード/エンコード)
├── XxxWarpHandler.java        # 上流 handler
├── XxxPacket.java             # packet 表現
├── XxxPacketPacker.java       # wire 形式に encode
├── XxxPacketUnPacker.java     # wire 形式から decode
└── command/
    ├── CmdXxx.java            # 各 command
    └── ...
```

## servlet module

Java 版固有。Servlet 4.0 仕様を実装:

- `ServletDocker` — Club Docker、Servlet を実行
- `ServletEngine` — Servlet API 実装
- `HttpServletRequestDuck` / `HttpServletResponseDuck` — Servlet 仕様の Req/Res

## Letter 機構

Multiplexer 間 (= 異種スレッド間) でイベントを渡す仕組み。

```java
public class ReadLetter extends Letter {
    public final Rudder rudder;
    public final ByteBuffer buf;
    public final int n;
    // ...
}
```

`agent.sendLetter(letter)` で送信、`GrandAgent.dispatch()` で受信側が処理。

## ソース読解の入口

「動的 HTTP リクエスト処理を追いたい」場合のおすすめ読み順:

1. `BayServer.main()` (= 起動)
2. `GrandAgent.run()` (= イベントループ)
3. `SpiderMultiplexer.receive()` (= epoll_wait)
4. `H1InboundHandler.bytesReceived()` (= HTTP/1 デコード)
5. `Tour.go()` (= 該当 city/town/club に振分け)
6. `FileDocker.arrive()` 等 (= Club Docker の処理)
7. `InboundShip.sendResContent()` (= レスポンス送信)
8. `SpiderMultiplexer.onWritable()` (= 書き戻し)

---

## 関連

- [ビルド方法](building.md)
- [コーディングスタイル](coding-style.md)
- [新しい Docker を書く](adding-a-docker.md)
- [Tour / Ship / Agent](../architecture/tour-ship-agent.md)
- [Multiplexer](../architecture/multiplexer.md)
