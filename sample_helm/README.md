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
  + 参考: https://helm.sh/docs/intro/install/#from-script

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 0700 get_helm.sh
./get_helm.sh
```

+ 意図した Helm のバージョンがインストール出来ているか確認する
  + 2019/11/29 時点では v3.0.0 が最新バージョン

```
helm version
```
```
# helm version
version.BuildInfo{Version:"v3.0.0", GitCommit:"e29ce2a54e96cd02ccfce88bee4f58bb6e2a28b6", GitTreeState:"clean", GoVersion:"go1.13.4"}
```

## Helmの初期化

:warning: この時点でKubernetesと認証が通っている必要があります

+ Podの確認

```
# kubectl get pod -o wide
No resources found in default namespace.
```

+ コマンド

```
helm ls
```
```
# helm ls
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
```

+ パブリック公開されている WordPress の chart(s) を helm コマンドで確認する 
  + GitHub: https://github.com/helm/charts/tree/master/stable/wordpress

```
helm search hub wordpress
```
```
$ helm search hub wordpress
URL                                                     CHART VERSION   APP VERSION     DESCRIPTION
https://hub.helm.sh/charts/bitnami/wordpress            8.0.1           5.3.0           Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/presslabs/wordpress-...      v0.6.3          v0.6.3          Presslabs WordPress Operator Helm Chart
https://hub.helm.sh/charts/presslabs/wordpress-...      v0.7.3          v0.7.3          A Helm chart for deploying a WordPress site on ...
```

+ helm コマンドで Wordpress をインストールする

```
helm repo add bitnami https://charts.bitnami.com
helm install bitnami/wordpress --version 8.0.1
helm install iganari-wordpress bitnami/wordpress --version 8.0.1
```
```
$ helm install iganari-wordpress bitnami/wordpress --version 8.0.1
NAME: iganari-wordpress
LAST DEPLOYED: Fri Nov 29 13:36:37 2019
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the WordPress URL:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w iganari-wordpress'
  export SERVICE_IP=$(kubectl get svc --namespace default iganari-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo "WordPress URL: http://$SERVICE_IP/"
  echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Login with the following credentials to see your blog

  echo Username: user
  echo Password: $(kubectl get secret --namespace default iganari-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

+ 確認

```
$ kubectl get svc --namespace default -w iganari-wordpress
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
iganari-wordpress   LoadBalancer   10.55.252.103   34.66.64.106   80:30084/TCP,443:30774/TCP   66s
```

+ username と passwordの作成

```
$ echo Username: user
Username: user

$ echo Password: $(kubectl get secret --namespace default iganari-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --d
ecode)
Password: x7iJkLSTEG
```

```
$ helm ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
iganari-wordpress       default         1               2019-11-29 13:36:37.990965003 +0900 +09 deployed        wordpress-8.0.1 5.3.0
```




+ 結果
  + ブラウザで `34.66.64.106` にアクセスして、 Wordpress を確認出来たのと `34.66.64.106/wp-login.php` で管理ページにログインが出来ることを確認
  
  
+ K8s 的な確認

```
$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
iganari-wordpress-847ccd8f9-vlzlz   1/1     Running   0          6m10s
iganari-wordpress-mariadb-0         1/1     Running   0          6m10s

$ kubectl get service
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
iganari-wordpress           LoadBalancer   10.55.252.103   34.66.64.106   80:30084/TCP,443:30774/TCP   6m18s
iganari-wordpress-mariadb   ClusterIP      10.55.240.38    <none>         3306/TCP                     6m18s
kubernetes                  ClusterIP      10.55.240.1     <none>         443/TCP                      52m

$ kubectl get deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
iganari-wordpress   1/1     1            1           6m27s
$
```

## リソース削除

+ リリースの確認

```
$ helm list
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
iganari-wordpress       default         1               2019-11-29 13:36:37.990965003 +0900 +09 deployed        wordpress-8.0.1 5.3.0
```
```
$ helm uninstall iganari-wordpress
release "iganari-wordpress" uninstalled
```
```
$ helm list
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
```


+ repo

```
$ helm repo list
NAME    URL
bitnami https://charts.bitnami.com
```
```
$ helm repo remove bitnami
"bitnami" has been removed from your repositories
```
```
$ helm repo list
Error: no repositories to show
```

+ 確認

```
$ kubectl get pod
No resources found.

$ kubectl get service
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.55.240.1   <none>        443/TCP   63m

$ kubectl get deployment
No resources found.
```
