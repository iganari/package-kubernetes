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

+ ダウンロード
    + `amd64` の場合

```
mkdir /usr/local/src
cd /usr/local/src
wget -O /usr/local/src/helm-${latest_tag}-linux-amd64.tar.gz "https://storage.googleapis.com/kubernetes-helm/helm-${latest_tag}-linux-amd64.tar.gz"
```

+ 展開

```
tar -zxvf helm-${latest_tag}-linux-amd64.tar.gz
```
```
### 以下のようなディレクトリが作成されます

# ls -1
helm-v2.11.0-linux-amd64.tar.gz
linux-amd64
# tree linux-amd64/
linux-amd64/
├── LICENSE
├── README.md
├── helm
└── tiller



```
