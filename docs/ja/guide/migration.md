# Apache / Nginx / Tomcat からの移行

既存サーバから BayServer への乗り換え時、設定ファイルの考え方の対応表です。

## 用語対応表

| 概念 | Apache | Nginx | Tomcat | **BayServer** |
|---|---|---|---|---|
| サーバ全体 | (= `httpd.conf` の root) | `http { }` | `<Server>` | **`[harbor]`** |
| ポート | `Listen 80` | `listen 80;` | `<Connector port="80">` | **`[port 80]`** |
| TLS | `<VirtualHost _:443>` + `SSLEngine on` | `listen 443 ssl;` | `<Connector secure="true">` | **`[port 443] [secure] key cert`** |
| バーチャルホスト | `<VirtualHost>` | `server { server_name }` | `<Host name="...">` | **`[city <hostname>]`** |
| URL パス | `<Location /admin>` | `location /admin { }` | `<Context path="/admin">` | **`[town /admin]`** |
| 静的ファイル配信 | `DocumentRoot` | `root` | `docBase` | **`location` (= Town docker のパラメータ)** |
| 拡張子ハンドラ | `AddType` / `SetHandler` | `location ~ \.php$ { }` | `<Servlet>` mapping | **`[club *.php] docker php`** |
| IP 制限 | `Require ip` | `allow / deny` | `<Valve>` | **`[permission] admit / refuse`** |
| Basic 認証 | `AuthType Basic` + `htpasswd` | `auth_basic` | `<security-constraint>` | **`[permission] user ... password=`** |
| リバースプロキシ | `ProxyPass` | `proxy_pass` | (= AJP コネクタ) | **`[club *] docker httpWarp/ajpWarp/fcgiWarp`** |
| URL 書換 | `RewriteRule` | `rewrite` | `RewriteValve` | **`[reroute]`** |
| アクセスログ | `CustomLog` | `access_log` | `<AccessLogValve>` | **`[log <path>]`** |
| エラーページ | `ErrorDocument` | `error_page` | `<error-page>` | **`[trouble]`** |
| CGI | `ScriptAlias` | `fastcgi_pass`+php-fpm 経由 | (= CGI Servlet) | **`[club *.cgi] docker cgi`** |

## Apache からの移行

### 典型的な `httpd.conf` の例:

```apache
Listen 80
<VirtualHost *:80>
    ServerName www.example.com
    DocumentRoot /var/www/example
    DirectoryIndex index.html

    <Location /admin>
        Require ip 192.168.1.0/24
    </Location>

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php-fpm.sock|fcgi://localhost"
    </FilesMatch>

    CustomLog /var/log/access.log combined
</VirtualHost>
```

### 同等の BayServer `.plan`:

```
[harbor]
    charset UTF-8

[port 80]

[city www.example.com]
    [town /]
        location /var/www/example
        index    index.html
        [club *.php]
            docker fcgiWarp
            destCity :unix:/run/php/php-fpm.sock

    [town /admin]
        [permission]
            admit 192.168.1.0/24
            refuse all

[log /var/log/access.log]
    format %h %l %u %t "%r" %>s %b
```

ポイント:

- **`<VirtualHost>` → `[city]`**、**`<Location>` → `[town]`**
- 全部 1 つの `.plan` に並べる (= `httpd.conf` のセクション順に近い)
- インデントで親子関係を表現 (= `<...>...</...>` の閉じタグ不要)

## Nginx からの移行

### 典型的な `nginx.conf` の例:

```nginx
http {
    server {
        listen 80;
        server_name www.example.com;
        root /var/www/example;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            fastcgi_pass unix:/run/php/php-fpm.sock;
            include fastcgi_params;
        }

        location /admin {
            allow 192.168.1.0/24;
            deny all;
        }

        access_log /var/log/access.log;
    }
}
```

### 同等の BayServer `.plan`:

```
[harbor]
    charset UTF-8

[port 80]

[city www.example.com]
    [town /]
        location /var/www/example
        index    index.html
        [club *.php]
            docker fcgiWarp
            destCity :unix:/run/php/php-fpm.sock

    [town /admin]
        [permission]
            admit 192.168.1.0/24
            refuse all

[log /var/log/access.log]
    format %h %l %u %t "%r" %>s %b
```

ポイント:

- **`server` ブロック → `[city]`**、**`location` ブロック → `[town]`**
- `fastcgi_pass` の Unix socket は `destCity :unix:/path/to/sock` の形式に
- `allow / deny` は `admit / refuse` (= 意味は同じ、評価順も同じ)

## Tomcat からの移行

Tomcat の `server.xml` + `web.xml` (= Servlet 仕様) に対して、BayServer は `.plan` + 既存の `web.xml` の組合せ。

### 典型的な `server.xml` (= 抜粋):

```xml
<Server port="8005">
    <Service name="Catalina">
        <Connector port="8080" protocol="HTTP/1.1"/>
        <Connector port="8443" SSLEnabled="true" ...>
            <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol"/>
        </Connector>
        <Engine defaultHost="localhost">
            <Host name="localhost" appBase="webapps"/>
        </Engine>
    </Service>
</Server>
```

### 同等の BayServer (= Java 版) `.plan`:

```
[harbor]
    charset UTF-8

[port 8080]

[port 8443]
    enableH2 on
    [secure]
        key  cert/server.key
        cert cert/server.crt

[city *]
    [town /]
        location webapps/ROOT
        [club *.do]
            docker servlet
        [club *.jsp]
            docker servlet
```

ポイント:

- `<Connector>` → `[port]`
- HTTP/2 化は `enableH2 on` 1 行 + Secure Docker
- `<Host appBase="webapps">` → city の `town` 配下に展開
- **`web.xml` はそのまま使える** (= Servlet 仕様準拠)、`<servlet-mapping>` も生きる

## 並行運用 (= 既存サーバの前に BayServer を挟む)

Apache / Nginx を急に置き換えるのが怖い場合、まず **フロントに BayServer、バックに既存サーバ** という構成にしてリスクを下げられます:

```
ブラウザ → BayServer (TLS 終端 / リバースプロキシ) → Apache (= 既存設定そのまま)
```

```
[port 443]
    [secure]
        key  /etc/letsencrypt/live/example.com/privkey.pem
        cert /etc/letsencrypt/live/example.com/fullchain.pem

[city *]
    [town /]
        [club *]
            docker httpWarp
            destCity localhost
            destPort 80         # = 既存 Apache をローカルで動かしたまま
```

実績が積めたら段階的に静的配信や動的アプリ実行を BayServer 側に移していきます。

---

## 関連

- [`.plan` 文法](../reference/plan-syntax.md)
- [リバースプロキシ](reverse-proxy.md)
- [HTTPS / TLS の設定](https.md)
- [なぜ BayServer か](../architecture/why-bayserver.md) — 他サーバとの設計比較
