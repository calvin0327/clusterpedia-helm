# Clusterpedia

This name Clusterpedia is inspired by Wikipedia. It is an encyclopedia of multi-cluster to synchronize, search for, and
simply control multi-cluster resources.

Clusterpedia can synchronize resources with multiple clusters and provide more powerful search features on the basis of
compatibility with Kubernetes OpenAPI to help you effectively get any multi-cluster resource that you are looking for in
a quick and easy way.

## Prerequisites

* [Install Helm version 3 or later](https://helm.sh/docs/intro/install/)

First, add the Clusterpedia chart repo to your local repository.

```bash
$ helm repo add clusterpedia https://clusterpedia-io.github.io/clusterpedia-helm/
$ helm repo list
NAME          	URL
clusterpedia  	https://clusterpedia-io.github.io/clusterpedia-helm/
```

With the repo added, available charts and versions can be viewed.

```bash
helm search repo clusterpedia
```

## Install

### 1. Choose storage components

The Clusterpedia chart provides two storage components such as `bitnami/postgresql` and `bitnami/mysql` to choose from
as sub-charts.

`postgresql` is the default storage component. IF you want to use MySQL, you can
add `--set postgresql.enabled=false --set mysql.enabled=true` in the subsequent installation command.

For specific configuration about storage components,
see [bitnami/postgresql](https://github.com/bitnami/charts/tree/master/bitnami/postgresql)
and [bitnami/mysql](https://github.com/bitnami/charts/tree/master/bitnami/mysql).

**You can also choose not to install any storage component, but use external components. It is already in the charts
directory, so you can directly use [values.yaml](./values.yaml)**

### 2. Choose a installation or management mode for CRDs

Clusterpedia requires proper CRD resources to be created in the retrieval environment. You can choose to manually deploy
CRDs by using YAML, or you can manage it with Helm.

#### Manage manually

```bash
kubectl apply -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/cluster.clusterpedia.io_clustersyncresources.yaml
kubectl apply -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/cluster.clusterpedia.io_pediaclusters.yaml
kubectl apply -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/policy.clusterpedia.io_clusterimportpolicies.yaml
kubectl apply -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/policy.clusterpedia.io_pediaclusterlifecycles.yaml
```

#### Install/Upgrade with Helm

Manually add `--set installCRDs=true` in the subsequent installation command.

### 3. Check if you need to create a local PV

Through the Clusterpedia chart, you can create storage components to use a local PV.

**You need to specify the node where the local PV is located through `--set persistenceMatchNode=<selected node name>`
during installation.**

If you need not create the local PV, you can use `--set persistenceMatchNode=None` to declare it explicitly.

### 4. Install Clusterpedia

After the above procedure is completed, you can run the following command to install Clusterpedia:
> If you want to specify the version, you can install the chart with the `--version` argument.

```bash
helm install clusterpedia clusterpedia/clusterpedia \
--namespace clusterpedia-system \
--create-namespace \
--set persistenceMatchNode= $LOCAL_PV_NODE \
--set installCRDs=true
```

### 5. Create Cluster Auto Import Policy —— ClusterImportPolicy

After 0.4.0, Clusterpedia provides a more friendly way to interface to multi-cloud platforms.

Users can create `ClusterImportPolicy` to automatically discover managed clusters in the multi-cloud platform and
automatically synchronize them as `PediaCluster`,
so you don't need to maintain `PediaCluster` manually based on the managed clusters.

We maintain `PediaCluster` for each multi-cloud platform in
the [Clusterpedia repository](https://github.com/clusterpedia-io/clusterpedia/tree/main/deploy/clusterimportpolicy).
ClusterImportPolicy` for each multi-cloud platform.

**People also submit ClusterImportPolicy to Clusterpedia for interfacing to other multi-cloud platforms.**

After installing Clusterpedia, you can create the appropriate `ClusterImportPolicy`,
or [create a new `ClusterImportPolicy`](https://clusterpedia.io/docs/usage/interfacing-to-multi-cloud-platforms/#new-clusterimportpolicy)
according to your needs (multi-cloud platform).

For details, please refer
to [Interfacing to Multi-Cloud Platforms](https://clusterpedia.io/docs/usage/interfacing-to-multi-cloud-platforms)

```bash
kubectl get clusterimportpolicy
```

## Uninstall

If you have deployed `ClusterImportPolicy` then you need to clean up the `ClusterImportPolicy` resources first.

```bash
kubectl get clusterimportpolicy
```

Before uninstallation, you shall manually clear all `PediaCluster` resources.

```bash
kubectl get pediacluster
```

You can run the command to uninstall it after the `PediaCluster` resources are cleared.

```bash
helm -n clusterpedia-system uninstall clusterpedia
```

You need to delete the crd manually whether or not manually created.

```bash
kubectl delete -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/cluster.clusterpedia.io_clustersyncresources.yaml
kubectl delete -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/cluster.clusterpedia.io_pediaclusters.yaml
kubectl delete -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/policy.clusterpedia.io_clusterimportpolicies.yaml
kubectl delete -f https://raw.githubusercontent.com/clusterpedia-io/clusterpedia-helm/main/charts/clusterpedia/_crds/policy.clusterpedia.io_pediaclusterlifecycles.yaml
```

**Note that PVC and PV will not be deleted. You need to manually delete them.**

If you created a local PV, you need log in to the node and remove all remained data about the local PV.

```bash
# Log in to the node with Local PV
rm /var/local/clusterpedia/internalstorage/<storage type>
```

## Values

### General parameters

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| commonAnnotations | object | `{}` |  |
| commonLabels | object | `{}` |  |
| global.imagePullSecrets | list | `[]` |  |
| global.imageRegistry | string | `""` |  |
| hookJob.image.pullPolicy | string | `"IfNotPresent"` |  |
| hookJob.image.registry | string | `"ghcr.io"` |  |
| hookJob.image.repository | string | `"cloudtty/cloudshell"` |  |
| hookJob.image.tag | string | `"v0.4.0"` |  |
| installCRDs | bool | `false` |  |
| storage.image.pullPolicy | string | `"IfNotPresent"` |  |
| storage.image.pullSecrets | list | `[]` |  |
| storage.image.registry | string | `""` |  |
| storage.image.repository | string | `""` |  |
| storage.image.tag | string | `""` |  |
| storage.name | string | `""` |  |
| storage.pluginDir | string | `""` |  |

### StorageConfig

| Key | Type | Default | Description |
|-----|------|---------|-------------|

### ExternalStorage

| Key | Type | Default | Description |
|-----|------|---------|-------------|

### Apiserver

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| apiserver.affinity | object | `{}` |  |
| apiserver.enableSHA1Cert | bool | `false` |  |
| apiserver.featureGates.AllowRawSQLQuery | bool | `false` |  |
| apiserver.featureGates.RemainingItemCount | bool | `false` |  |
| apiserver.image.pullPolicy | string | `"IfNotPresent"` |  |
| apiserver.image.pullSecrets | list | `[]` |  |
| apiserver.image.registry | string | `"ghcr.io"` |  |
| apiserver.image.repository | string | `"clusterpedia-io/clusterpedia/apiserver"` |  |
| apiserver.image.tag | string | `"v0.5.1"` |  |
| apiserver.labels | object | `{}` |  |
| apiserver.nodeSelector | object | `{}` |  |
| apiserver.podAnnotations | object | `{}` |  |
| apiserver.podLabels | object | `{}` |  |
| apiserver.replicaCount | int | `1` |  |
| apiserver.resources | object | `{}` |  |
| apiserver.tolerations | list | `[]` |  |

### ClustersynchroManager

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| clustersynchroManager.affinity | object | `{}` |  |
| clustersynchroManager.featureGates.AllowSyncAllCustomResources | bool | `false` |  |
| clustersynchroManager.featureGates.AllowSyncAllResources | bool | `false` |  |
| clustersynchroManager.featureGates.PruneLastAppliedConfiguration | bool | `true` |  |
| clustersynchroManager.featureGates.PruneManagedFields | bool | `true` |  |
| clustersynchroManager.image.pullPolicy | string | `"IfNotPresent"` |  |
| clustersynchroManager.image.pullSecrets | list | `[]` |  |
| clustersynchroManager.image.registry | string | `"ghcr.io"` |  |
| clustersynchroManager.image.repository | string | `"clusterpedia-io/clusterpedia/clustersynchro-manager"` |  |
| clustersynchroManager.image.tag | string | `"v0.5.1"` |  |
| clustersynchroManager.labels | object | `{}` |  |
| clustersynchroManager.nodeSelector | object | `{}` |  |
| clustersynchroManager.podAnnotations | object | `{}` |  |
| clustersynchroManager.podLabels | object | `{}` |  |
| clustersynchroManager.replicaCount | int | `1` |  |
| clustersynchroManager.resources | object | `{}` |  |
| clustersynchroManager.tolerations | list | `[]` |  |

### ControllerManager

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| controllerManager.affinity | object | `{}` |  |
| controllerManager.featureGates | object | `{}` |  |
| controllerManager.image.pullPolicy | string | `"IfNotPresent"` |  |
| controllerManager.image.pullSecrets | list | `[]` |  |
| controllerManager.image.registry | string | `"ghcr.io"` |  |
| controllerManager.image.repository | string | `"clusterpedia-io/clusterpedia/controller-manager"` |  |
| controllerManager.image.tag | string | `"v0.5.1"` |  |
| controllerManager.labels | object | `{}` |  |
| controllerManager.nodeSelector | object | `{}` |  |
| controllerManager.podAnnotations | object | `{}` |  |
| controllerManager.podLabels | object | `{}` |  |
| controllerManager.replicaCount | int | `1` |  |
| controllerManager.resources | object | `{}` |  |
| controllerManager.tolerations | list | `[]` |  |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs](https://github.com/norwoodj/helm-docs)
