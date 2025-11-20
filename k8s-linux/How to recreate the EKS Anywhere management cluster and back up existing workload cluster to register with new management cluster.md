## This document will help with 'How to recreate the EKS Anywhere management cluster and back up existing workload cluster to register with new management cluster'

[EKS A Document Reference](https://anywhere.eks.amazonaws.com/docs/clustermgmt/cluster-backup-restore/restore-cluster/#cluster-not-accessible-or-infrastructure-components-changed-after-etcd-backup-was-taken)

#### Use the same cluster name if the newly created management cluster has the same cluster name as the old one
```
MGMT_CLUSTER_OLD="mgmt-old"
MGMT_CLUSTER_NEW="mgmt-new"
MGMT_CLUSTER_NEW_KUBECONFIG=${MGMT_CLUSTER_NEW}/${MGMT_CLUSTER_NEW}-eks-a-cluster.kubeconfig
```
```
WORKLOAD_CLUSTER_1="w01"
```
#### Substitute the workspace path with the workspace you are using
```
WORKSPACE_PATH="/home/ec2-user/eks-a"
```
#### Retrieve the Cluster API backup folder path that are automatically generated during the cluster upgrade
#### This folder contains all the resources that represent the cluster state of the old management cluster along with its workload clusters
```
CLUSTER_STATE_BACKUP_LATEST=$(ls -Art ${WORKSPACE_PATH}/${MGMT_CLUSTER_OLD} | grep ${MGMT_CLUSTER_OLD}-backup | tail -1)
CLUSTER_STATE_BACKUP_LATEST_PATH=${WORKSPACE_PATH}/${MGMT_CLUSTER_OLD}/${CLUSTER_STATE_BACKUP_LATEST}/
```
#### Substitute the EKS Anywhere release version with the EKS Anywhere version of the original management cluster
```
EKSA_RELEASE_VERSION=v0.17.3
BUNDLE_MANIFEST_URL=$(curl -s https://anywhere-assets.eks.amazonaws.com/releases/eks-a/manifest.yaml | yq ".spec.releases[] | select(.version==\"$EKSA_RELEASE_VERSION\").bundleManifestUrl")
CLI_TOOLS_IMAGE=$(curl -s $BUNDLE_MANIFEST_URL | yq ".spec.versionsBundles[0].eksa.cliTools.uri")
```
##### - The clusterctl move command needs to be executed for each workload cluster.
##### - It will only move the workload cluster resources from the EKS Anywhere backup to the new management cluster.
##### - If you have multiple workload clusters, you have to run the command for each cluster as shown below.

#### Move workload cluster w01 resources to the new management cluster mgmt-new
```
docker run -i --network host -w $(pwd) -v $(pwd):/$(pwd) --entrypoint clusterctl ${CLI_TOOLS_IMAGE} move \
    --namespace eksa-system \
    --filter-cluster ${WORKLOAD_CLUSTER_1} \
    --from-directory ${CLUSTER_STATE_BACKUP_LATEST_PATH} \
    --to-kubeconfig ${MGMT_CLUSTER_NEW_KUBECONFIG}
```
