# Cloud Storage FUSE CSI driver

## 公式ドキュメント

https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver

## TBD

- マニフェスト作成

```
touch storage-fuse.yaml
```

- 環境変数

```
export _gc_pj_id='Your Google Cloud Project ID'

export _common='gke-fuse'
```

- Cloud Storage Buckets の作成

```
gcloud storage buckets create gs://${_gc_pj_id}-${_common} \
  --default-storage-class standard \
  --location=asia \
  --project ${_gc_pj_id}
```

- GKE で NameSpace を作成

```
cat << __EOF__ >> "storage-fuse.yaml"
apiVersion: v1
kind: Namespace
metadata:
  name: ${_common}
__EOF__
```
```
kubectl apply -f storage-fuse.yaml
```

- GKE で Service Account を作成する

```
cat << __EOF__ >> "storage-fuse.yaml"

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: k8s-${_common}
  namespace: ${_common}
__EOF__
```
```
kubectl apply -f storage-fuse.yaml
```

- Kubernetes の Service Account と Google Cloud の Service Account を紐づける
  - https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver#authentication


```
### Project Number

gcloud projects describe ${_gc_pj_id} --format json | jq -r .projectNumber
```
```
gcloud storage buckets add-iam-policy-binding gs://${_gc_pj_id}-${_common} \
  --member "principal://iam.googleapis.com/projects/`gcloud projects describe ${_gc_pj_id} --format json | jq -r .projectNumber`/locations/global/workloadIdentityPools/${_gc_pj_id}.svc.id.goog/subject/ns/${_common}/sa/k8s-${_common}" \
  --role "roles/storage.admin" \
  --project ${_gc_pj_id}
```

- Cloud Storage にテストファイルを置く (job)
  - https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/cloud-storage-fuse-csi-driver#consume-ephemeral-volume-pod

---> ここまで。検証中

```
cat << __EOF__ >> "storage-fuse.yaml"

---

apiVersion: v1
kind: Pod
metadata:
  name: gcs-fuse-csi-example-ephemeral
  namespace: ${_common}
  annotations:
    gke-gcsfuse/volumes: "true"
spec:
  terminationGracePeriodSeconds: 60
  containers:
  - image: busybox
    name: busybox
    command: ["sleep"]
    args: ["infinity"]
    volumeMounts:
    - name: gcs-fuse-csi-ephemeral
      mountPath: /data
      readOnly: false
  serviceAccountName: k8s-${_common}
  volumes:
  - name: gcs-fuse-csi-ephemeral
    csi:
      driver: gcsfuse.csi.storage.gke.io
      readOnly: false
      volumeAttributes:
        bucketName: ${_gc_pj_id}-${_common}
        mountOptions: "implicit-dirs"
        gcsfuseLoggingSeverity: warning
__EOF__
```
```
kubectl apply -f storage-fuse.yaml
```

- ログイン

```
kubectl get pod --namespace ${_common}

kubectl exec gcs-fuse-csi-example-ephemeral --namespace ${_common} -c busybox -- ls
kubectl exec gcs-fuse-csi-example-ephemeral --namespace ${_common} -c busybox -- "ls /data"
kubectl exec gcs-fuse-csi-example-ephemeral --namespace ${_common} -c busybox -- 'echo "test" > /data/test-file'
```



















- nginx のカスタマイズページを置く (deployment/service)

```
TBD
```
