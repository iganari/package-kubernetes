# Sample HELM

## ドキュメント

+ HELM
    + https://docs.helm.sh
+ 公式インストール方法
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
