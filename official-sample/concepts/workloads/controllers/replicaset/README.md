# ReplicaSet

## document

 + ReplicaSet
   + https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

## 1. hogehoge

+ Create NameSpace

```
kubectl apply -f 01_namespace.yaml
```

+ Create Pod of ReplicaSet

```
kubectl create -f 11_frontend.yaml
```

+ Check ReplicaSet

```
kubectl get rs --namespace=sample-replicaset
```
```
### Ex

# kubectl get rs --namespace=sample-replicaset
NAME       DESIRED   CURRENT   READY   AGE
frontend   20        20        20      13m
```

+ Delete Pod

```
kubectl delete -f 11_frontend.yaml
```

## 2. hogehoge

```
kubectl create -f 21_non-template-pod.yaml
kubectl create -f 22_frontend.yaml
```
```
# kubectl get rs --namespace=sample-replicaset
NAME       DESIRED   CURRENT   READY   AGE
frontend   10        10        10      65s
```
```
# kubectl get pods --namespace=sample-replicaset
NAME             READY   STATUS    RESTARTS   AGE
frontend-545fn   1/1     Running   0          2m36s
frontend-9gcr7   1/1     Running   0          2m36s
frontend-cnxg4   1/1     Running   0          2m36s
frontend-nbmcx   1/1     Running   0          2m36s
frontend-nlzhr   1/1     Running   0          2m36s
frontend-pbmv9   1/1     Running   0          2m36s
frontend-twxxk   1/1     Running   0          2m36s
frontend-vjxwt   1/1     Running   0          2m36s
pod1             1/1     Running   0          2m41s
pod2             1/1     Running   0          2m41s
```

+ Delete Pod

```
kubectl delete -f 22_frontend.yaml
```

+ Check Pod
  + hogehoge

```
# kubectl get pods --namespace=sample-replicaset
No resources found in sample-replicaset namespace.
```