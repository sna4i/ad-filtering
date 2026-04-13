# ad-filtering

Pi-hole + AdGuard 向けの広告フィルターリスト。  
DNS レベルのドメインブロックとブラウザレベルのコスメティックフィルターを組み合わせて使用する。

## フィルター一覧

| ファイル | 内容 | 対象ツール |
|---|---|---|
| `social-cosmetic.txt` | SNS 上の広告要素を非表示にするフィルター | AdGuard / uBlock Origin |
| `dns-blocklists.txt` | 推奨する第三者ブロックリストの URL 一覧 | Pi-hole / AdGuard Home |
| `ad-tracking-domains.txt` | 広告・トラッキングドメインのブロック | Pi-hole / AdGuard Home |
| `popup-redirect-domains.txt` | ポップアップ・リダイレクト広告ドメインのブロック | Pi-hole / AdGuard Home |
| `regex-rules.txt` | 正規表現によるパターンマッチルール | Pi-hole |

## 使い方

### social-cosmetic.txt (ブラウザ拡張)

AdGuard ブラウザ拡張または uBlock Origin のユーザーフィルターに追加する。

**AdGuard:** 設定 → フィルター → ユーザールール → 内容をコピー&ペースト  
**uBlock Origin:** ダッシュボード → My filters → 内容をコピー&ペースト

> AdGuard の場合、テキストマッチ構文が `:contains()` となる。  
> uBlock Origin の場合は `:has-text()` に読み替えること。

### dns-blocklists.txt (購読リスト)

記載されている URL を Pi-hole または AdGuard Home の adlist (購読リスト) に登録する。

**Pi-hole:**
```bash
# Web UI: Adlists ページから URL を1件ずつ追加
# または CLI:
sudo pihole-FTL sqlite3 /etc/pihole/gravity.db \
  "INSERT OR IGNORE INTO adlist (address, enabled, comment) VALUES ('URL', 1, 'comment');"
sudo pihole -g
```

**AdGuard Home:** フィルタ → DNS ブロックリスト → ブロックリストを追加 → URL を入力

### ad-tracking-domains.txt / popup-redirect-domains.txt (ドメインブロック)

`||domain^` 形式で記載。Pi-hole のカスタム deny リストに登録する。

**Pi-hole:**
```bash
# 1件ずつ登録
sudo pihole deny <domain>

# ワイルドカード (サブドメインも含めてブロック)
sudo pihole --wild <domain>
```

**AdGuard Home:** フィルタ → DNS ブロックリスト → カスタムフィルタリングルールに追加

### regex-rules.txt (正規表現ルール)

パターンマッチでドメインをブロックする。量産される広告ドメインに有効。

**Pi-hole:**
```bash
sudo pihole --regex '<pattern>'
# 例:
sudo pihole --regex '(\.|^)eix[0-9]+\.com$'
```

## 構成の考え方

```
[ネットワーク全体]          [ブラウザ]
Pi-hole / AdGuard Home     AdGuard / uBlock Origin
        │                          │
  DNS ドメインブロック        コスメティックフィルター
  (広告サーバーへの接続遮断)   (ページ内の広告要素を非表示)
        │                          │
        └──────── 併用 ────────────┘
```

- **DNS ブロック**: 広告サーバーへの接続自体を遮断。ネットワーク上の全デバイスに適用される
- **コスメティックフィルター**: DNS ではブロックできない同一ドメイン配信の広告要素をブラウザ側で非表示にする

## 参考・謝辞

### DNS ブロックリスト

| プロジェクト | 用途 |
|---|---|
| [StevenBlack/hosts](https://github.com/StevenBlack/hosts) | 広告・マルウェアの統合 hosts リスト |
| [OISD](https://oisd.nl/) | 広告・トラッキング・マルウェアの包括的ブロックリスト |
| [Hagezi DNS Blocklists](https://github.com/hagezi/dns-blocklists) | Multi Pro / TIF / PopUp Ads 等の高品質リスト群 |
| [AdAway](https://adaway.org/) | モバイル広告ブロックの定番 hosts リスト |
| [URLhaus (abuse.ch)](https://urlhaus.abuse.ch/) | マルウェア配布 URL のデータベース |
| [Phishing Army](https://phishing.army/) | フィッシングドメインのブロックリスト |
| [Yuki2718/adblock](https://github.com/Yuki2718/adblock) | 日本語サイト向け広告フィルター (ゆきフィルタ) |
| [d43m0nhLInt3r/socialblocklists](https://github.com/d43m0nhLInt3r/socialblocklists) | SNS・モバイルアプリ広告のブロックリスト |

### ツール

| ツール | 役割 |
|---|---|
| [Pi-hole](https://pi-hole.net/) | DNS レベルの広告ブロック |
| [AdGuard](https://adguard.com/) | ブラウザレベルのコスメティックフィルタリング |
| [uBlock Origin](https://github.com/gorhill/uBlock) | ブラウザレベルのコンテンツフィルタリング |

## License

MIT
