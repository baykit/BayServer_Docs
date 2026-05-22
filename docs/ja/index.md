# BayServer ドキュメント

**BayServer** は「**簡単**」「**軽量**」「**高速**」を設計目標にした、オープンソースの Web / Proxy サーバです。本サイトでは BayServer の概念、使い方、運用、性能 tuning、開発情報を集約しています。

製品の概要は [**BayServer とは**](about.md) から。初めての方は [**チュートリアル**](getting-started/tutorial/index.md) を順に進めるのもおすすめです。

## どこから読む?

<div class="grid cards" markdown>

-   :material-anchor: __はじめに__

    インストールから最初の Hello World、3 部構成のチュートリアルまで。

    [→ 入門ページ](getting-started/index.md)

-   :material-compass: __ガイド__

    静的配信、リバースプロキシ、HTTPS / HTTP/3、アクセス制限、移行ガイド。

    [→ ガイドへ](guide/index.md)

-   :material-clipboard-list: __実例集__

    Rails / WordPress 等の完成形 `.plan` レシピ。

    [→ 実例集へ](examples/index.md)

-   :material-book-open-page-variant: __リファレンス__

    `.plan` 文法、用語集、Docker 種別の正確な仕様。

    [→ リファレンスへ](reference/index.md)

-   :material-sail-boat: __仕組みを知る__

    Tour / Ship / Agent モデル、設計の意図。

    [→ 仕組みを知る](architecture/index.md)

-   :material-speedometer: __パフォーマンス__

    計測指針、tuning、Tips。

    [→ パフォーマンス](performance/index.md)

</div>

## 言語実装ごとの差分

BayServer は同じプラン記法を、異なる言語実装で動かすことを目指しています。ランタイム固有の挙動 / 制約 / tuning ポイントは下記から:

<div class="grid cards" markdown>

-   :fontawesome-brands-java: __Java__

    JDK 1.8+、Servlet 内蔵、HTTP/2 + HTTP/3。

    [→ Java 版](languages/java.md)

-   :fontawesome-brands-php: __PHP__

    PHP 7.4+、composer 配布、WordPress 連携。

    [→ PHP 版](languages/php.md)

-   :material-language-ruby: __Ruby__

    Ruby 2.7+、Rack 内蔵 (Rails / Sinatra)。

    [→ Ruby 版](languages/ruby.md)

-   :fontawesome-brands-python: __Python__

    Python 3.7+、WSGI 内蔵、HTTP/3 対応。

    [→ Python 版](languages/python.md)

-   :material-nodejs: __TypeScript__

    Node.js v16+、npm 配布。

    [→ TypeScript 版](languages/typescript.md)

-   :material-language-go: __Go__

    goroutine ベース、単一バイナリ配布。

    [→ Go 版](languages/go.md)

</div>

## より深く

<div class="grid cards" markdown>

-   :material-toolbox: __開発者向け__

    ビルド、コーディング規約、Docker 拡張の書き方、内部実装の地図。

    [→ 開発者向け](developer/index.md)

-   :material-newspaper: __What's New__

    主要バージョンの更新情報。

    [→ What's New](whats-new.md)

-   :material-help-circle: __FAQ__

    よくある質問、トラブルシューティング。

    [→ FAQ](faq.md)

</div>

---

!!! note "本サイトは整備中です"
    既存の [https://baykit.yokohama](https://baykit.yokohama) の内容を順次取り込み、より詳細なドキュメントに置き換えていきます。
