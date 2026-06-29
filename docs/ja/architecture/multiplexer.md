# Multiplexer

BayServer の I/O 多重化エンジンです。各 Grand Agent が 1 つの Multiplexer を持ち、自分が担当する Ship の I/O イベントを待ち受けます。

`.plan` の `[harbor]` で `netMultiplexer <type>` を指定。デフォルトは `spider`。

## Multiplexer 一覧

| Type | 内部実装 | 用途 |
|---|---|---|
| **spider** | epoll (Linux) / kqueue (macOS/BSD) | **デフォルト推奨**。ノンブロッキング、最高性能 |
| **spin** | スピンロックでビジーウェイト | 超低レイテンシ用途、CPU 食いまくる |
| **pigeon** | 言語固有の async I/O ライブラリ | Java 版は AsynchronousSocketChannel など |
| **taxi** | スレッドプール | 1 I/O = 1 タスク、ファイル送信などに |
| **train** | キュー + ワーカ | 順序保証が必要な処理に |
| **job** | OS スレッド (= 1 接続 1 スレッド) | レガシー / デバッグ用 |

## デフォルト (spider) を選ぶべき理由

`spider` (= epoll / kqueue) は **ノンブロッキング** で動作し、1 つのスレッドが多数の Ship のイベントをまとめて拾えます。BayServer のメインユースケース (= 高 RPS な HTTP サーバ / リバースプロキシ) では他より速いです。

他の Multiplexer は特殊な用途 (= AsynchronousFileChannel が必要なファイル送信、レガシー Java の non-NIO 環境、等) で限定的に使います。

## ファイル転送用の Multiplexer

ネットワーク I/O とは別に **ファイル I/O 用** の Multiplexer も設定できます。

```
[harbor]
    netMultiplexer  spider
    fileMultiplexer pigeon
    logMultiplexer  pigeon
```

| パラメータ | 意味 |
|---|---|
| `netMultiplexer` | ネットワーク I/O (= Ship の読み書き) |
| `fileMultiplexer` | ファイル I/O (= 静的ファイル送信) |
| `logMultiplexer` | ログ書き込み |
| `cgiMultiplexer` | CGI プロセスとの I/O |

ファイル送信は内部的に `sendfile` (= Direct Boarding) 経由になることが多く、ファイル Multiplexer の影響は限定的です。

## Direct Boarding (= sendfile fast path)

大きな静的ファイルを返すときに、ユーザ空間を通さずに `sendfile(2)` システムコールで直接ソケットに転送する経路です。

`.plan` で:

```
[harbor]
    directBoarding on
    maxDirectBoardings 10
```

`maxDirectBoardings` は同時 sendfile 数の上限。

巨大ファイル配信時に CPU 使用率が大幅に下がります。MemBarge cache (= [Barge Docker](../reference/plan-reference.md)) と組み合わせるとさらに高速化。

## Barge Docker (= メモリキャッシュ)

ファイル内容をプロセス内 LRU キャッシュに保持する仕組みです:

```
[harbor]
    maxCargoSize 1100000      # 1 ファイルあたりのキャッシュ可否しきい値

[city *]
    [town /]
        location www/root
        [barge]
            capacity 100M     # 100 MB のメモリキャッシュ
```

ファイルが `maxCargoSize` 以下なら次回アクセス時にディスクを叩かず、メモリから直接送ります。

## 内部の登場人物 (= ソース読解用)

ソースを読むと出てくる用語:

| 用語 | 意味 |
|---|---|
| **Transporter** | I/O 読み書きを抽象化したクラス |
| **Rudder** | Socket / File / Pipe の handle 抽象 |
| **Letter** | Multiplexer 間で渡されるイベント |
| **Store** | プールされたオブジェクト (= ObjectStore / CommandStore / RudderStateStore) |

これらは将来的に [開発者向け / 内部実装の地図](../developer/internals.md) で詳述予定。

---

## 関連

- [Tour / Ship / Agent](tour-ship-agent.md) — Multiplexer の上位構造
- [パフォーマンス / Tuning](../performance/tuning.md) — Multiplexer 選択の指針
- [Docker 種別 / Harbor Docker](../reference/plan-reference.md)
