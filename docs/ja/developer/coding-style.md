# コーディングスタイル

BayServer 全実装で共通する作法と、各言語実装ごとの慣習。

## 共通原則

### 命名は港湾の比喩を踏襲

クラス名 / メソッド名は BayServer の比喩 (= port / city / town / club / ship / tour / agent / docker) に合わせる。

```java
// good
class WarpShipStore { ... }
class GrandAgent { ... }

// bad
class ProxyConnectionPool { ... }    // BayServer 用語じゃない
class WorkerThreadManager { ... }
```

### ロックを最小化

各 Grand Agent は **自スレッド内で完結** するのが基本。スレッド間でデータを渡す場合は `Letter` を使う。

```java
// good
agt.sendLetter(new ReadLetter(rudder, buf));

// bad
synchronized (sharedState) { ... }   // 必要なときだけ
```

### Object pool を活用

頻繁に作るオブジェクト (= Tour, Ship, Command 等) は **Store** クラスで pool する。

```java
Tour tour = TourStore.getStore(agt.agentId).rent();
// ... use tour ...
TourStore.getStore(agt.agentId).Return(tour);
```

### Reset → Reuse パターン

pool から rent したオブジェクトは `reset()` で前回状態をクリアしてから使う。

### バッファコピーを避ける

`ByteBuffer.wrap()` で参照だけ渡す、`System.arraycopy()` も最小限。

## Java 版

- **`javax.servlet`** (5+ なら `jakarta.servlet`) のクラスを除き、外部ライブラリ依存ゼロ
- インデント 4 スペース
- ブレースは Java 標準 (Egyptian / K&R)
- ロガーは `BayLog` (= 独自実装) を使う、`System.out.println` 禁止
- 例外は `IOException` / `HttpException` / `Sink` (= 内部 unrecoverable)
- 文字列フォーマットは `String.format` ではなく `BayLog.info("Status: %d", code)` のような可変引数版

```java
public void someMethod(int arg) throws IOException {
    BayLog.debug("called with %d", arg);
    // ...
}
```

## Ruby 版

- Ruby スタイルガイド準拠 (= スネークケース)
- インデント 2 スペース
- `puts` ではなく `BayLog.debug` 等を使う

## Python 版

- PEP 8 準拠
- インデント 4 スペース
- ロガーは標準の `logging` モジュールではなく `BayLog` を使う

## PHP 版

- PSR-12 準拠
- ネームスペース: `Baykit\Bayserver`

## TypeScript 版

- TypeScript 標準 (= camelCase 関数 / メソッド、PascalCase クラス)
- インデント 2 スペース
- `prettier` + `eslint`

## C 版

- K&R スタイル
- インデント 4 スペース
- `snake_case` 関数

## コミットメッセージ

各リポジトリの `CONTRIBUTING.md` (= ある場合) に従う。基本:

- 1 行サマリ (50 文字以下)、英語
- 本文は WHY を中心に (= WHAT は diff で読める)
- `Co-Authored-By:` 等は不要

```
fix: H1 listener uses Transfer-Encoding: chunked when no Content-Length

Without this, the response was either truncated at the EOF marker
or padded by the framework's auto-length detection, depending on
the protocol handler. Make it explicit so chunked is used uniformly.
```

---

## 関連

- [ビルド方法](building.md)
- [新しい Docker を書く](adding-a-docker.md)
