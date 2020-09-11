# Kubernetes Commands

+ あるのは大きく分けて 2 つ
  + Kubernetes 特有のコマンド
  + クラウド特有のもの

+ 公式リファレンス
  + https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands


## Nodeに関して

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

WIP

### Nodeの減らし方

WIP









## Podに関して

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

### Podの再起動方法

+ [kubectl rollout restart](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-restart-em-)

說明 | コマンド
:- | :-
Deployment の再起動 | `kubectl rollout restart deployment/nginx`
daemonset の再起動 | `kubectl rollout restart daemonset/abc`

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

+ シングルコンテナな Pod の場合

說明 | コマンド
:- | :-
現時点での pod のログを確認する | `kubectl logs ${pod_name}`
リアルタイムで pod のログを標準出力に出す | `kubectl logs -f ${pod_name}`

+ 複数コンテナが入っている Pod の場合

說明 | コマンド
:- | :-
現時点での pod のログを確認する | `kubectl logs ${pod_name} ${contaner_name}`
リアルタイムで pod のログを標準出力に出す | `kubectl logs -f ${pod_name} ${contaner_name}`

## Namespaces

+ 公式ドキュメント
  + https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

+ コマンドにて指定する場合

コマンド | 説明
:- | :-
kubectl get namespace | 現在の環境の Namespace を確認する
kubectl create -f hogehoge.yaml --namespace='insert-namespace-name-here' | 対象のオブジェクトを特定の Namespce に作成する 

+ YAML に書く場合
  + metadata に記載する

```
metadata:
  name: app
  namespace: my-workspace  <--- これ
```
