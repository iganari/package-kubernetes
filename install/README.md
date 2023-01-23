# インストール方法

## kubectl コマンドのインストール

https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/

+ 最新リリースをダウンロード

```
cd
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
```

+ kubectlバイナリを実行可能にする

```
chmod +x ./kubectl
```

+ バイナリを移動

```
sudo mv ./kubectl /usr/local/bin/kubectl
```

+ バージョンを確認

```
kubectl version --client
```
