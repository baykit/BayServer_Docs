# What's New (= 主要更新)

BayServer の主要バージョンの追加機能・変更点の早見表。完全なリリースノートは [baykit.yokohama](https://baykit.yokohama/) の各リリース告知や、各実装の GitHub Releases ページを参照してください。

## Version 4 (= 開発中)

開発中のため未発表。リリース後に主要差分をここに掲載予定。

リファレンス整備状況: [Docker 種別 (V4 以降)](reference/docker-types.md) (= 整備中スタブ)

## Version 3 系 (= 現行)

| バージョン | 主な変更 |
|---|---|
| 3.3.2 | (= Java / Ruby / PHP 等、各言語版で安定化リリース) |
| 3.3.x | I/O 多重化の選択肢を拡張、Direct Boarding / Barge Docker などの性能機構 |
| 3.2.x | HTTP/2 CONTINUATION フレーム対応など |
| 3.1.x | PigeonMultiplexer の安定化、設定パーサ強化 |
| 3.0.x | **Version 3 系の初版**。大規模リファクタリング、Multiplexer 概念の整理 |

V3 系の Docker / `.plan` 仕様は [リファレンス V3 (凍結)](reference/docker-types-v3.md) を参照。

## Version 2 系 (= 旧)

「簡単」「軽量」「高速」を掲げて Version 1 からゼロベースで作り直した版。V3 以前との互換性は基本維持されています。

## Version 1 系 (= 初期)

「XML アプリケーションが簡単に開発できる」を売りにしていた初期版。現行とは別物。

---

## 各実装のリリース履歴

GitHub Releases / RSS を参照:

- [BayServer for Java](https://github.com/baykit/BayServer_Java/releases)
- [BayServer for Ruby](https://github.com/baykit/BayServer_Ruby/releases)
- [BayServer for Python](https://github.com/baykit/BayServer_Python/releases)
- [BayServer for PHP](https://github.com/baykit/BayServer_PHP/releases)
- [BayServer for TypeScript](https://github.com/baykit/BayServer_TypeScript/releases)
- [BayServer for Go](https://github.com/baykit/BayServer_Go/releases)
- [BayServer for C](https://github.com/baykit/BayServer_C/releases)

公式アナウンス:

- [baykit.yokohama / News](https://baykit.yokohama/)

---

## このページの位置付け

「**今のバージョンで何が変わったか**」をすばやく確認したいユーザ向けの目次です。詳細仕様は [リファレンス](reference/index.md) を参照してください。
