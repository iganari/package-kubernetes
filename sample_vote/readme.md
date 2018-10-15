# Sample Vote

## 流れ

+ 作成
    +[ 1. Redsisを用意する]()
    + [2. フロントのアプリを作成する]()
    + [3. ブラウザで確認する]()
+ 削除
    + hogehoge

## 投票システムの作成方法

### 1. Redsisを用意する

+ redisのdeployment作成

```
kubectl create -f 11_back-redis-deployment.yaml
```

+ podsの確認

```
kubectl get pods

or

kubectl get pods | grep vote | grep back
```

+ deploymentの確認

```
kubectl get deployments

or

kubectl get deployments | grep vote | grep back
```

+ redisのserviceを起動

```
kubectl create -f 12_back-redis-service.yaml 
```

+ redisのserviceを確認

```
kubectl get service

or

kubectl get services | grep vote | grep back
```


### 2. フロントのアプリを作成する


+ フロントのdeploymentを作成する


```
kubectl create -f 21_front-app-deployment.yaml
```

+ podsの確認

```
kubectl get pods

or

kubectl get pods | grep vote | grep front
```

+ deploymentの確認

```
kubectl get deployments

or

kubectl get deployments | grep vote | grep front
```


+ redisのserviceを起動
    + :yen: 課金対象です


```
kubectl create -f 22_front-app-service.yaml
```

+ redisのserviceを確認

```
kubectl get service

or

kubectl get services | grep vote | grep front
```

+ serviceの実行状況確認コマンド その1

```
kubectl describe services ${service名}
```
```
### 例

# kubectl describe services azure-vote-front
Name:                     azure-vote-front
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=azure-vote-front
Type:                     LoadBalancer
IP:                       10.0.5.154
LoadBalancer Ingress:     23.100.100.206
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31656/TCP
Endpoints:                10.244.1.42:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  EnsuringLoadBalancer  2m    service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   57s   service-controller  Ensured load balancer
```

+ serviceの実行状況確認コマンド その2


```
kubectl get services ${service名}
```
```
### 例

# kubectl get services azure-vote-front
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.5.154   23.100.100.206   80:31656/TCP   4m
```

### 3. ブラウザで確認する


+ hogehoge


## 投票システムの削除方法

### フロントのserviceの削除

+ :yen: `EXTERNAL-IP` が課金対象のため、使用していないときはこのserviceは削除しておく

```
kubectl delete service ${service名}
```
```
### 例

# kubectl get services
NAME               TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
azure-vote-back    ClusterIP      10.0.132.107   <none>           6379/TCP       9m
azure-vote-front   LoadBalancer   10.0.5.154     23.100.100.206   80:31656/TCP   7m
kubernetes         ClusterIP      10.0.0.1       <none>           443/TCP        26d
# kubectl delete service azure-vote-front
service "azure-vote-front" deleted
# kubectl get services
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
azure-vote-back   ClusterIP   10.0.132.107   <none>        6379/TCP   10m
kubernetes        ClusterIP   10.0.0.1       <none>        443/TCP    26d
```
