# Squid Proxy (Docker 開発用)

シンプルに Squid を Docker で立ち上げて手元の HTTP/HTTPS 通信をプロキシ経由でテストするための最小構成です。

## 構成
- `docker-compose.yml` : Ubuntu ベース公式イメージ `ubuntu/squid`
- `squid.conf` : 開発用 (キャッシュ無効 / 広めのプライベートレンジ許可)
- `logs/` : `access.log` / `cache.log` をホストへマウント

## 起動 / 停止
```bash
docker compose up -d
docker compose logs -f squid   # 起動ログ監視
docker compose restart squid   # 設定変更後の再読み込み
docker compose down            # 停止
```

## 動作確認 (curl)
最もシンプルな疎通確認は `-x <proxy>` オプションでプロキシを指定するだけです。

```bash
# Google へ HEAD リクエスト (HTTP)
curl -x http://localhost:3128 -I http://www.google.com

# httpbin (グローバルIP取得) を HTTP 経由
curl -x http://localhost:3128 http://httpbin.org/ip

# HTTPS サイト (CONNECT メソッド動作確認)
curl -x http://localhost:3128 -I https://www.google.com
```

レスポンスヘッダが返り、`access.log` に `TCP_TUNNEL` や `TCP_MISS/200` 等が記録されれば成功です。失敗時は `TCP_DENIED/403` が出るので ACL 設定 (`squid.conf`) を確認してください。

## ログ確認
```bash
tail -f logs/access.log
tail -f logs/cache.log
```

代表的なログ例:
- `TCP_DENIED/403` : ACL によって拒否。接続元 IP が `acl localnet` に含まれているか確認。
- `TCP_TUNNEL/200` : HTTPS CONNECT トンネル成功。
