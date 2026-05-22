# パフォーマンス Tips

実運用で効きやすい、小さめのチューニングノウハウ集。[Tuning ガイド](tuning.md) と合わせて参照してください。

## 計測前に必ずやること

1. **アクセスログを最小限に** — `logLevel warn` (= info 以下は本番では無効化推奨)
2. **debug ビルド / debug 拡張を外す** — 言語ごとに違うが、PHP の Xdebug、Python の debug build 等
3. **opcache / JIT を有効に** — PHP の OPcache、Java は JIT (= 起動後 30 秒程度のウォームアップが必要)
4. **CPU governor を performance に** — `cpufreq-set -g performance` (Linux)

## ベンチ実施時

- **ウォームアップ 3-5 秒** を必ず取る (= JIT / connection pool / disk cache の起動)
- 計測時間は **15-30 秒** が標準
- **エラー件数を必ず提示** (= 完走 RPS だけだとタイムアウト中の見落としが起きる)
- 同じシナリオを **3-6 回** 取って中央値を採る

詳細: [ベンチマーク方法論](benchmarks.md)。

## 静的ファイル配信

### Barge cache を効かせる

```
[city *]
    [town /]
        location www/root
        [barge]
            capacity 100M    # 100 MB のメモリキャッシュ
```

`/wp-content/uploads/` のような頻繁アクセスの静的領域に置くだけで、ディスク I/O が消えて RPS が数倍。

### maxCargoSize を上げる

```
[harbor]
    maxCargoSize 1100000
```

`maxCargoSize` 以下のファイルだけ cache に入るため、適切に上げないと中サイズファイルが cache に乗らない。

### sendfile path を活用

`directBoarding on` で大サイズファイルが kernel-space で送信される。

## リバースプロキシ用途

### UNIX ドメインソケットを使う

ローカル接続 (= BayServer + PHP-FPM on same host) は TCP より UDS の方が高速:

```
[club *.php]
    docker fcgiWarp
    destCity :unix:/run/php/php8-fpm.sock
```

### keep-alive を維持

上流接続を維持するため `maxShips` を上げる:

```
[club *]
    docker httpWarp
    destCity backend.internal
    destPort 8080
    maxShips 100
```

### Nagle / delayed-ACK 問題

上流が PHP-FPM 等で Nagle が効いていると、レスポンス末尾の小さなセグメントが 40ms 遅延することがあります。BayServer は warp 接続に **TCP_QUICKACK を自動 re-arm** するため、通常は気にしなくて済みますが、最新のソースを使っているか確認。

## HTTP/2 多重化

```
[port 443]
    enableH2 on
    [harbor 等で]
    maxToursPerShip 200
```

H2 で同じ接続から 100+ の Tour が並列で来るので、`maxToursPerShip` を上げないとスロットリングされます。

## メモリ管理

### Object pool (= ObjectStore)

BayServer は内部で大量のオブジェクトを **ObjectStore** で pool しています。これは BayServer 側の最適化で、ユーザ側でできる調整はあまりありませんが、ヒープサイズの設定 (= JVM の `-Xmx`) は重要。

### GC

Java 版: **G1GC** が推奨 (= 21+ のデフォルト)。ZGC は BayServer のワークロードに合わず実測で遅い。

```bash
BSERV_OPT="-Xmx4g -XX:+UseG1GC" bin/bayserver.sh -start
```

## ログ周りの最適化

### access log を完全に切る

```
# Log Docker を .plan から削除するだけ
```

性能比較や負荷テストでは access log を OFF にして「ピュアな転送性能」を計測。

### error log だけ残す

致命的エラーだけ拾いたい場合は logLevel を `error` に:

```
[harbor]
    logLevel error
```

## SSL の最適化

### Session resumption

TLS セッション再利用が効くように、サーバ間でセッションキャッシュを揃える (= 詳細は実装依存)。

### HTTP/2 を併用

TLS handshake のコストを 1 接続で多数の Tour に分散できるため、SSL ありなら H2 / H3 必須レベル:

```
[port 443]
    enableH2 on
    enableH3 on
    [secure]
        key  cert/server.key
        cert cert/server.crt
```

## NUMA

複数 NUMA ノードのマシンでは `numactl --cpunodebind=0 --membind=0 bin/bayserver.sh -start` で 1 ノードに閉じ込めた方が安定。これは BayServer 側ではなく OS レイヤの話。

---

## 関連

- [Tuning ガイド](tuning.md)
- [ベンチマーク方法論](benchmarks.md)
- [Multiplexer](../architecture/multiplexer.md)
