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


gcloud container clusters resize no-downtime --node-pool default-pool \
  --num-nodes 2 \
  --region us-central1

```

+ 気づき
  + node の数を増やすだけだと、バージョンは上がらない
  
+ node-pool 単位で増やしてみる
  + add-pool という Node pool を増やす

```
### 例
gcloud container node-pools create [POOL_NAME] --cluster [CLUSTER_NAME]

gcloud container node-pools create add-pool --cluster no-downtime --num-nodes 1 --region us-central1
```

+ 確認
  + 新しい node-poolはmaster と同じversionになっている(なぜかはちゃんとdocumentを確認する)

```
$ kubectl get node -o wide
NAME                                         STATUS                     ROLES    AGE    VERSION           INTERNAL-IP   EXTERNAL-IP       OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-no-downtime-add-pool-14f515db-002l       Ready                      <none>   109s   v1.13.12-gke.8    172.16.0.40   34.69.231.137     Container-Optimized OS from Google   4.14.138+        docker://18.9.7
gke-no-downtime-add-pool-1a7629c6-7hw2       Ready                      <none>   110s   v1.13.12-gke.8    172.16.0.41   104.155.160.114   Container-Optimized OS from Google   4.14.138+        docker://18.9.7
gke-no-downtime-add-pool-a7f76d2b-dq99       Ready                      <none>   108s   v1.13.12-gke.8    172.16.0.39   23.236.63.190     Container-Optimized OS from Google   4.14.138+        docker://18.9.7
gke-no-downtime-default-pool-1894e82b-rmxd   Ready,SchedulingDisabled   <none>   84m    v1.12.10-gke.17   172.16.0.26   35.232.88.172     Container-Optimized OS from Google   4.14.138+        docker://17.3.2
gke-no-downtime-default-pool-8d4eb0ed-qpbw   Ready                      <none>   84m    v1.12.10-gke.17   172.16.0.27   34.66.72.175      Container-Optimized OS from Google   4.14.138+        docker://17.3.2
gke-no-downtime-default-pool-d5a8d6e0-564s   Ready                      <none>   84m    v1.12.10-gke.17   172.16.0.29   35.238.191.152    Container-Optimized OS from Google   4.14.138+        docker://17.3.2
igarashi_toru@cloudshell:~ (ca-igarashi-test)$ kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP          NODE                                         NOMINATED NODE   READINESS GATES
nginx-deployment-84645fc577-4rqhd   1/1     Running   0          15m   10.60.7.5   gke-no-downtime-default-pool-8d4eb0ed-qpbw   <none>           <none>
nginx-deployment-84645fc577-7pjfs   1/1     Running   0          15m   10.60.9.5   gke-no-downtime-default-pool-d5a8d6e0-564s   <none>           <none>
nginx-deployment-84645fc577-zqsrm   1/1     Running   0          15m   10.60.7.6   gke-no-downtime-default-pool-8d4eb0ed-qpbw   <none>           <none>
```


+ Pod がどの Node にあるか確認
  + gke-no-downtime-default-pool-8d4eb0ed-qpbw
    + default-pool
  + gke-no-downtime-default-pool-d5a8d6e0-564s
    + default-pool
  + gke-no-downtime-default-pool-8d4eb0ed-qpbw
    + default-pool

```
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP          NODE                                         NOMINATED NODE   READINESS GATES
nginx-deployment-84645fc577-4rqhd   1/1     Running   0          18m   10.60.7.5   gke-no-downtime-default-pool-8d4eb0ed-qpbw   <none>           <none>
nginx-deployment-84645fc577-7pjfs   1/1     Running   0          18m   10.60.9.5   gke-no-downtime-default-pool-d5a8d6e0-564s   <none>           <none>
nginx-deployment-84645fc577-zqsrm   1/1     Running   0          18m   10.60.7.6   gke-no-downtime-default-pool-8d4eb0ed-qpbw   <none>           <none>
```

+ PodDisruptionBudget を登録する
  + https://qiita.com/tkusumi/items/946b0f31931d21a78058

```
vim podDisruptionbudget.yaml
```
```
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: myapp
spec:
  # 最大で無効状態な Pod は 1 に設定。これを超えては退去させられない
  maxUnavailable: 1
  # 対象 Pod のセレクタ
  selector:
    matchLabels:
      run: nginx
```
```
kubectl create -f podDisruptionbudget.yaml
```

```
$ kubectl get pdb
NAME    MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
myapp   N/A             1                 0                     73m
```

+ default-pool のnodeをdrainする

```
kubectl drain --ignore-daemonsets --force gke-no-downtime-default-pool-1894e82b-rmxd
kubectl drain --ignore-daemonsets --force gke-no-downtime-default-pool-8d4eb0ed-qpbw
kubectl drain --ignore-daemonsets --force gke-no-downtime-default-pool-d5a8d6e0-564s
```

```
$ kubectl get pdb
NAME    MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
myapp   N/A             1                 0                     78m
```

+ Pod が乗っている Node を確認する
  + あたらしいノードに写ってる

```
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE                                     NOMINATED NODE   READINESS GATES
nginx-deployment-84645fc577-64jbj   1/1     Running   0          2m11s   10.60.14.3   gke-no-downtime-add-pool-a7f76d2b-dq99   <none>           <none>
nginx-deployment-84645fc577-kmmkq   1/1     Running   0          2m11s   10.60.12.2   gke-no-downtime-add-pool-1a7629c6-7hw2   <none>           <none>
nginx-deployment-84645fc577-qndlj   1/1     Running   0          83s     10.60.13.3   gke-no-downtime-add-pool-14f515db-002l   <none>           <none>
```


+ nodeの削除

```
kubectl drain --ignore-daemonsets --force gke-no-downtime-default-pool-1894e82b-2b2j gke-no-downtime-default-pool-1894e82b-c41w
gcloud container clusters resize no-downtime --node-pool default-pool   --num-nodes 2   --region us-central1
```

## 1.14.8-gke.12 もやってみる

+ Master の version を上げる

```
### 例
gcloud container clusters upgrade [CLUSTER_NAME] --master --cluster-version [CLUSTER_VERSION]

gcloud container clusters upgrade no-downtime --master --cluster-version 1.14.8-gke.12 --region us-central1
```

+ pod を確認する

```

```

+ 新しい node-pool を作成する

```
### 例
gcloud container node-pools create [POOL_NAME] --cluster [CLUSTER_NAME]

gcloud container node-pools create add-pool-v2 --cluster no-downtime --num-nodes 1 --region us-central1
```

+ pod を確認する

```

```

+ drain を仕掛ける

```

```

+ pod を確認する

```

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







