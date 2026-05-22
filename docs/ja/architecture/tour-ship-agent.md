# Tour / Ship / Agent

BayServer の処理モデルを理解する上で要となる 3 つの概念です。`.plan` を書くだけなら知らなくても困りませんが、性能 tuning やコードを読むときに不可欠です。

## 全体像

```
client ──TCP──▶ [Ship]    [Tour 1] ─┐
                  │       [Tour 2] ─┼─▶  Agent  ──▶  responses
                  │       [Tour 3] ─┘
                  └── (= 1 接続)
```

| 概念 | 比喩 | 実体 |
|---|---|---|
| **Ship** (船舶) | 港に停泊する船 | 1 TCP 接続 |
| **Tour** (旅) | 船から降りた観光客のツアー | 1 リクエスト/レスポンス |
| **Agent** (係員) | 港湾の従事者 | ワーカースレッド |

## Ship — 接続

1 つの Ship は 1 つの **TCP 接続** に対応します。HTTP/1.1 では keep-alive で同じ Ship 上に複数のリクエストが流れます。HTTP/2 / HTTP/3 では 1 Ship 上に複数の Tour を **多重化** できます。

Ship は Harbor の `maxShips` パラメータで上限を制御します。`maxShips` を超える接続要求は待たされます。

### Ship の種類

| 種類 | 役割 |
|---|---|
| **InboundShip** | クライアントからの接続 (= 通常の HTTP リクエスト) |
| **WarpShip** | 上流サーバへの接続 (= リバースプロキシで使用) |

## Tour — リクエスト

1 つの Tour は 1 つの **HTTP リクエスト/レスポンス** に対応。HTTP/1.1 では Ship 上に Tour が 1 つずつ直列で流れ、HTTP/2 では並列で複数の Tour が同時進行します。

Tour にはライフサイクルがあります:

```
Reading req headers
    ↓
Reading req content
    ↓
(handler 実行)
    ↓
Sending res headers
    ↓
Sending res content
    ↓
End
```

Tour の最大数は Harbor の `maxTours` / Ship 単位の `maxToursPerShip` で制御。

## Agent — ワーカー

**Grand Agent** は BayServer のワーカースレッドです。`.plan` の `[harbor] grandAgents N` で個数を指定します。

各 Agent は自分の **Multiplexer** を持ち、エポックループを 1 スレッドで回します:

```
while (running) {
    events = multiplexer.poll()    // epoll_wait / kqueue / etc.
    for (event in events) {
        dispatch(event)            // 該当 Ship の I/O 処理を進める
    }
}
```

CPU コア数程度の Agent を用意すると、各コアが 1 つの Agent (= 1 スレッド) を回し、コア間でロックを取らずに大量の Ship を捌けます (= C10K 問題に対する BayServer のアプローチ)。

## Multiplexer

各 Agent が使う I/O 多重化エンジン。種類は [Multiplexer の役割](multiplexer.md) を参照。

## 性能との関係

- **`grandAgents`** が少なすぎると CPU を遊ばせる、多すぎると context switch コスト
- **`maxShips`** はメモリ使用量と上限接続数のトレードオフ
- **`maxToursPerShip`** は HTTP/2 / HTTP/3 で 1 接続から大量のリクエストを送られた際の保護

詳細: [パフォーマンス / Tuning](../performance/tuning.md)。

---

## 関連

- [BayServer とは / 用語表](../about.md#1) — Docker 名の比喩
- [Multiplexer](multiplexer.md) — I/O 多重化の選択肢
- [パフォーマンス](../performance/index.md) — tuning パラメータ
