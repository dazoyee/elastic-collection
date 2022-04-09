elasticsearch
---

# 初期設定

## 1. config配下ファイルの書き換え

## 2. サービス名の書き換え

`bin/elasticsearch-service.bat`
12行目

```
if "%SERVICE_ID%" == "" set SERVICE_ID=elasticsearch-node-1
```

## 3. サービス登録

`bin/elasticsearch-service.bat install`

```
The service 'elasticsearch-node-1' has been installed.
```

※ 注意

- 起動順は気を付けたほうがいい（masterから）
- クラスタ設定をせずに起動してしまうとnodeのuuidが決まってしまうので、そのときには`data/`を削除すること

```
curl -X PUT "localhost:9200/_all/_settings" -H 'Content-Type: application/json' -d'{ "index.blocks.read_only" : false } }'
curl -XDELETE 'http://localhost:9200/.kibana_task_manager_1'
```

# アップグレード

## 1. アップグレードバージョンをダウンロードして解凍する

## 2. 解凍したディレクトリの`config`書き換えておく

## 3. シャード・アロケーションの無効化

### 以下リクエストをKibanaのdevtoolを使って実行する

```
PUT _cluster/settings
{
    "persistent": {
    "cluster.routing.allocation.enable": "primaries"
    }
}
```

これで、バージョンアップ対象のNodeがシャットダウンされた際に、不要なシャードのアロケーション処理が停止される。 この作業の目的は、不要なI/Oを抑えることにある。

## 4. 必要のないインデックス作成を停止し、同期フラッシュを実行(オプション)

### 以下リクエストをKibanaのdevtoolを使って実行する。

```
POST _flush/synced
```

（ローリング）アップグレード中にもインデックス作成は続けることが可能。 しかし、不要なインデックス作成を一時停止し、同期フラッシュを実行したほうがシャードの復旧は早くなる。

これは、シャード復旧を早くするための処置となる。

※ 注意点として、このリクエストは、失敗した場合でも200応答が得られる。 何度かリクエストを飛ばすことで、faild項目が0になるため、faildが0になるため何度か実施する必要がある。

## 5. アクティブな機械学習ジョブやデータフィードに関連するタスクを一時的に停止(オプション)

### 以下リクエストをKibanaのdevtoolを使って実行する

```
POST _ml/set_upgrade_mode?enabled=true
```

機械学習ジョブ起動している場合には必要

## 6. 対象ノードのelasticsearchを停止しアップグレードを実施

### 1. すべてのnodeをサービス停止する

### 2. 解凍したディレクトリの`data`を現バージョンのものに差し替える

## 7. アップグレードされたノードを起動する

```
http://localhost:9200/_cat/nodes?pretty
```

## 8. シャードの割り当てを再開

### kibanaのdevtoolで以下を実行

```
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}

```

アロケートの抑止を停止し、シャードのアロケートが再開される。

これで停止している間のシャード操作が追いつく

## 9. ノードが回復するのを待つ

kibanaのdevtoolかcurlにて以下エンドポイントへリクエストを行うと、ノード回復の進捗が確認できる。

```
http://localhost:9200/_cat/health?v=true
```

ステータス欄が緑に切り替わるのを待つ。 ノードが緑色になると、すべてのプライマリおよびレプリカのシャードが割り当てられたことになる。
