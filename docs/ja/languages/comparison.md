# 言語実装別 機能比較表

BayServer は Java / Ruby / Python / PHP / TypeScript (Node.js) / Go / C の各言語で実装されています。基本機能は揃えていますが、ランタイムの違いから一部の機能で差があります。

## プロトコル (サーバ機能)

クライアントから接続を **受ける** 側で対応するプロトコル:

| プロトコル | Java | Ruby | Python | PHP | TypeScript |
|---|:---:|:---:|:---:|:---:|:---:|
| HTTP/1.1 (plain) | ✓ | ✓ | ✓ | ✓ | ✓ |
| HTTP/1.1 (SSL) | ✓ | ✓ | ✓ | ✓ | ✓ |
| HTTP/2 (plain) | — | — | — | — | — |
| HTTP/2 (SSL) | ✓ | ✓ | ✓ | ✓ | ✓ |
| HTTP/3 | ✓ | — | ✓ | — | — |
| AJP | ✓ | ✓ | ✓ | ✓ | ✓ |
| FCGI | ✓ | ✓ | ✓ | ✓ | ✓ |

## プロトコル (プロキシ機能)

リバースプロキシとして **上流に接続する** 側で対応するプロトコル:

| プロトコル | Java | Ruby | Python | PHP | TypeScript |
|---|:---:|:---:|:---:|:---:|:---:|
| HTTP/1.1 (plain) | ✓ | ✓ | ✓ | ✓ | ✓ |
| HTTP/1.1 (SSL) | ✓ | ✓ | ✓ | ✓ | ✓ |
| HTTP/2 (plain) | — | — | — | — | — |
| HTTP/2 (SSL) | — | — | — | — | — |
| HTTP/3 | — | — | — | — | — |
| AJP | ✓ | ✓ | ✓ | ✓ | ✓ |
| FCGI | ✓ | ✓ | ✓ | ✓ | ✓ |

## その他の機能

| 機能 | Java | Ruby | Python | PHP | TypeScript |
|---|:---:|:---:|:---:|:---:|:---:|
| UNIX ドメインソケット | ✓ (※1) | ✓ | ✓ | ✓ | ✓ |
| マルチコアモード | ✓ | ✓ | ✓ | ✓ (※2) | ✓ |
| シングルコアモード | — | ✓ | ✓ | ✓ (※3) | — |
| Basic 認証 | ✓ | ✓ | ✓ | ✓ | ✓ |
| 共通エラーページ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Servlet API | ✓ | — | — | — | — |
| Rack API | — | ✓ | — | — | — |
| WSGI API | — | — | ✓ | — | — |

(※1) JDK 16 以上が必要
(※2) Windows プラットフォームでは未サポート (Cygwin/MinGW 版なら可能性あり)
(※3) エージェント数 (スレッド数) は 1

## 推奨用途

- **Java** — Servlet アプリ (war ファイル) を動かしたい、JVM のチューニングを活用したい
- **Ruby** — Rails / Sinatra など Rack アプリのフロント / サーバ
- **Python** — Django / Flask など WSGI アプリのフロント / サーバ
- **PHP** — WordPress などの PHP アプリのフロント
- **TypeScript (Node.js)** — Node エコシステム内に統合したい場合
- **Go / C** — 最軽量、組込、ネイティブビルドが必要な場合

---

## 関連

- [Java 版](java.md)
- [Ruby 版](ruby.md)
- [Python 版](python.md)
- [PHP 版](php.md)
- [TypeScript 版](typescript.md)
- [C 版](c.md)
