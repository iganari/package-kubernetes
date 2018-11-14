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

```
helm inspect stable/wordpress
```

+ 確認

```
# kubectl get pod
NAME                                         READY     STATUS              RESTARTS   AGE
reeling-unicorn-mariadb-0                    0/1       ContainerCreating   0          1m
reeling-unicorn-wordpress-6b69994dd5-f4lqb   0/1       ContainerCreating   0          1m
```
