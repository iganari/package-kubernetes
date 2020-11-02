# Sample Guestbook

## 参考資料

このドキュメントは以下の公式ドキュメントを実際にやってみるサンプルです。

+ [GKE] Redis と PHP を使用したゲストブックの作成
    + https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook

## ソースコードの取得

+ ソースコードを取得し、ディレクトリに移動する

```
git clone https://github.com/iganari/package-kubernetes.git
cd package-kubernetes/sample_guestbook
```

## ゲストブックの作成方法

### 1. マスターの Redsis を用意する


+ マニュフェストファイルを指定して、マスターの Redis をデプロイします

```
kubectl create -f 11_redis-master-deployment.yaml
```

+ 作成したデプロイメントを確認する

```
kubectl get deployments
```
```
### 例

# kubectl get deployments
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
redis-master   1         1         1            1           28m
```

+ Podが正常に作成されているか確認する

```
kubectl get pods

or

kubectl get pods | grep redis-master-
```
```
### 例

# kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
redis-master-698964bc87-289rt   1/1       Running   0          7m
```

+ マスターのRedisのログを眺める


```
kubectl logs -f $(kubectl get pods | grep redis-master- | awk '{print $1}')
```

+ マスターのRedisのサービスの起動をする

```
kubectl create -f 12_redis-master-service.yaml 
```

+ サービスが作成されたことを確認する

```
kubectl get service
```
```
### 例

# kubectl get service
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.0.0.1      <none>        443/TCP    14d
redis-master   ClusterIP   10.0.82.135   <none>        6379/TCP   1m
```

### 2. ワーカーのRedisを設定する

+ ワーカーのRedisをデプロイを作成する

```
kubectl create -f 21_redis-slave-deployment.yaml
```

+ Podが正常に作成されているか確認する

```
kubectl get pods

or

kubectl get pods | grep redis-slave-
```


+ ワーカーのRedisのサービスを起動する

```
kubectl create -f 22_redis-slave-service.yaml
```

+ サービスが作成されたことを確認する

```
kubectl get service
```
```
### 例

# kubectl get service
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.0.0.1      <none>        443/TCP    14d
redis-master   ClusterIP   10.0.82.135   <none>        6379/TCP   35m
redis-slave    ClusterIP   10.0.59.241   <none>        6379/TCP   50s
```

### 3. フロントのPHPを設定する

+ フロント(PHP)のデプロイを作成する

```
kubectl create -f 31_frontend-deployment.yaml
```

+ Podが正常に作成されているか確認する

```
kubectl get pods

or

kubectl get pods | grep frontend-
```


+ サービスを起動する
    + :yen: 追加課金が発生します

```
kubectl create -f 32_frontend-service.yaml 
```

### 4. それぞれを確認してみます

+ Node の確認

```
kubectl get node
or
kubectl get node -o wide
```
```
### 例

$ kubectl get nodes
NAME                                         STATUS   ROLES    AGE     VERSION
gke-iganari-k8s-default-pool-4ddcd7bf-ds9t   Ready    <none>   6m1s    v1.13.11-gke.14
gke-iganari-k8s-default-pool-ad8a4b26-b7pq   Ready    <none>   6m5s    v1.13.11-gke.14
gke-iganari-k8s-default-pool-bf49f799-0mns   Ready    <none>   5m57s   v1.13.11-gke.14
```

+ Pod の確認

```
kubectl get pods
or
kubectl get pods -o wide
```
```
### 例

$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
frontend-857ff74f94-b75md      1/1     Running   0          5m47s
frontend-857ff74f94-cxlk8      1/1     Running   0          5m47s
frontend-857ff74f94-nwgzf      1/1     Running   0          5m47s
redis-master-547db6dfd-zs7qh   1/1     Running   0          6m12s
redis-slave-5945fb945d-8mjsp   1/1     Running   0          5m58s
redis-slave-5945fb945d-nwlsr   1/1     Running   0          5m58s
```

+ Deployment の確認

```
kubectl get deployment
or
kubectl get deployment -o wide
```
```
$ kubectl get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
frontend       3/3     3            3           6m57s
redis-master   1/1     1            1           7m22s
redis-slave    2/2     2            2           7m8s
```

+ Service の確認

```
kubectl get service
or
kubectl get service -o wide
```
```
$ kubectl get service
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
frontend       LoadBalancer   10.11.241.179   35.222.143.119   80:31859/TCP   8m6s
kubernetes     ClusterIP      10.11.240.1     <none>           443/TCP        10m
redis-master   ClusterIP      10.11.250.36    <none>           6379/TCP       8m29s
redis-slave    ClusterIP      10.11.242.178   <none>           6379/TCP       8m17s
```


### 5. ゲストブックのウェブサイトにアクセスする

+ 実行状況確認コマンド

```
kubectl describe services frontend
```

+ サービスの確認
    + 実際にアクセス出来るIP等が分かる

```
kubectl get service frontend
```
```
### 例

$ kubectl get service frontend
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
frontend   LoadBalancer   10.11.241.179   35.222.143.119   80:31859/TCP   64s
```

+ ブラウザから確認する


![](./images/gb-01.png)

## ゲストブックの削除方法

### 1. フロントのサービスを削除する

+ Service を削除します。
  + :yen: `EXTERNAL-IP` は課金対象なので、使わない時は削除しておきましょう。

```
kubectl delete -f 32_frontend-service.yaml
```

+ Deployment を削除します。

```
kubectl delete -f 31_frontend-deployment.yaml
```


+ 確認します。

```
### 例

$ kubectl get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   1/1     1            1           20m
redis-slave    2/2     2            2           20m
```
```
### 例

$ kubectl get service
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.11.240.1     <none>        443/TCP    22m
redis-master   ClusterIP   10.11.250.36    <none>        6379/TCP   20m
redis-slave    ClusterIP   10.11.242.178   <none>        6379/TCP   20m
```


### 2. ワーカーの Redis を削除する

+ Service を削除します。

```
kubectl delete -f 22_redis-slave-service.yaml
```

+ Deployment を削除します。

```
kubectl delete -f 21_redis-slave-deployment.yaml
```

+ 確認します。

```
### 例

$ kubectl get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   1/1     1            1           23m
```
```
### 例

$ kubectl get service
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.11.240.1    <none>        443/TCP    24m
redis-master   ClusterIP   10.11.250.36   <none>        6379/TCP   23m
```


### 3. マスターの Redsis を削除する

+ Service を削除します。

```
kubectl delete -f 12_redis-master-service.yaml
```

+ Deployment を削除します。

```
kubectl delete -f 11_redis-master-deployment.yaml
```

+ 確認します。

```
### 例

$ kubectl get deployment
No resources found.
```
```
### 例

$ kubectl get service
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.11.240.1   <none>        443/TCP   26m
```


以上です!!

お疲れさまでした!!:raised_hands:
