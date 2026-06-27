# 言語実装ごとの差分

BayServer は複数のプログラミング言語実装で書かれており、利用シーンに応じて選べます。ランタイムの違いから、言語ごとに固有の挙動 / 制約 / tuning ポイントがあります。

汎用的な内容 (= プラン文法、Docker 種別、運用方法) は [リファレンス](../reference/index.md) と [ガイド](../guide/index.md) を参照。本セクションでは「Java 版だけここが違う」「PHP 版だけここが違う」といった**実装固有の差分**のみを扱います。

## 各実装

- [Java](java.md) — Servlet コンテナ内蔵、JVM チューニング、HTTP/2 / HTTP/3
- [PHP](php.md) — composer 配布、WordPress 連携
- [Ruby](ruby.md) — Rack サーバ内蔵 (Rails / Sinatra)
- [Python](python.md) — WSGI サーバ内蔵 (Django / Flask)、HTTP/3 対応
- [TypeScript (Node.js)](typescript.md) — npm 配布
- [Go](go.md) — ネイティブビルド、単一バイナリ配布

## 機能対応表

実装ごとのプロトコル対応 / 機能差を一覧で見るには [機能比較表](comparison.md) を参照。
