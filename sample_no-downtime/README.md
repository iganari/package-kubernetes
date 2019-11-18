# :warning: WIP

# 要件

+ ダウンタイム無く、 Node のアップグレードをしたい

## 環境準備(GKE)

サンプルは Cloud Shell からの実行で動作確認しています。 --> [参考](https://github.com/iganari/package-gcp/tree/master/kubernetes/sample-basic/gcloud)



+ GCP の認証
  + ブラウザを通しての認証を行います。


```
gcloud auth application-default login
```

+ gcloud コマンドの configure 機能を持ちいて設定を管理します
  + また、 GCP 上のプロジェクト名を使用します。
  
```
export _pj='GCP 上のプロジェクト名'
  
  
gcloud config configurations create ${_pj}
gcloud config set project ${_pj}
gcloud config configurations list
```

+ 実験用の VPC ネットワークとそれに付随するサブネットワークを作成します。
  
```
gcloud beta compute networks create no-downtime-nw \
  --subnet-mode=custom
```
```
gcloud beta compute networks subnets create no-downtime-sb \
  --network no-downtime-nw \
  --region us-central1 \
  --range 172.16.0.0/12
```

+ Firewall Rule を作成します。

```
gcloud compute firewall-rules create no-downtime-nw-allow-internal \
  --network no-downtime-nw \
  --allow tcp:0-65535,udp:0-65535,icmp
```

+ GKE を Node(n1-standard-1) が合計 3 個を起動します。
  + Node は preemptible instance を用います。
  + クラスタのバージョンをあえて、古いバージョンに指定して作製します。
    + 2019/11/18 現在は デフォルトのバージョンは `1.13.11-gke.14` です。

```
gcloud beta container clusters create no-downtime \
  --network=no-downtime-nw \
  --subnetwork=no-downtime-sb \
  --zone us-central1 \
  --num-nodes=1 \
  --release-channel stable \
  --preemptible \
  --cluster-version=1.12.10-gke.17
```

## 実験













## リソース削除

+ GKE のリソース削除

```
gcloud beta container clusters delete no-downtime --zone us-central1
```

+ Firewall Rule の削除

```
WIP
```

+ サブネットの削除

```
WIP
```

+ VPC ネットワークの削除

```
WIP
```







