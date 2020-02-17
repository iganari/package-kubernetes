# Sample of Nginx Ingress Controller

## What's this

You will create Nginx Ingress Controller.

## code

WIP


+ create Namespase

```
kubectl create -f 01_namespace.yaml
```

+ Check Namespace

```
# kubectl get namespace
NAME                              STATUS   AGE
default                           Active   29m
kube-node-lease                   Active   29m
kube-public                       Active   29m
kube-system                       Active   29m
sample-nginx-ingress-controller   Active   27s
```

+ Create Pod on my NameSpace

```
kubectl create -f 11_apple-pod.yaml --namespace=sample-nginx-ingress-controller
kubectl create -f 21_banana-pod.yaml --namespace=sample-nginx-ingress-controller
```

+ Check Pod

```
# kubectl get  po --namespace=sample-nginx-ingress-controller
NAME         READY   STATUS    RESTARTS   AGE
apple-app    1/1     Running   0          14s
banana-app   1/1     Running   0          7s
```

+ Create Service on my NameSpace

```
kubectl create -f 12_apple-service.yaml --namespace=sample-nginx-ingress-controller
kubectl create -f 22_banana-service.yaml --namespace=sample-nginx-ingress-controller
```

+ Check Service

```
# kubectl get service --namespace=sample-nginx-ingress-controller
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
apple-service    ClusterIP   10.31.245.4     <none>        5678/TCP   60s
banana-service   ClusterIP   10.31.250.194   <none>        5678/TCP   53s
```

+ Create Ingress on my NameSpace

```
kubectl create -f 31-ingress.yaml --namespace=sample-nginx-ingress-controller
```

+ Check Ingress

```
# kubectl get ingress --namespace=sample-nginx-ingress-controller
NAME             HOSTS   ADDRESS   PORTS   AGE
example-ngress   *                 80      17s
```
