# BayServer ドキュメント

**BayServer** は、同じ `.plan` (= 設定ファイル) で複数言語実装 (Java / PHP / Ruby / Python / C) を切り替えられる高性能 Web / Proxy サーバです。本サイトでは BayServer の概念、使い方、運用、性能 tuning、開発情報を集約しています。

## どこから読む?

<div class="grid cards" markdown>

-   :material-anchor: __はじめに__

    インストールから最初の Hello World、言語実装の選択までを順を追って。

    [→ 入門ページ](getting-started/)

-   :material-compass: __ガイド__

    静的配信、リバースプロキシ、HTTPS / HTTP2 / HTTP3、監視、本番デプロイ。

    [→ ガイドへ](guide/)

-   :material-book-open-page-variant: __リファレンス__

    `.plan` 文法、docker 種別、CLI、設定キーの正確な仕様。

    [→ リファレンスへ](reference/)

-   :material-sail-boat: __アーキテクチャ__

    Tour / Ship / Agent の関係、Multiplexer の役割、設計の意図。

    [→ アーキテクチャへ](architecture/)

</div>

## 言語実装ごとの差分

BayServer は同じプラン記法を、異なる言語実装で動かすことを目指しています。ランタイム固有の挙動 / 制約 / tuning ポイントは下記から:

<div class="grid cards" markdown>

-   :fontawesome-brands-java: __Java__

    JDK 21+、JVM チューニング、Conscrypt、libphp 埋め込み (Pharos)。

    [→ Java 版](languages/java.md)

-   :fontawesome-brands-php: __PHP__

    phpenv、ext-event、ReactPHP / AmPHP / Workerman、PHPVerse。

    [→ PHP 版](languages/php.md)

-   :material-language-ruby: __Ruby__

    rbenv、Falcon / Puma / Iodine 比較、async IO。

    [→ Ruby 版](languages/ruby.md)

-   :fontawesome-brands-python: __Python__

    venv、uvloop、asgi 系との比較。

    [→ Python 版](languages/python.md)

-   :material-language-c: __C__

    ビルド、libevent、static binary。

    [→ C 版](languages/c.md)

</div>

## より深く

<div class="grid cards" markdown>

-   :material-speedometer: __パフォーマンス__

    計測指針、tuning、Tips。

    [→ パフォーマンス](performance/)

-   :material-toolbox: __開発者向け__

    ビルド、コーディング規約、Docker 拡張の書き方、内部実装の地図。

    [→ 開発者向け](developer/)

-   :material-help-circle: __FAQ__

    よくある質問、トラブルシューティング。

    [→ FAQ](faq.md)

</div>

---

!!! note "本サイトは整備中です"
    既存の [https://baykit.yokohama](https://baykit.yokohama) の内容を順次取り込み、より詳細なドキュメントに置き換えていきます。
