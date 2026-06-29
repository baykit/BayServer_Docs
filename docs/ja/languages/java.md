# Java 版 BayServer

BayServer for Java は JDK 1.8 以降 (OpenJDK 含む) で動作します。Servlet コンテナを内蔵しているため、既存の Servlet アプリ (war / classes) をそのまま動かせます。

## 主な特徴

- **Servlet API 対応** — Servlet 4.0 (Jakarta Servlet 5+ も対応見込み)
- **HTTP/2 / HTTP/3 対応** — TLS 上の HTTP/2 と HTTP/3 (QUIC) をサポート
- **JVM チューニング** — 標準的な `-Xmx` / GC オプションがそのまま使える
- **配布** — jar 配布

## インストール

[インストール](../getting-started/install.md#java) を参照（jar 配布が基本）。

```bash
jar xf BayServer_Java-X.Y.Z.jar
cd BayServer_Java-X.Y.Z
bin/bayserver.sh -start
```

## Servlet を使う

Servlet 用の Club Docker を `*.do` などの拡張子に紐づけ、`town` の `location` に WAR を展開したディレクトリを指定します。

### ディレクトリ構成

```
www/myapp/
├── WEB-INF/
│   ├── web.xml          # Servlet 構成
│   ├── classes/
│   │   └── com/example/MyServlet.class
│   └── lib/
│       └── *.jar
├── index.jsp
└── static/
    └── style.css
```

### `.plan` 設定例

```
[city *]
    [town /myapp]
        location www/myapp
        [club *.do]
            docker servlet
```

`*.do` の URL リクエストが Servlet にディスパッチされます (= `web.xml` の `<servlet-mapping>` に従う)。

### web.xml

Servlet 4.0 に準拠した標準的な記法:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         version="4.0">
    <servlet>
        <servlet-name>MyServlet</servlet-name>
        <servlet-class>com.example.MyServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>MyServlet</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```

### JSP

JSP も Servlet として動作します。`*.jsp` 用に Club を追加:

```
[club *.jsp]
    docker servlet
```

## HTTP/2 / HTTP/3 サポート

TLS 上で ALPN ネゴシエーション。`.plan` で `[port]` 内の `[secure]` ブロックを設定すれば、対応クライアントからは HTTP/2 / HTTP/3 で接続されます。

```
[port 2024]
    enableH2 on
    enableH3 on
    [secure]
        key  cert/server.key
        cert cert/server.crt
```

HTTP/3 のサーバ実装はネイティブ拡張に依存します ([HTTP/3 を有効化する](../guide/http3.md) 参照)。

## JVM オプション

`bayserver.sh` 経由で起動する場合、環境変数 `BSERV_OPT` で JVM オプションを渡せます:

```bash
BSERV_OPT="-Xmx2g -XX:+UseG1GC" bin/bayserver.sh -start
```

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| `java: command not found` | JDK インストール確認 (`java -version`) |
| HTTP/3 が起動しない | ネイティブライブラリの導入 ([HTTP/3 を有効化する](../guide/http3.md)) |

---

## 関連

- [インストール](../getting-started/install.md)
- [言語実装別 比較表](comparison.md)
- [`.plan` 文法](../reference/plan-syntax.md)
- [Docker 種別](../reference/docker-types.md)
