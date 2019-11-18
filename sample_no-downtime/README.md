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


---> 出来ているが疎通が


## 実験

WIP











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







