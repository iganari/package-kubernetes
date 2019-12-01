# Sample HELM

WIP

## 公式ドキュメント

+ HELM
    + https://docs.helm.sh
+ インストール方法
    + https://docs.helm.sh/using_helm/#installing-helm
+ GitHub
    + https://github.com/helm/helm


## Helm をインストールする

+ 最新のリリース情報
    + https://github.com/helm/helm/releases

+ スクリプトを使用して、最新の Helm をインストールします。
  + 参考: https://helm.sh/docs/intro/install/#from-script

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 0700 get_helm.sh
./get_helm.sh
```

+ 意図した Helm のバージョンがインストール出来ているか確認します。
  + 2019/11/29 時点では v3.0.0 が最新バージョンです。

```
helm version
```
```
### 例

# helm version
version.BuildInfo{Version:"v3.0.0", GitCommit:"e29ce2a54e96cd02ccfce88bee4f58bb6e2a28b6", GitTreeState:"clean", GoVersion:"go1.13.4"}
```

## Helm を使って、 WordPress を起動してみる

:warning: この時点でKubernetesと認証が通っている必要があります

+ Pod の確認
  + まだ Pod は何も起動していない想定です。

```
kubectl get pod -o wide
```
```
### 例

# kubectl get pod -o wide
No resources found in default namespace.
```

+ helm コマンドにて確認
  + こちらもまだ何も登録されていない想定です。

```
helm ls
```
```
### 例

# helm ls
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
```

+ WordPress の Charts を確認
  + パブリック公開されている WordPress の chart(s) を helm コマンドで確認します。
  + GitHub: https://github.com/helm/charts/tree/master/stable/wordpress

```
helm search hub wordpress
```
```
### 例

# helm search hub wordpress
URL                                                     CHART VERSION   APP VERSION     DESCRIPTION
https://hub.helm.sh/charts/bitnami/wordpress            8.0.1           5.3.0           Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/presslabs/wordpress-...      v0.6.3          v0.6.3          Presslabs WordPress Operator Helm Chart
https://hub.helm.sh/charts/presslabs/wordpress-...      v0.7.3          v0.7.3          A Helm chart for deploying a WordPress site on ...
```

+ Chat のレポジトリの追加
  + helm コマンドで 使用したい Wordpress の Repository をインストールします。

```
helm repo add bitnami https://charts.bitnami.com
```
```
### 確認

$ helm repo list
NAME    URL                       
bitnami https://charts.bitnami.com
```

## WordPress を構築してみる

### ほぼ、デフォルトの状態で構築する

+ WordPress のインストール
  + helm コマンドを用いて、 Chart から WordPress をインストールします。 

```
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

+ 作成された WordPress の外部 IP アドレスの確認

```
$ kubectl get svc --namespace default -w iganari-wordpress
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE
iganari-wordpress   LoadBalancer   10.55.252.103   34.66.64.106   80:30084/TCP,443:30774/TCP   66s
```

+ password の作成
  + 出力結果の通りのコマンドを実行します。

```
$ echo Password: $(kubectl get secret --namespace default iganari-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
Password: 0FywWk90us


### Alpine Linux 上でコマンドを実行の際は --decode ではなく、 -d を使って下さい

$ echo Password: $(kubectl get secret --namespace default iganari-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
```

+ helm コマンドを使って確認

```
$ helm ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
iganari-wordpress       default         1               2019-12-01 13:33:36.542685106 +0900 JST deployed        wordpress-8.0.1 5.3.0
```

+ 外部 IP アドレスの確認

```
kubectl get services iganari-wordpress
NAME                TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
iganari-wordpress   LoadBalancer   10.55.247.62   35.223.131.61   80:30884/TCP,443:32320/TCP   27m
```

+ ブラウザでの確認


IMG



+ 結果
  + ブラウザで `http://35.223.131.61` にアクセスして、 Wordpress を確認出来たのと `http://35.223.131.61/wp-login.php` で管理ページにログインが出来ることを確認出来ました

### ユーザ名やパスワードをアレンジしてみる

WIP

### HTTPS してみる

WIP

## リソース削除

試験が終わったらリソースを削除しておきましょう

+ リリースを削除
  + リリースの確認 -> 削除 -> 確認と進みます

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


+ 自分で追加した Repository を削除
  + Repository の確認 -> 削除 -> 確認と進みます

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

以上です 

お疲れさまでした!! :raised_hands:
