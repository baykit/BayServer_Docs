# BayServer ドキュメント

**BayServer** は、同じ `.plan` (= 設定ファイル) で複数言語実装 (Java / PHP / Ruby / Python / C) を切り替えられる高性能 Web / Proxy サーバです。本サイトでは BayServer の概念、使い方、運用、性能 tuning、開発情報を集約しています。

## まずはここから

- [はじめに](getting-started/) — インストールから Hello World まで
- [ガイド](guide/) — 静的ファイル / リバースプロキシ / HTTPS / 監視 などの How-to
- [リファレンス](reference/) — `.plan` 文法、docker 種別、CLI など仕様
- [アーキテクチャ](architecture/) — Tour / Ship / Agent の関係、設計の意図

## 言語実装ごとの差分

BayServer は同じプラン記法を異なる言語実装で動かせます。ランタイム固有の挙動・制約・tuning ポイントは [言語別](languages/) を参照:

- [Java](languages/java.md)
- [PHP](languages/php.md)
- [Ruby](languages/ruby.md)
- [Python](languages/python.md)
- [C](languages/c.md)

## より深く

- [パフォーマンス](performance/) — 計測指針、tuning、Tips
- [開発者向け](developer/) — BayServer への contribute、内部設計

---

!!! note "本サイトは整備中です"
    既存の [https://baykit.yokohama](https://baykit.yokohama) の内容を順次取り込み、より詳細なドキュメントに置き換えていきます。
