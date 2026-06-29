# Version 3 アーカイブ

!!! warning "凍結セクション — Version 3 以前専用"
    このセクションは **BayServer Version 3 以前** の情報のうち、Version 4 と異なる部分だけを集めた凍結アーカイブです。**今後の編集は行いません**。現行 (Version 4) のドキュメントは各セクションのトップから辿ってください。

Version 3 以前を使っていて、現行 (V4) と挙動が異なる項目を確認したい場合のみ、このアーカイブを参照してください。

## このアーカイブのページ

- [インストール (V3 以前)](install.md) — ダウンロード版 / パッケージ版の二分法による配布

## バージョン略史

- **Version 1** — 「XML アプリケーションが簡単に開発できる」を売りにしていた初期版
- **Version 2** — 「簡単」「軽量」「高速」をテーマにゼロから再設計
- **Version 3** — 大幅なリファクタリングを実施し、I/O 多重化の選択肢を拡張

| バージョン | 主な変更 |
|---|---|
| 3.3.x | I/O 多重化の選択肢を拡張、Direct Boarding / Barge Docker などの性能機構 |
| 3.2.x | HTTP/2 CONTINUATION フレーム対応など |
| 3.1.x | PigeonMultiplexer の安定化、設定パーサ強化 |
| 3.0.x | Version 3 系の初版。大規模リファクタリング、Multiplexer 概念の整理 |

## その他

- 各実装の正確なリリース履歴は [GitHub Releases](https://github.com/baykit) を参照してください。
- V3 以前の総合的な情報は、旧サイト [baykit.yokohama](https://baykit.yokohama/) にも残っています。
