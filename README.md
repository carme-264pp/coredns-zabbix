# CoreDNS Zabbix

ZabbixでCoreDNSメトリックスを取得するためのテンプレートです。

Zabbix 8.0 のテンプレート画面で [coredns_templates.yaml](coredns_templates.yaml) をインポートします。

| テンプレート | 収集内容 |
| --- | --- |
| `CoreDNS` | DNS リクエスト、応答時間、フォワーダーの応答時間 |
| `CoreDNS Cache` | キャッシュエントリ数とヒット数（成功・拒否の計4項目）、キャッシュのグラフ2枚 |

必要なテンプレートをホストにリンクします。それぞれが `http://{HOST.CONN}:9253/metrics` を取得するため、両方をリンクしたホストでは取得が2系統になります。

## DNSリクエスト件数グラフ

`CoreDNS` テンプレートは、検出したサーバー・ゾーン・ビューの組み合わせごとに、その IP バージョンの TCP と UDP のグラフを作成します。

- `DNS requests IPv4/TCP for {#SERVER} {#ZONE} {#VIEW}`
- `DNS requests IPv4/UDP for {#SERVER} {#ZONE} {#VIEW}`
- `DNS requests IPv6/TCP for {#SERVER} {#ZONE} {#VIEW}`
- `DNS requests IPv6/UDP for {#SERVER} {#ZONE} {#VIEW}`

各グラフは、`A`、`AAAA`、`HTTPS`、`SVCB`、`other` など、DNS クエリタイプ別の件数を積み上げて表示します。各系列の値は累積カウンター `coredns_dns_requests_total` の前回取得値との差分です。前回の取得から今回の取得までに受け付けたリクエストの件数と内訳を確認できます。

## 応答時間グラフ

`CoreDNS` テンプレートは、DNSリクエストをサーバー・ゾーン・ビュー別に、フォワーダーを転送先・応答コード（`NOERROR`、`NXDOMAIN`、`SERVFAIL`）別に表示します。テンプレートの上書きインポート後、ディスカバリが既存の遅延グラフを平均応答時間と p95 の2系列に更新します。従来の件数は既存アイテムの履歴から確認できます。

DNSリクエストのグラフ名は `DNS Request durations {#SERVER} {#ZONE} {#VIEW}` です。フォワーダーのグラフ名は `DNS forward durations {#SERVER} NOERROR`、`DNS forward durations {#SERVER} NXDOMAIN`、`DNS forward durations {#SERVER} SERVFAIL` です。

各点は、連続して取得できた2つのヒストグラムの差分から計算します。平均応答時間は合計処理時間の増分をリクエスト件数の増分で割った値、p95 は件数が95%に達するバケット内で線形補間した推定値です。単位はどちらもミリ秒です。

初回、観測0件、ヒストグラムの欠損・不整合、カウンターリセットやCoreDNS再起動を検出した区間は欠測になります。再起動の判定には、取得できる場合に `process_start_time_seconds` を使います。p95 が最終バケット（8.192秒超）に入る場合、その区間は平均応答時間のみを表示します。長期間の表示で複数の点が重なる場合は、平均応答時間は各点の平均、p95 は各点の最大値を表示します。

## キャッシュグラフ

`CoreDNS Cache` テンプレートは、正常応答のキャッシュ（`success`）と否定応答のキャッシュ（`denial`）をそれぞれのグラフに2系列で表示します。

| グラフ名 | 表示内容 |
| --- | --- |
| `CoreDNS: Cache entries` | 取得時点でキャッシュに保持されているエントリ数（`coredns_cache_entries`）の推移 |
| `CoreDNS: Cache hits` | 前回の取得から今回の取得までのヒット件数の推移。累積カウンター `coredns_cache_hits_total` の差分を表示 |
