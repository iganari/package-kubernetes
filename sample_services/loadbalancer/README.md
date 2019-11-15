# Creating a Service of type LoadBalancer

## 公式ドキュメント

Exposing applications using services / Creating a Service of type LoadBalancer

https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps#creating_a_service_of_type_loadbalancer

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
kubectl delete -f service.yaml
```

+ deployment の削除

```
kubectl delete -f deployment.yaml
```
