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
    + 2019/11/18 現在は デフォルトのバージョンは `1.13.11-gke.14` であり、選択出来るバージョンで最も古いのは `1.12.10-gke.17` です。

```
gcloud beta container clusters create no-downtime \
  --network=no-downtime-nw \
  --subnetwork=no-downtime-sb \
  --zone us-central1 \
  --num-nodes=1 \
  --preemptible \
  --cluster-version=1.12.10-gke.17
```

+ kubectl コマンドにて確認を行います。
  + Node のバージョンが指定したバージョンになっていることを確認出来ました。

```
kubectl get nodes
NAME                                         STATUS   ROLES    AGE   VERSION
gke-no-downtime-default-pool-1894e82b-2b2j   Ready    <none>   57s   v1.12.10-gke.17
gke-no-downtime-default-pool-8d4eb0ed-78r1   Ready    <none>   57s   v1.12.10-gke.17
gke-no-downtime-default-pool-d5a8d6e0-vmvd   Ready    <none>   76s   v1.12.10-gke.17
```

## Deployment を作成します

+ `nginx-deployment.yaml` を作成します。

```
vim nginx-deployment.yaml
```
```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.5   # https://hub.docker.com/_/nginx
        ports:
        - containerPort: 80

```

+ deployment を作製します。

```
kubectl create -f nginx-deployment.yaml
```

+ Pod の確認を行います。
  + 出力結果より、どの Node に Pod が乗っているかが分かります。

```
$ kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP          NODE                                         NOMINATED NODE
nginx-deployment-84645fc577-8ctd7   1/1     Running   0          3m25s   10.60.1.3   gke-no-downtime-default-pool-8d4eb0ed-78r1   <none>
nginx-deployment-84645fc577-lkwnm   1/1     Running   0          3m25s   10.60.1.2   gke-no-downtime-default-pool-8d4eb0ed-78r1   <none>
nginx-deployment-84645fc577-rzgkw   1/1     Running   0          3m25s   10.60.2.2   gke-no-downtime-default-pool-1894e82b-2b2j   <none>
```

+ Service を作成します。

```
vim nginx-service-loadbalancer.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-serv-lb
spec:
  selector:
    app: nginx-deployment
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
  type: LoadBalancer
```

+ service を作製します。

```
kubectl create -f nginx-service-loadbalancer.yaml
```

+ service の確認をします。

```
$ kubectl get service
$ kubectl get service
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
kubernetes      ClusterIP      10.63.240.1    <none>          443/TCP          54m
nginx-serv-lb   LoadBalancer   10.63.251.38   34.67.125.234   8080:32387/TCP   67s
```


## 実験

+ クラスターのバージョンの確認します。

```
gcloud container clusters describe no-downtime --zone us-central1
---> これだと出力が多いので、 grep します

gcloud container clusters describe no-downtime --zone us-central1 | grep version
```
```
### 例

$ gcloud container clusters describe no-downtime --zone us-central1 | grep version
  version: 1.12.10-gke.17
```

+ Node のバージョンを確認します。

```
$ kubectl get node
NAME                                         STATUS   ROLES    AGE    VERSION
gke-no-downtime-default-pool-1894e82b-2b2j   Ready    <none>   140m   v1.12.10-gke.17
gke-no-downtime-default-pool-8d4eb0ed-78r1   Ready    <none>   140m   v1.12.10-gke.17
gke-no-downtime-default-pool-d5a8d6e0-vmvd   Ready    <none>   141m   v1.12.10-gke.17
```

+ クラスタ マスターのバージョンをアップグレードするには、まず次のコマンドを実行して利用可能なバージョンを表示します
  + https://cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster#upgrade_master

```
gcloud container get-server-config --region us-central1
```
```
$ gcloud container get-server-config --region us-central1
Fetching server config for us-central1
defaultClusterVersion: 1.13.11-gke.14
defaultImageType: COS
validImageTypes:
- COS
- UBUNTU
- COS_CONTAINERD
- UBUNTU_CONTAINERD
validMasterVersions:
- 1.14.8-gke.12
- 1.14.7-gke.23
- 1.13.12-gke.8
- 1.13.11-gke.14
- 1.12.10-gke.17
validNodeVersions:
- 1.14.8-gke.12
- 1.14.8-gke.2
.
.
.
割愛
```

+ Master の version を上げる

```
### 例
gcloud container clusters upgrade [CLUSTER_NAME] --master --cluster-version [CLUSTER_VERSION]


gcloud container clusters upgrade no-downtime --master --cluster-version 1.13.12-gke.8 --region us-central1
```

+ node と Pod を再度、確認する

```
$ kubectl get node
NAME                                         STATUS   ROLES    AGE     VERSION
gke-no-downtime-default-pool-1894e82b-2b2j   Ready    <none>   71m     v1.12.10-gke.17
gke-no-downtime-default-pool-8d4eb0ed-78r1   Ready    <none>   3h46m   v1.12.10-gke.17
gke-no-downtime-default-pool-d5a8d6e0-vmvd   Ready    <none>   3h46m   v1.12.10-gke.17
```
```
$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-84645fc577-8ctd7   1/1     Running   0          3h33m
nginx-deployment-84645fc577-c6djz   1/1     Running   0          74m
nginx-deployment-84645fc577-lkwnm   1/1     Running   0          3h33m
```

+ クラスタのサイズ変更

```
### 例
gcloud container clusters resize [CLUSTER_NAME] --node-pool [POOL_NAME] \
    --num-nodes [NUM_NODES]


gcloud container clusters resize [CLUSTER_NAME] --node-pool [POOL_NAME] \
    --num-nodes [NUM_NODES]

```

+ Node pool を Master に追従させる
  + default-pool は後ほど修正する

```
### 例
gcloud container clusters upgrade [CLUSTER_NAME] --node-pool=[NODE-POOL-NAME] --cluster-version [CLUSTER_VERSION]

gcloud container clusters upgrade no-downtime --node-pool=default-pool	 --cluster-version 1.13.11-gke.14 --region us-central1
```




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







