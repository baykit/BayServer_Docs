# Servlet を使う

Java 版 BayServer は **Servlet コンテナを内蔵** しているため、`.war` を展開した Servlet アプリをそのまま動かせます。Tomcat / Jetty の代替として使えます。

!!! note "Java 版限定"
    Servlet を直接動かせるのは **Java 版 BayServer のみ**。他言語版から AJP Warp で Tomcat 等にプロキシする構成については [リバースプロキシ](reverse-proxy.md) を参照。

## 前提

- Java 版 BayServer ([インストール](../getting-started/install.md))
- Servlet アプリ (= `.war` 展開済 または classes/lib 配置済)
- `web.xml` (Servlet 4.0 仕様)

## ディレクトリ構成

```
www/myapp/
├── WEB-INF/
│   ├── web.xml                      # Servlet 構成
│   ├── classes/
│   │   └── com/example/MyServlet.class
│   └── lib/
│       └── *.jar
├── index.jsp
└── static/
    └── style.css
```

## 最小構成

```
[city *]
    [town /myapp]
        location www/myapp
        [club *.do]
            docker servlet
```

`*.do` の URL に来たリクエストが `web.xml` の `<servlet-mapping>` に従って Servlet にディスパッチされます。

## web.xml

標準的な Servlet 4.0 記法:

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

## JSP

JSP も同じく Servlet として動作。`*.jsp` 用に Club を追加:

```
[city *]
    [town /myapp]
        location www/myapp
        [club *.do]
            docker servlet
        [club *.jsp]
            docker servlet
```

## 複数の URL パターンを束ねる

すべてのリクエストを Servlet 経由にしたい場合 (= Spring Boot のようなフレームワーク向け):

```
[club *]
    docker servlet
```

`*` ですべてキャッチ。静的ファイル配信を共存させたいなら、静的 town を先に置く:

```
[city *]
    [town /static]
        location www/myapp/static

    [town /]
        location www/myapp
        [club *]
            docker servlet
```

## TLS + HTTP/2 を併用

```
[port 8443]
    enableH2 on
    [secure]
        key  cert/server.key
        cert cert/server.crt

[city *]
    [town /]
        location www/myapp
        [club *]
            docker servlet
```

## war ファイルの取り扱い

BayServer は `.war` を **自動展開しません**。事前に手動で展開してから `location` で指す:

```bash
mkdir www/myapp
cd www/myapp
jar xf /path/to/myapp.war
```

## JVM オプション

```bash
BSERV_OPT="-Xmx2g -XX:+UseG1GC" bin/bayserver.sh -start
```

詳細: [Java 版](../languages/java.md), [パフォーマンス Tuning](../performance/tuning.md)。

## Servlet API バージョン

| BayServer for Java | サポート Servlet API |
|---|---|
| 2.x | `javax.servlet` 4.0 |
| 3.x | `javax.servlet` 4.0 (= `jakarta.servlet` への移行検討中) |

Jakarta Servlet (= namespace 移行後) への対応は将来予定。

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| ClassNotFoundException | `WEB-INF/classes/` または `WEB-INF/lib/` にクラス / jar があるか |
| 404 で Servlet が呼ばれない | `web.xml` の `<servlet-mapping>` と Club の URL パターンが整合 |
| Session が消える | `web.xml` の `<session-config>` 確認、cookie path 確認 |
| `javax` vs `jakarta` の混在エラー | アプリ側の依存を `javax.servlet-api 4.x` に統一 |
| 起動時に長時間止まる | アプリの `ServletContextListener` 内処理確認 (= DB 接続等で詰まることがある) |

---

## 関連

- [Java 版 BayServer](../languages/java.md)
- [リバースプロキシ](reverse-proxy.md) — Tomcat に AJP で繋ぐパターン
- [HTTPS / TLS の設定](https.md)
- [Docker 種別 / Servlet Docker (V3)](../reference/docker-types-v3.md)
