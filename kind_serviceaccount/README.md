# ServiceAccount

## 公式ドキュメント

- https://kubernetes.io/docs/concepts/security/service-accounts/
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

## sa-01

```
kubectl apply -f sa-01.yaml
```
```
# kubectl get sa
NAME      SECRETS   AGE
default   0         4d23h
sa-01     0         7s
```
