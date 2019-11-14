# Hello World

## 参考

公式ドキュメント

https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/

## hogehoge

+ 実行

```
kubectl apply -f load-balancer-example.yaml
```

+ 確認方法

```
kubectl get deployments hello-world
```
```
$ kubectl get deployments hello-world
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   0/5     5            0           11s
```

+ WIP

```
kubectl describe deployments hello-world
```
```
$ kubectl describe deployments hello-world
Name:                   hello-world
Namespace:              default
CreationTimestamp:      Thu, 14 Nov 2019 20:12:57 +0900
Labels:                 app.kubernetes.io/name=load-balacer-example
Annotations:            deployment.kubernetes.io/revision: 1
.
.
.
割愛
```

+ WIP

```
kubectl get replicasets
```
```
$ kubectl get replicasets
NAME                     DESIRED   CURRENT   READY   AGE
hello-world-65f5f799db   5         5         0       112s
```

+ WIP

```
kubectl describe replicasets
```
```
$ kubectl describe replicasets
Name:           hello-world-65f5f799db
Namespace:      default
Selector:       app.kubernetes.io/name=load-balancer-example,pod-template-hash=65f5f799db
Labels:         app.kubernetes.io/name=load-balancer-example
                pod-template-hash=65f5f799db
.
.
.
割愛
```

+ 

## service の作成

+ WIP

```
kubectl apply -f service.yaml
```

+ WIP

```
kubectl get services
```
```
$ kubectl get services
NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes              ClusterIP      10.11.240.1     <none>        443/TCP        15m
load-balancer-example   LoadBalancer   10.11.248.145   <pending>     80:32372/TCP   14s
```