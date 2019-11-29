# 単一レプリカのステートフルアプリケーションを実行する

## これは何?

+ 公式ドキュメント
  + https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application
+ 上記の公式ドキュメントを GKE で実際に動かすサンプルです。
  + AKS も後日追記する予定です。
+ 関連ブログ
  + :warning: 作成中です。

## Kubernetes 環境構築

+ 以下をやり方で GKE を構築します。
  + https://github.com/iganari/package-gcp/blob/master/kubernetes/sample-basic/gcloud/README.ja.md

## 構築

### ドキュメントとの相違点

+ 変更点
  + Persistent Volume は作成しません。
  + Persistent Volume Claim の記述をすこし修正します。
+ Why
  + PVC のみで動作します。
  + :warning: 後日、詳細を記載します。

### Persistent Volume Claim

+ デプロイ

```
kubectl create -f gke-mysql-pv-pvc.yaml
```
```
### 例

$ kubectl create -f gke-mysql-pv-pvc.yaml
persistentvolumeclaim/mysql-pv-claim created
```

+ 確認

```
kubectl get pvc
```
```
### 例

$ kubectl get pvc
NAME             STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    mysql-pv-volume   20Gi       RWO            manual         49s
```

### Deployment

+ デプロイ

```
kubectl create -f gke-mysql-deployment.yaml
```
```
$ kubectl create -f gke-mysql-deployment.yaml
deployment.apps/mysql created
```

+ 確認

```
kubectl get deployment
OR
kubectl get deployment {deployment_name}
```
```
$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
mysql   0/1     1            0           72s
```

### Service

+ デプロイ

```
kubectl create -f gke-mysql-service.yaml
```
```
$ kubectl create -f gke-mysql-service.yaml
service/mysql created
```

+ 確認

```
kubectl get services
```
```
$ kubectl get services
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.31.240.1   <none>        443/TCP    66m
mysql        ClusterIP   None          <none>        3306/TCP   12s
```
