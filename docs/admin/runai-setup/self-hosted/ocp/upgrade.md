---
title: Upgrade self-hosted OpenShift installation
---
# Upgrade Run:ai 

## Preparations

=== "Connected"
    No preparation required.

=== "Airgapped" 
    * Ask for a tar file `runai-air-gapped-<new-version>.tar` from Run:ai customer support. The file contains the new version you want to upgrade to. `new-version` is the updated version of the Run:ai control plane.
    * Upload the images as described [here](preparations.md#runai-software-files).

## Upgrade from Version 2.7 or 2.8.

Before upgrading the control plane, run: 

``` bash
kubectl delete --namespace runai-backend --all \
    deployments,statefulset,svc,routes,ServiceAccount
kubectl delete svc -n kube-system runai-cluster-kube-prometh-kubelet
for secret in `kubectl get secret -n runai-backend | grep -v helm.sh/release.v1 | grep -v NAME | awk '{print $1}'`; do kubectl delete secrets -n runai-backend $secret; done
kubectl patch pvc -n runai-backend pvc-postgresql  -p '{"metadata": {"annotations":{"helm.sh/resource-policy": "keep"}}}'
```

Then upgrade the control plane as described [below](#upgrade-the-control-plane). 


## Upgrade from Version 2.9 or 2.10.

Two significant changes to the control-plane installation have happened with version 2.12: _PVC ownership_ and _installation customization_. 
#### PVC Ownership

Run:ai has transferred control of storage to the customer. Specifically, the Kubernetes Persistent Volumes are now owned by the customer and will not be deleted when the Run:ai control-plane is uninstalled. 

To remove the ownership in an older installation, run:

```
kubectl patch pvc -n runai-backend pvc-postgresql  -p '{"metadata": {"annotations":{"helm.sh/resource-policy": "keep"}}}'
```

#### Installation Customization

The Run:ai control-plane installation has been rewritten and is no longer using a _backend values file_. Instead, to customize the installation use standard `--set` flags. If you have previously customized the installation, you must now extract these customizations and add them as `--set` flag to the helm installation:

* Find older customizations (Run:ai provides a utility for that).
* Find the customization in the [optional configurations](./backend.md#optional-additional-configurations) table and add it in the new format. 



Then upgrade the control plane as described [below](#upgrade-the-control-plane). 

## Upgrade the Control Plane

=== "Connected"
    ``` bash
    helm upgrade -i runai-backend -n runai-backend runai-backend/control-plane  \
    --set global.domain=runai.apps.<OPENSHIFT-CLUSTER-DOMAIN> \ #(1)
    --set global.config.kubernetesDistribution=openshift \
    --set backend.config.openshiftIdpFirstAdmin=<FIRST_ADMIN_USER_OF_RUNAI> \ # (2)
    --set thanos.query.stores={thanos-grpc-port-forwarder:10901} \
    --set postgresql.primary.persistence.existingClaim=pvc-postgresql
    ```

    1. The subdomain configured for the OpenShift cluster.
    2. Name of the administrator user in the company directory.

=== "Airgapped"
    ``` bash

    helm upgrade -i runai-backend  ./runai-backend-<version>.tgz -n runai-backend \
    --set global.domain=runai.apps.<OPENSHIFT-CLUSTER-DOMAIN> \ #(1)
    --set global.config.kubernetesDistribution=openshift \
    --set backend.config.openshiftIdpFirstAdmin=<FIRST_ADMIN_USER_OF_RUNAI> \ # (2)
    --set thanos.query.stores={thanos-grpc-port-forwarder:10901} \
    --set postgresql.primary.persistence.existingClaim=pvc-postgresql
    ```

    1. The subdomain configured for the OpenShift cluster.
    2. Name of the administrator user in the company directory.

## Upgrade Cluster 

=== "Connected"
    To upgrade the cluster follow the instructions [here](../../cluster-setup/cluster-upgrade.md).

=== "Airgapped"
    ```
    helm get values runai-cluster -n runai > values.yaml
    helm upgrade runai-cluster -n runai runai-cluster-<version>.tgz -f values.yaml
    ```
    (replace `<version>` with the cluster version)