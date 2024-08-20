# Google Cloud

## Google Kubernetes Engine

- GKE クラスタと認証をする

```
export _gc_pj_id='Your Google Cloud Project ID'

export _gke_cluster_name='GKE Cluster Name'
export _gke_cluster_region='GKE Cluster Region'
```
```
gcloud beta container clusters get-credentials ${_gke_cluster_name} \
  --region ${_gke_cluster_region} \
  --project ${_gc_pj_id}
```

### エラーになる場合

- gcloud を再インストールする
- GCE は特にデフォルトで入っているが、それではなく、普通にインストールする
  - TBD
