# 実例集 (= Examples / Recipes)

実際のアプリケーションを BayServer で動かす **完成形のレシピ** 集。[ガイド](../guide/index.md) が機能単位の How-to なのに対し、ここは **シナリオ単位の end-to-end 設定**を載せます。

## レシピ一覧

- [Rails アプリをデプロイする](rails.md) — Rack 経由で Ruby on Rails をフロント + 静的アセット最適化
- [WordPress を完全構成で動かす](wordpress-full.md) — PHP-FPM + FCGI Warp + TLS + 静的 uploads 最適化
- Django + Gunicorn の前に立てる *(TBD)*
- Servlet アプリ + Tomcat の代わり *(TBD)*
- 静的サイト + CDN front (= keep-alive 設計) *(TBD)*

## このセクションの位置付け

| 章 | 性格 | 例 |
|---|---|---|
| [チュートリアル](../getting-started/tutorial/index.md) | 学習用、1 つを完成させる連続記事 | 静的サイト → HTTPS → プロキシ |
| [ガイド](../guide/index.md) | 機能単位の How-to | HTTPS の設定方法、Permission の使い方 |
| **実例集 (ここ)** | **シナリオ単位の完成形** | **Rails アプリ全部入り** |
| [リファレンス](../reference/index.md) | 仕様 | `.plan` 文法、各 Docker のパラメータ |

「自分のスタックに近い例」を探して、`.plan` をコピペして調整、というのが想定される使い方です。
