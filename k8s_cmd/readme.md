# Sample Kubernetes Commands

+ 事前にやること
    + クラウドにログイン(マネージドを使っている場合)
    + Kubernetesと認証を通す
+ Nodeに関して
+ Podに関して

## 事前にやること

+ 各クラウドの認証を通す
    + GCP
    + Azure
    + AWS
+ Kubernetesに対して、認証を通す
    + WIP

## Nodeに関して

+ Nodeの確認方法
+ Nodeの増やし方
+ Nodeの減らし方

### Node確認方法

+ :whale: nodeの確認コマンド

```
kubectl get nodes -o wide
```
```
# kubectl get nodes -o wide
NAME                     STATUS    ROLES     AGE       VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
aks-default-12260899-0   Ready     agent     52m       v1.11.2   10.240.0.4    <none>        Ubuntu 16.04.5 LTS   4.15.0-1021-azure   docker://1.13.1
```

### Nodeの増やし方

+ 検討中 :bow:

### Nodeの減らし方

+ 検討中 :bow:

## Podに関して

+ Podの確認方法
+ Podの作成方法
+ Podの削除方法

### Podの確認方法

+ :whale: podの確認コマンド

```
kubectl get pods -o wide
```

### Podの作成方法

+ :whale: コマンド ??

```
WIP
```

### Podの削除方法

+ :whale: Podの削除
    + 2パターンある

```
kubectl delete pod [Pod名]
```

```
kubectl delete -f [作成時のPod定義ファイル]
```
