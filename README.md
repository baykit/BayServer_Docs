# BayServer_Docs

[BayServer](https://baykit.yokohama) ドキュメントの集約リポジトリ。

- 全言語実装 (Java / PHP / Ruby / Python / C) 共通の概念・設定・運用・Tips を一元管理
- 日本語 (canonical) と英語 (= 後追い翻訳) の 2 言語
- [MkDocs](https://www.mkdocs.org/) + [Material](https://squidfunk.github.io/mkdocs-material/) で静的サイト化

## ローカルでプレビュー

```bash
# 1. pip を bootstrap (Ubuntu で python3-pip 未導入の場合)
sudo apt install -y python3-pip python3-venv   # or curl -sS https://bootstrap.pypa.io/get-pip.py | python3

# 2. venv + deps
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt

# 3. dev server
.venv/bin/mkdocs serve
# → http://127.0.0.1:8000/
```

## 構成

```
docs/
├── ja/            # 日本語 (canonical) — まずここを書く
│   ├── index.md
│   ├── getting-started/  # Tutorials (= 入門)
│   ├── guide/            # How-to (= 課題解決)
│   ├── reference/        # Reference (= 仕様)
│   ├── architecture/     # Explanation (= 概念)
│   ├── languages/        # 言語実装ごとの差分
│   ├── performance/      # 性能 tuning + Tips
│   ├── developer/        # BayServer 自体の開発者向け
│   └── faq.md
└── en/            # English (= 日本語が固まったら同構造で複製・翻訳)
```

セクション分類は [Diátaxis](https://diataxis.fr/) を踏襲 (= Tutorial / How-to / Reference / Explanation を混ぜない)。

## 翻訳ワークフロー

1. 日本語版を書く / 更新する
2. 内容が固まったら `docs/en/<同じパス>.md` に翻訳版を作る
3. 翻訳の first-pass は LLM で生成、人がレビュー
4. 未翻訳ページは `mkdocs-static-i18n` の fallback で日本語版が表示される (= リンク切れにはならない)

## デプロイ

GitHub Pages への自動デプロイ workflow は未着手 (= フレームワーク + 初期コンテンツが揃ってから追加予定)。
