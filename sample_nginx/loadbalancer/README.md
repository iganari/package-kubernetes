# loadbalancer

## 作成

+ deployment の作成

```
kubectl create -f deployment.yaml
```

+ Service の作成

```
kubectl create -f service.yaml
```

## 削除

+ Service の削除

```
kubectl delete -f service-loadbalancer.yaml
```

+ deployment の削除

```
kubectl delete -f deployment.yaml
```
