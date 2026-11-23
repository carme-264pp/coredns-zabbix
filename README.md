# CoreDNS Zabbix

ZabbixでCoreDNSメトリックスを取得するためのテンプレートです。

Zabbix 8.0 のテンプレート画面で [coredns_templates.yaml](coredns_templates.yaml) をインポートします。

| テンプレート | 収集内容 |
| --- | --- |
| `CoreDNS` | DNS リクエスト、応答時間、フォワーダーの応答時間 |
| `CoreDNS Cache` | キャッシュエントリ数とヒット数（成功・拒否の計4項目）、キャッシュのグラフ2枚 |

必要なテンプレートをホストにリンクします。それぞれが `http://{HOST.CONN}:9253/metrics` を取得するため、両方をリンクしたホストでは取得が2系統になります。
