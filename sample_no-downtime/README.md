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

+ 実験用の VPC ネットワークを作成します
  
```
gcloud beta compute networks create no-downtime \
  --subnet-mode=custom
```
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
