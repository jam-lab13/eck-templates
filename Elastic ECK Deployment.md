
## Troubleshooting Guide
# https://olamideolajide.medium.com/eck-quickstart-guide-troubleshooting-3f60e7deb22d
# when applying files: use --v=5 for increased output verbosity

## Elastic ECK Deployment

#Install Custom Definitions
kubectl create -f https://download.elastic.co/downloads/eck/2.6.1/crds.yaml

#Install ECK operator with RBAC rules
kubectl apply -f https://download.elastic.co/downloads/eck/2.6.1/operator.yaml

#You should now have an operator deployed in the elastic-system namespace

cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: es_cluster_1
spec:
  version: 8.6.2
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
EOF

# Setting node.store.allow_mmap: false has performance implications and should be tuned for production workloads as described in the Virtual memory section.

### PVC OUPUT:
#apiVersion: v1
#kind: PersistentVolumeClaim
#metadata:
#  creationTimestamp: "2023-02-26T22:17:32Z"
#  finalizers:
#  - kubernetes.io/pvc-protection
#  labels:
#    common.k8s.elastic.co/type: elasticsearch
#    elasticsearch.k8s.elastic.co/cluster-name: test
#    elasticsearch.k8s.elastic.co/statefulset-name: test-es-default
#  name: elasticsearch-data-test-es-default-0
#  namespace: default
#  ownerReferences:
#  - apiVersion: elasticsearch.k8s.elastic.co/v1
#    kind: Elasticsearch
#    name: test
#    uid: 43a4528e-abd8-46e5-9c25-27cf5c4c1d93
#  resourceVersion: "135408"
#  uid: f1ac6cde-d1bb-4aa4-a603-934f5c292c16
#spec:
#  accessModes:
#  - ReadWriteOnce
#  resources:
#    requests:
#      storage: 1Gi
#  storageClassName: local-storage
#  volumeMode: Filesystem

### Create a PV for the pod
cat <<EOF | tee /etc/dev_range/kube_configs/eck_configs/es-cluster-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
name: elasticsearch-data-quickstart-es-pv
spec:
capacity:
storage: 1Gi
volumeMode: Filesystem
accessModes:
— ReadWriteOnce
hostPath:
path: /tmp
EOF
# Storage class is only used for dynamic PV provisioning

##Another Example:
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elasticsearch-data-quickstart-es-pv
  labels:
   type: local
spec:
  capacity:
    storage: 10Gi
  storageClassName: local-storage
  volumeMode: Filesystem
  accessModes:
  — ReadWriteOnce
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/server2
          operator: In
          values:
          - test-volume
  local:
    path: "/tmp"
