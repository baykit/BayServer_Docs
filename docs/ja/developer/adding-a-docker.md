# 新しい Docker を書く

BayServer は Docker (= 設定ブロック) を **抽象クラスの拡張** で追加できる設計です。`.plan` で `[club hogeWarp]` のような自作 Docker を使えるようにする方法。

## どんなときに新規 Docker を書くか

- 既存の Club Docker (= file / cgi / php / servlet / *Warp 等) では足りない処理を入れたい
- 独自プロトコルを処理したい
- 特殊な認証 / 認可ロジックを Permission Docker 拡張で実現したい

既存の Docker (= WordPress Docker など) で目的が達せるなら、それを使う方が保守コストが低いです。

## 拡張ポイント

| 親クラス (= Java の場合) | 拡張内容 |
|---|---|
| `ClubBase` | 拡張子 / パスハンドラ (= file 等の代替) |
| `WarpBase` | リバースプロキシの上流接続パターン |
| `PortBase` | 新規プロトコル待受 |
| `PermissionBase` | カスタム認証ロジック |
| `LogBase` | 独自ログフォーマッタ |
| `RerouteBase` | URL 書き換え |

他言語版にもそれぞれ対応する基底クラスがあります (= 命名は同一)。

## Java 版で Club Docker を書く例

`*.foo` というパターンに反応する独自 Docker を作るとします。

### 1. クラスを書く

```java
package com.example.bayserver.docker.foo;

import yokohama.baykit.bayserver.docker.base.ClubBase;
import yokohama.baykit.bayserver.bcf.BcfElement;
import yokohama.baykit.bayserver.tour.Tour;

public class FooDocker extends ClubBase {

    @Override
    public void init(BcfElement elm, Docker parent) throws ConfigException {
        super.init(elm, parent);
        // .plan の独自パラメータがあればここで読む
    }

    @Override
    public void arrive(Tour tour) throws HttpException {
        // .foo へのリクエストはここに来る
        tour.res.headers.setContentType("text/plain");
        tour.res.sendResContent(tour.id(), "Hello, foo!".getBytes(), null);
        tour.res.endResContent(tour.id(), null);
    }
}
```

### 2. dockers.bcf に登録

`modules/bayserver/src/main/resources/conf/dockers.bcf` に追加:

```
club:foo    com.example.bayserver.docker.foo.FooDocker
```

### 3. ビルドして配置

```bash
mvn package
cp target/example-foo-1.0.jar <BayServerホーム>/lib/
```

### 4. `.plan` で使う

```
[city *]
    [town /]
        [club *.foo]
            docker foo
```

`/path/to/x.foo` で「Hello, foo!」が返れば成功。

## Tour ライフサイクル

Club Docker の `arrive()` が呼ばれた時点で `tour.req` は読み終わっている (= ヘッダ完了)。リクエストボディを読みたい場合は別途 `onReadReqContent` を実装します。

レスポンスを返す流れ:

```java
public void arrive(Tour tour) throws HttpException {
    tour.res.headers.setStatus(200);
    tour.res.headers.setContentType("application/json");
    tour.res.headers.setContentLength(body.length);
    tour.res.sendResHeaders(tour.id());        // ヘッダ送信
    tour.res.sendResContent(tour.id(), body, null);  // ボディ
    tour.res.endResContent(tour.id(), null);   // 完了
}
```

長いボディは複数回 `sendResContent` を呼べる (= chunked transfer)。最後の chunk で listener を渡すと送信完了通知が来ます。

## 非同期処理

Tour の処理を別スレッドや待機状態にする場合は `Train` / `Taxi` インフラを使います。

```java
// Train に積んで delay 実行
TrainRunner.post(tour, agentId, () -> {
    // 別の Taxi スレッドで動く
    // 完了後に tour.res.* を呼ぶ
});
```

詳細は [内部実装の地図](internals.md) で。

## 注意点

- **スレッドモデル** — Tour の処理は基本同一 Agent (= スレッド) 内で完結。別スレッドに渡すなら必ず Letter 経由 or 同期化
- **エラーハンドリング** — `HttpException` で投げると BayServer が標準エラーページを返す。`Sink` は致命的 (= プロセス終了)
- **メモリリーク** — pool から rent したものは必ず Return する

## 他言語版

Ruby / Python / PHP / TypeScript / Go 版でも同じ拡張パターンが使えます。基底クラス / モジュールの名前は各言語の慣習に合わせて変わります。

---

## 関連

- [ビルド方法](building.md)
- [コーディングスタイル](coding-style.md)
- [内部実装の地図](internals.md)
- [Docker 種別](../reference/docker-types.md)
