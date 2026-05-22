# ベンチマーク方法論

BayServer の性能を再現性のある形で計測するためのガイド。Apache / Nginx 等との比較や、設定変更前後の比較に使えます。

## 計測対象を明確に

ベンチ前に **何を計測したいのか** を決めます:

| 計測したいもの | 該当するワークロード |
|---|---|
| 静的ファイル配信 | 単純な GET、`*.html` |
| 動的アプリ実行 | PHP / Servlet / Rack / WSGI 経由 |
| リバースプロキシ | Warp Docker で別サーバへ転送 |
| 大ファイル転送 | 数 MB〜数十 MB ファイル |
| 多数の並列接続 | 1k〜10k 同時接続 |

## 計測ツール

### wrk (推奨、HTTP/1.1)

```bash
wrk -t8 -c128 -d15s --latency http://localhost:2020/
```

| オプション | 意味 |
|---|---|
| `-t8` | スレッド数 |
| `-c128` | 同時接続数 |
| `-d15s` | 計測時間 (= 15 秒) |
| `--latency` | レイテンシ統計を出力 |

### h2load (HTTP/2)

```bash
h2load -t8 -c32 -m32 -D 15 https://localhost:2024/
```

| オプション | 意味 |
|---|---|
| `-c32` | 同時接続数 |
| `-m32` | 1 接続あたり最大ストリーム数 |
| `-D 15` | 計測時間 (秒) |

`-n` (= count-based) は使わない。事前 calibration で偽の値が出ることがあるため。

### h2spec / h3spec (RFC 準拠チェック)

性能ではなく仕様遵守の検査:

```bash
h2spec -h localhost -p 2024 -t
h3spec localhost 2024
```

## 計測手順

1. **stale プロセス掃除** — 前回のサーバが残っていないか確認:
   ```bash
   pkill -f bayserver
   ss -tln | grep -E '2020|2024'
   ```

2. **設定変更** — `.plan` を本番想定の値に

3. **アクセスログ OFF** — Log Docker を外す or `logLevel warn`

4. **CPU pinning** — リアルな計測なら taskset を使い、サーバとクライアントを別 core に
   ```bash
   taskset -c 0-7 bin/bayserver.sh -start
   taskset -c 16-23 wrk ...
   ```

5. **ウォームアップ** — JIT 起動と connection pool 確立のため 3-5 秒の空打ち

6. **本計測** — 15 〜 30 秒
7. **複数 round** — 3-6 回回して中央値 (median) を採用

## 計測項目

各ラン後に以下を記録:

- **RPS** (= Requests/sec)
- **レイテンシ** (= avg, p50, p99, max)
- **エラー件数** (= socket errors / non-2xx) ← **必ず明示**
- **CPU 使用率** (= mpstat / pidstat で取る)
- **メモリ使用量** (= ps / pmap)

### エラー件数の重要性

`wrk` の `Socket errors: connect/read/write/timeout` のいずれかが非ゼロなら、ベンチ結果に必ず注記します。

理由: RPS は **完了したリクエストでしか積まれない** ため、タイムアウトしたリクエストは集計から除外されます。エラー件数を黙ると「RPS が上限に達している」のか「タイムアウトで件数が落ちている」のかを取り違えます。

## サイズ展開

固定の単一ファイルだけでなく、複数サイズで測ると傾向が見える:

| サイズ | 計測の目的 |
|---|---|
| 128 B | 短ボディ、リクエスト処理コストが支配 |
| 1 KB | HTTP オーバヘッドとボディの比が均等 |
| 10 KB | 一般的な HTML ページ程度 |
| 100 KB | 中ボディ、cache 効果が顕著 |
| 1 MB | 大ボディ、I/O 帯域支配 |

## 比較レポート

同じ条件で複数構成 (= BayServer / Nginx / php-fpm 直 / 等) を測ったら、こんな表で:

| size | BayServer | Nginx | PHP-FPM |
|------|---:|---:|---:|
| 128 | 50,000 | 65,000 | 45,000 |
| 1k | 49,000 | 64,000 | 44,000 |
| ... | ... | ... | ... |

加えて: **どのバージョンで、どのオプションで、どの環境で計測したか** を必ず併記する。

## ハマりやすい落とし穴

| 罠 | 対処 |
|---|---|
| 古いサーバプロセスがポートを掴んでいて新サーバが上がらない | `pkill -f` で完全停止 + `ss -tln` で確認 |
| クライアント側のソケット枯渇 | `ulimit -n 65535`, `net.ipv4.tcp_tw_reuse=1` |
| Nagle / delayed-ACK で 40ms 遅延 | TCP_NODELAY / TCP_QUICKACK 確認 |
| LAN 経由で測ってネットワークが律速 | localhost で取るか、専用 NIC |
| 1 サイクル目だけ遅い | ウォームアップ無しで JIT 未起動 |

## マシン情報の記録

公開する計測結果には以下を併記しないと再現性が無い:

- CPU 型番 / コア数 / NUMA 構成 / クロック / Turbo Boost の有無
- メモリ容量 / 種別 (DDR4-3200 等)
- OS / カーネルバージョン
- BayServer のバージョン (= git commit hash)
- 比較対象サーバのバージョン

なお、機密情報 (= ホスト名 / 内部 IP / 個人のホーム path 等) はリポジトリに含めない方針推奨。

---

## 関連

- [Tuning ガイド](tuning.md)
- [パフォーマンス Tips](tips.md)
- [BayServer_Performance_Check](https://github.com/baykit/BayServer_Performance_Check) — 実際のベンチ計測スクリプト集 (= 別リポ)
