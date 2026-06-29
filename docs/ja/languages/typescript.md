# TypeScript 版 (Node.js) BayServer

BayServer for TypeScript は Node.js v16.19.0 以降で動作します。Node.js / TypeScript エコシステムと統合できます。

## インストール

```bash
npm install -g @baykit/bayserver
bayserver -init
bayserver
```

より詳しい手順・起動オプションは [インストール](../getting-started/install.md#typescript-nodejs) を参照。

## マルチコア / シングルコア

```
[harbor]
    multiCore on
    grandAgents 4
```

TypeScript 版は Node.js の cluster モジュール相当でワーカープロセスを spawn します (= マルチコアモード時)。

## HTTP/3 サポート

TypeScript 版は HTTP/3 を **未サポート** です (= 2026 時点)。HTTP/2 (TLS) は対応しています。

## 使用例

静的ファイル配信:

```
[city *]
    [town /]
        location www/root
        index    index.html
```

リバースプロキシ:

```
[city *]
    [town /api]
        [club *]
            docker httpWarp
            destCity backend.internal
            destPort 8080
            destTown /api
```

---

## 関連

- [インストール](../getting-started/install.md)
- [言語実装別 比較表](comparison.md)
- [リバースプロキシ](../guide/reverse-proxy.md)
