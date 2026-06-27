# なぜ BayServer か

Apache / Nginx が既に成熟している中で、BayServer がなぜ別の Web サーバとして存在するのか。設計上の選択肢を他の主要 Web サーバと比較して整理します。

## 設計の 3 つの目標

[BayServer とは](../about.md) で挙げた通り、設計目標は **簡単・軽量・高速** の 3 つです。これらをそれぞれ既存サーバと比べて見てみます。

## 簡単さ

| サーバ | 設定ファイル | 学習コスト |
|---|---|---|
| Apache | `httpd.conf` (XML 風、ディレクティブ多数) | 高 |
| Nginx | `nginx.conf` (中括弧と `directive value;`) | 中 |
| Tomcat | `server.xml` (XML)、`web.xml` (Servlet) | 高 |
| **BayServer** | `bayserver.plan` (ini 風、port/city/town/club の比喩) | **低** |

BayServer は **設定の比喩 (= 港湾)** を採用しています。port (港) / city (都市) / town (町) / club (クラブ) という直感的な区分で、設定ファイルが「何を設定しているか」が読み取りやすくなっています。

```
[city www.example.com]   # ホスト
    [town /admin]        # パス
        [club *.do]      # 拡張子ハンドラ
            docker servlet
```

## 軽量さ

| サーバ | 依存パッケージ | バイナリサイズ |
|---|---|---|
| Apache | apr / openssl / pcre / 各種モジュール | ~10MB+ |
| Nginx | openssl / pcre / zlib | ~1MB (本体) |
| Tomcat | Java + 各種 jar | ~10MB+ |
| **BayServer (Java 版)** | JDK のみ | **<1MB** |

BayServer は **各言語の標準 API + 最小限の外部パッケージ** で書かれており、重い依存ツリーは持ちません。

## 高速さ

| サーバ | 並列モデル | 性能特性 |
|---|---|---|
| Apache (prefork) | プロセス | 安定 / メモリ消費大 |
| Apache (worker) | スレッド | 中庸 |
| Nginx | event-driven (epoll) | 高性能 |
| Tomcat (NIO) | event-driven + thread pool | 中 |
| **BayServer (spider)** | event-driven (epoll/kqueue) | **高** |

BayServer は Nginx と同様にノンブロッキング I/O を採用しつつ、**マルチコアモードでは複数 Agent が並列にイベントループを回す** ことで CPU を限界まで使い切ります。

実測ベースの比較は [パフォーマンス / ベンチマーク](../performance/benchmarks.md) を参照。

## アーキテクチャ上の特徴

### Same-thread tour completion

1 つの Tour は、原則として **同じ Agent (= スレッド)** 内で開始から終了まで処理されます。これにより以下が達成できます:

- スレッド間ロックが最小化
- Cache locality が良い
- デバッグが追いやすい

### Protocol agnostic な内部構造

HTTP/1, HTTP/2, HTTP/3, AJP, FCGI を **同一の Tour/Ship 抽象** で扱えます。新しいプロトコルを追加する際も、Tour/Ship の API に合わせれば既存のミドルウェア (= Permission, Reroute, Trouble Docker 等) がそのまま使えます。

### 同一プラン記法での多言語実装

Java / Ruby / Python / PHP / TypeScript / Go 版が **同じ `.plan` ファイル** で動きます。アプリケーションの言語に合わせて選べる柔軟性は他のサーバには無い特徴です。

## どんなときに BayServer を選ぶか

| シナリオ | BayServer が向く理由 |
|---|---|
| Servlet アプリのフロント | Java 版なら Tomcat 代替可能 |
| Rack / WSGI アプリのフロント | Ruby / Python 版で直接実行可能 |
| 設定をシンプルに保ちたい | `.plan` の比喩で意図が読み取れる |
| 性能を絞り出したい | Nginx と同等以上の RPS、tuning 余地大 |
| Apache / Nginx より軽くしたい | 外部依存ゼロ |

## どんなときに他を選ぶか

- **WAF / モジュールエコシステムが必要** → Apache (= mod_security 等の成熟したモジュール)
- **大規模 Kubernetes 環境で Ingress 用途** → Nginx Ingress Controller / Envoy / Traefik
- **既存設定資産の流用** → 既に Apache/Nginx を運用中なら無理に置換しない

---

## 関連

- [BayServer とは](../about.md)
- [Tour / Ship / Agent](tour-ship-agent.md)
- [パフォーマンス](../performance/index.md)
