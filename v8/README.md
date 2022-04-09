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
.\elastic-agent.exe install  `
  --fleet-server-es=https://127.0.0.1:9200 `
  --fleet-server-service-token=<TOKEN> `
  --fleet-server-policy=fleet-server-policy `
  --fleet-server-es-ca-trusted-fingerprint=<CA> `
  --fleet-server-insecure-http
```

- ただ以下のようなエラーになることがあるかも

```
Error: remove C:\Program Files\Elastic\Agent\elastic-agent.exe: Access is denied.
For help, please see our troubleshooting guide at https://www.elastic.co/guide/en/fleet/8.1/fleet-troubleshooting.html
```

- おそらく、`install`コマンドで`Program Files`にリンクしてサービス登録するっぽい
- そのときに失敗すると削除までやってくれているのだが、そこでエラーが出ている

- そのときは`enroll`コマンド＆同じパラメータで再実行

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
