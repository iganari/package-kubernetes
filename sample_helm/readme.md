# Sample HELM

WIP

## 公式ドキュメント

+ HELM
    + https://docs.helm.sh
+ インストール方法
    + https://docs.helm.sh/using_helm/#installing-helm
+ GitHub
    + https://github.com/helm/helm


## HELMをインストールする

+ 最新のリリース情報
    + https://github.com/helm/helm/releases

+ 最新のタグを取得する

```
export latest_tag=$(curl 'https://github.com/helm/helm/releases.atom' | grep '<id>tag:github.com,2008:Repository/43723161/' | head -n1 | awk -F\/ '{print $3}' | sed 's/<//g')
echo ${latest_tag}
```

+ スクリプトを使用して、最新のHELM(とtiller)をインストールする

```
cd ~
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
chmod 0700 get_helm.sh
./get_helm.sh
```

+ 確認する

```
helm version
```
```
# helm version
Client: &version.Version{SemVer:"v2.11.0", GitCommit:"2e55dbe1fdb5fdb96b75ff144a339489417b146b", GitTreeState:"clean"}
Error: could not find tiller
```

## Helmの初期化

:warning: この時点でKubernetesと認証が通っている必要があります

+ Podの確認

```
# kubectl get pod -o wide
No resources found.
```

+ 初期化コマンドを実行

```
helm init
```

+ `tiller` 用のPodが立ち上がっていることを確認する(?)

```
kubectl get pods --namespace kube-system | grep tiller-
```

## WordPressを立ち上げてみる

+ helmコマンドを用いて、WordPressをインストールする

```
helm inspect stable/wordpress
```

+ 以下のような、メッセージが出てきます

```
NOTES:
1. Get the WordPress URL:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.                                                            
        Watch the status with: 'kubectl get svc --namespace default -w reeling-unicorn-wordpress'                                     

  export SERVICE_IP=$(kubectl get svc --namespace default reeling-unicorn-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo "WordPress URL: http://$SERVICE_IP/"
  echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Login with the following credentials to see your blog

  echo Username: user
  echo Password: $(kubectl get secret --namespace default reeling-unicorn-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

+ 1. Get the WordPress URLをやってみる

```
# kubectl get svc --namespace default -w reeling-unicorn-wordpress
NAME                        TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)                      AGE
reeling-unicorn-wordpress   LoadBalancer   10.0.75.221   40.115.154.255   80:31891/TCP,443:31350/TCP   8m
```

---> `EXTERNAL-IP` 欄の `40.115.154.255` をブラウザで見てみる(自動で取得したグローバルIPアドレスのため、固定ではありません)




+ Podを確認する

```
# kubectl get pod
NAME                                         READY     STATUS              RESTARTS   AGE
reeling-unicorn-mariadb-0                    0/1       ContainerCreating   0          1m
reeling-unicorn-wordpress-6b69994dd5-f4lqb   0/1       ContainerCreating   0          1m
```
