# Tuning ガイド

BayServer の性能を引き出すための代表的なパラメータと指針です。`.plan` の Harbor / Port / Club Docker を中心に調整します。

## Grand Agents 数

```
[harbor]
    grandAgents <N>
```

ワーカースレッド数。**物理 CPU コア数に揃える** のが目安。

- 少なすぎる → CPU を遊ばせる
- 多すぎる → コンテキストスイッチコスト + キャッシュ局所性悪化

[Tour / Ship / Agent](../architecture/tour-ship-agent.md) で説明している通り、各 Agent は独立した epoll ループを回します。`grandAgents = nproc` が出発点で、ベンチ取りながら微調整。

## Multiplexer 選択

```
[harbor]
    netMultiplexer  spider
```

迷ったら **`spider`** (= epoll/kqueue) を選ぶこと。他の選択肢は特殊用途向け。詳細: [Multiplexer](../architecture/multiplexer.md)。

## Max Ships / Tours

```
[harbor]
    maxShips         10000     # 最大同時接続数
    maxTours         100000    # 最大同時リクエスト数
    maxToursPerShip  100       # 1 Ship あたりの最大 Tour (= H2/H3 で重要)
```

`maxShips` はメモリ使用量とリンク。1 Ship あたり数 KB〜数十 KB を消費するため、大規模化する場合は OS の `ulimit -n` (= fd 上限) も併せて上げる:

```bash
ulimit -n 65535
```

## Buffer Size

```
[harbor]
    shipBufferSize 65536    # 1 Ship あたりの I/O バッファ
```

大きくすると 1 syscall で運べるデータ量が増えるが、メモリ消費も増えます。**64 KB 程度** が目安。大ボディ配信が中心なら 128 KB / 256 KB に上げると効果あり。

## Cargo / Direct Boarding

大ファイル配信時の性能を決めるパラメータです:

```
[harbor]
    directBoarding      on       # sendfile fast path を有効化
    maxDirectBoardings  10       # 同時 sendfile 数の上限
    maxCargoSize        50000    # in-memory cache の対象上限 (バイト)
    maxCargoSizeSecure  1100000  # SSL の場合の cache 対象上限
```

- **`directBoarding on`** → static file は kernel space で直接送信、CPU 使用率激減
- **`maxCargoSize`** を上げる → 中サイズファイル (= ~1MB 程度) もメモリキャッシュに乗る
- SSL 時は別パラメータ (= `maxCargoSizeSecure`)

実測ではこの 2 つの組合せで 100 KB ファイルの RPS が数倍上がるケースがあります。

## TCP オプション

接続レベルの最適化:

```
[harbor]
    enableTcpNoDelay on    # Nagle 無効 (= 小さなレスポンスを即送出)
```

リバースプロキシ用途 (= 上流が PHP-FPM 等で Nagle が効いている) では **`TCP_QUICKACK`** が自動適用され、delayed-ACK スタール (40ms) を回避します。

## Event Batch

```
[harbor]
    maxEventsPerReceive  32
```

1 回の epoll_wait で取り出す最大イベント数。大きいとレイテンシ下がるが他 Ship が遅れる、小さいと一律に分配。**32 程度** が中庸。

## Log の影響

```
[harbor]
    logLevel  warn        # debug/info で性能大幅低下
```

- `info` の access log は **`SimpleDateFormat` + `pwrite64`** をリクエストごとに呼ぶので CPU を食う。実測で **RPS が 1/5** になるケースあり (= BayServer Java の場合)
- 本番では `warn` 以下 + redirectFile で I/O 軽量化
- 開発時のみ `debug` を有効に

## アクセスログを無効化

完全に切る場合は Log Docker を `.plan` から削除するか、`redirectFile` を `/dev/null` 相当に:

```
[log /dev/null]
    format ""
```

## HTTP/2 / HTTP/3 関連

```
[port 443]
    enableH2 on
    enableH3 on
```

- H2 多重化で 1 接続あたり多数の Tour を捌けるため、`maxToursPerShip` を **少なくとも 100** に
- H3 は UDP + QUIC でロスに強い反面、CPU コストが H1 / H2 より高め

## JVM 版固有 (= Java)

```bash
BSERV_OPT="-Xmx4g -XX:+UseG1GC -XX:ActiveProcessorCount=8" bin/bayserver.sh -start
```

- **`-Xmx`** はワーキングセットの倍程度
- **G1GC** がデフォルト推奨 (ZGC は BayServer のオブジェクト確保パターンには合わず、実測で遅い)
- **`-XX:ActiveProcessorCount`** で利用 CPU 数を明示すると `Runtime.availableProcessors()` の結果が安定

## ベンチマークの取り方

[ベンチマーク方法論](benchmarks.md) を参照。

---

## 関連

- [パフォーマンス Tips](tips.md) — 個別の小ネタ
- [Multiplexer](../architecture/multiplexer.md)
- [Harbor Docker パラメータ](../reference/plan-reference.md)
