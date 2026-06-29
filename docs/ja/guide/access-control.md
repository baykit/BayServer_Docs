# アクセス制限

BayServer のアクセス制限は **Permission Docker** で行います。IP/ホストによる制限、Basic 認証、グループ管理などをサポート。

## 配置できる場所

Permission Docker は以下にネストできます:

- `[port]` 配下 — そのポート全体に制限
- `[city]` 配下 — そのバーチャルホスト全体に制限
- `[town]` 配下 — そのパス区画に制限

ネスト箇所が深いほど制限の範囲が狭くなります。

## IP / ホストによる制限

### 特定の IP / CIDR を許可

```
[town /admin]
    [permission]
        admit 192.168.1.0/24
        admit 10.0.0.5
        refuse all
```

`admit` ルールにマッチしたら通す、最後の `refuse all` で他は全部拒否。

### 特定の IP を拒否

```
[town /]
    [permission]
        refuse 203.0.113.99
        admit all
```

`refuse` で特定 IP を弾き、それ以外は全部通す。

### ホスト名による制限

```
[permission]
    admit *.example.com
    refuse all
```

DNS 逆引きが効く環境なら使えます (= 逆引きが遅いとアクセスが遅延するので注意)。

## 評価順

ルールは記述された順に評価され、最初にマッチしたものが採用されます。`refuse` を後ろに書くと先に `admit` がマッチしてしまう場合があるので注意:

```
[permission]
    admit 192.168.1.0/24       # ← これにマッチすると refuse は無視
    refuse 192.168.1.99
```

特定の IP を弾きつつ範囲を許可したい場合は順序を逆に:

```
[permission]
    refuse 192.168.1.99        # 先に特定 IP を拒否
    admit 192.168.1.0/24
```

## Basic 認証

### 単一ユーザ

```
[town /admin]
    [permission]
        user admin password=secretpass
```

`user <name> password=<password>` でユーザ追加。プレーンテキストはセキュリティ的に弱いので bcrypt ハッシュを推奨:

```
[permission]
    user admin password=$2y$10$abc...xyz
```

### グループ

```
[permission]
    group editors
        user alice password=$2y$10$...
        user bob   password=$2y$10$...
    group admins
        user carol password=$2y$10$...
```

### IP 制限 + Basic 認証の併用

両方の条件をクリアしないと通れない:

```
[town /admin]
    [permission]
        admit 192.168.1.0/24
        refuse all
        user admin password=$2y$10$...
```

社内 IP からのみ、かつ Basic 認証必須。

## 配置パターン例

### 公開 + 管理画面だけ制限

```
[city *]
    [town /]
        location www/root        # 公開エリア

    [town /admin]
        location www/admin
        [permission]
            admit 192.168.1.0/24
            refuse all
            user admin password=$2y$10$...
```

### ポート全体に制限

```
[port 8080]
    [permission]
        admit 10.0.0.0/8         # 内部ネットワーク専用
        refuse all
    [city *]
        ...
```

## トラブルシューティング

| 症状 | 確認 |
|---|---|
| 全部 403 になる | `refuse all` のみで `admit` が無い、または順序問題 |
| Basic 認証が出ない | Permission Docker が正しい場所にネストされているか |
| ホスト名指定が効かない | DNS 逆引き設定、`/etc/hosts` 等 |

---

## 関連

- [Docker 種別 / Permission Docker](../reference/plan-reference.md)
- [HTTPS / TLS の設定](https.md) — Basic 認証は HTTPS と併用推奨
