## Table of Content

####    Create a Managed Windows AD

####    Create a Windows Container Cluster

-   [Reference](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster-windows?hl=zh_tw)

-   Enable Kubernetes API

```shell
export CLUSTER_NAME=windows-gke
export WIN_NODE_POOL_NAME=windows-node-pool
export WIN_TYPE=WINDOWS_LTSC
```
-   Run below command to create a GKE cluster

```shell
gcloud beta container clusters create $CLUSTER_NAME \
  --enable-ip-alias \
  --num-nodes=2 \
  --release-channel=rapid
```

-   Run below command to create a Windows Node Poo;

```shell
gcloud container node-pools create $WIN_NODE_POOL_NAME \
  --cluster=$CLUSTER_NAME \
  --image-type=$WIN_TYPE \
  --enable-autoupgrade \
  --machine-type=n1-standard-2
```

-   Get GKE credential

```shell
gcloud container clusters get-credentials $CLUSTER_NAME
```

