# ReplicaSet

## document

 + ReplicaSet
   + https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

```
kubectl apply -f 01_namespace.yaml
```
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