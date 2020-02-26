# Kubernetes Commands

+ あるのは大きく分けて 2 つ
  + Kubernetes 特有のコマンド
  + クラウド特有のもの

## Nodeに関して

+ Nodeの確認方法

```
WIP
```

+ Nodeの増やし方

```
WIP
```

+ Nodeの減らし方

```
WIP
```

### Node確認方法

+ :whale: nodeの確認コマンド

```
kubectl get nodes -o wide
```
```
### 例
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

```
WIP
```

+ Podの作成方法

```
WIP
```

+ Podの削除方法

```
WIP
```

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

## log の確認方法


WIP

## Namespaces

+ 公式ドキュメント
  + https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

コマンド | 説明
:- | :-
kubectl get namespace | 現在の環境の Namespace を確認する
kubectl create -f hogehoge.yaml --namespace=<insert-namespace-name-here> | 対象のオブジェクトを特定の Namespce に作成する 
