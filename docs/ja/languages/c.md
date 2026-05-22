# C 版 BayServer

BayServer for C はネイティブビルドで動作する実装です。最軽量・組込・依存最小の用途に向きます。

## インストール

ソースから build します:

```bash
git clone https://github.com/baykit/BayServer_C.git
cd BayServer_C
make
```

または配布バイナリを利用します (= Go 版と同様の単一バイナリ提供)。

## 使用例

基本的な構成は他の言語版と同じ:

```
[harbor]
    charset UTF-8
    grandAgents 4

[port 8080]

[city *]
    [town /]
        location www/root
        index    index.html
```

## 制約

- 内蔵アプリ実行エンジン (Servlet / Rack / WSGI 相当) は **無し**
- 主に静的配信 + リバースプロキシ用途
- マルチコアモード対応

## 用途

- リバースプロキシ専用サーバ (= フロント担当)
- 組込デバイス (= メモリフットプリント最小)
- 静的サイトホスティング

---

## 関連

- [インストール](../getting-started/install.md)
- [言語実装別 比較表](comparison.md)
- [リバースプロキシ](../guide/reverse-proxy.md)
