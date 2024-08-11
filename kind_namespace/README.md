# NameSpace

## 公式ドキュメント

- https://kubernetes.io/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

## namaspace-01

```
kubectl apply -f namespace-01.yaml
```
```
# kubectl get namespace
NAME                             STATUS   AGE
default                          Active   4d23h
namespace-01                     Active   9s
```