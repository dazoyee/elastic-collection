elastic-collection
---

# get started Version.8.1.0

## 1. サービス起動

- `.\elasticsearch\bin\elasticsearch.bat`
- `.\kibana\bin\kibana.bat`

## 2. kibanaにアクセス

1. `http://localhost:5601`
2. Enrollment token

- `bin\elasticsearch-create-enrollment-token.bat --scope kibana`

3. Verification Code

- `bin\kibana-verification-code.bat`

4. Reload

## 3. パスワード生成

- 自動生成
    - `.\elasticsearch\bin\elasticsearch-setup-passwords.bat auto`

or

- 手動生成
    - `.\elasticsearch\bin\elasticsearch-setup-passwords.bat interactive`

```
elastic
組み込みのスーパーユーザー。

`elastic` Userとしてログインできる人は誰でも、.securityなどの制限されたインデックスに直接読み取り専用でアクセスできます。
このユーザーには、セキュリティを管理し、無制限の特権を持つロールを作成する機能もあります。

kibana_system
KibanaがElasticsearchとの接続と通信に使用するユーザー。

logstash_system
LogstashがElasticsearchにモニタリング情報を保存するときに使用するユーザー。

beats_system
BeatsがElasticsearchにモニタリング情報を保存するときに使用するユーザー。

apm_system
Elasticsearchにモニタリング情報を保存するときにAPMサーバーが使用するユーザー。

remote_monitoring_user
ユーザーMetricbeatは、Elasticsearchでモニタリング情報を収集および保存するときに使用します。
これには、`remote_monitoring_agent`および`remote_monitoring_collector`の組み込みの役割があります。
```

## 4. Fleetサーバのインストール

1. What type of host are you adding?

- プルダウンから選択

2. Download the Fleet Server to a centralized host

- 事前にダウンロードしておく

3. Choose a deployment mode for security

- `Quick start`でよい

4. Add your Fleet Server host

- `http://localhost:8220`

5. Generate a service token

- トークン発行

6. Start Fleet Server

- ダウンロードした`elastic-agent`から実行

```
.\elastic-agent.exe enroll  `
  --fleet-server-es=https://127.0.0.1:9200 `
  --fleet-server-service-token=<TOKEN> `
  --fleet-server-policy=fleet-server-policy `
  --fleet-server-es-ca-trusted-fingerprint=<CA> `
  --fleet-server-insecure-http
```

- `enroll`コマンドで実行

```
Successfully enrolled the Elastic Agent.
```

- 最終的にはこのログがでればよい
- そして起動

```
.\elastic-agent.exe run
```

## 5. Agent policies

- 基本的には画面の手順通り
- 追加したAgentに対してポリシー追加する必要がある

### 5.1. Custom Logs

Custom configurations

```
json:
  keys_under_root: true
  add_error_key: true
  overwrite_keys: true

```

### 5.2. Elastic APM

```
java -javaagent:/path/to/elastic-apm-agent-<version>.jar \
-Delastic.apm.service_name=my-application \
-Delastic.apm.server_urls=http://localhost:8200 \
-Delastic.apm.secret_token= \
-Delastic.apm.environment=production \
-Delastic.apm.application_packages=org.example \
-jar my-application.jar
```

# Upgrade Elasticsearch

1. シャードの割り当てを無効にします。

```
curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
'
```

2. 重要でないインデックス作成を停止し、フラッシュを実行します。(オプション)

```
curl -X POST "localhost:9200/_flush?pretty"
```

3. アクティブな機械学習ジョブとデータフィードに関連付けられているタスクを一時的に停止します。(オプション)

```
curl -X POST "localhost:9200/_ml/set_upgrade_mode?enabled=true&pretty"
```

4. 1 つのノードをシャットダウンします。
5. シャットダウンしたノードをアップグレードします。

6. プラグインをアップグレードします。

```
elasticsearch-plugin.bat
```

7. アップグレードされたノードを起動します。

8. シャードの割り当てを再度有効にします。

```
curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
'
```

9. ノードが回復するまで待ちます。

```
curl -X GET "localhost:9200/_cat/health?v=true&pretty"
```

```
curl -X GET "localhost:9200/_cat/recovery?pretty"
```
