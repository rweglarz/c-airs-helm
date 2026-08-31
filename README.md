# Prisma AIRS CNI Helm Chart

This repository contains the Helm chart for deploying the Prisma AIRS CNI plugin on a Kubernetes cluster.

## Overview

The `airs-cni` chart installs the Palo Alto Networks CNI binaries and network configuration on each node in your cluster using a DaemonSet.

## Prerequisites

- Helm 3.0+

## Installation

First, add the Helm repository:

```bash
helm repo add r-airs-cni https://rweglarz.github.io/c-airs-helm/
helm repo update
```

To install the chart with the release name `airs`:

```bash
helm install airs r-airs-cni/airs-cni
```

## Configuration

The following table lists the configurable parameters of the `airs-cni` chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `endpoints` | List of IP addresses for endpoints | `[{ip: 10.1.1.11}]` |
| `image.host_repo` | Image repository | `gcr.io/pan-cn-series/airs/pan-cni` |
| `image.tag` | Image tag | `4.0.2` |
| `namespace` | Namespace to deploy the CNI | `kube-system` |
| `deployTo` | Target deployment environment (`aks`, `gke`, `eks`, `openshift`) | `aks` |
| `clusterid` | Unique identifier for the cluster | `1` |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

### GKE Example

```bash
helm install airs r-airs-cni/airs-cni --set deployTo=gke --set "endpoints[0].ip"=172.16.10.4
```

### AWS EKS Example

For AWS EKS with multiple availability zones, you can specify individual endpoints:

```bash
helm install airs r-airs-cni/airs-cni \
	--set deployTo=eks \
	--set "endpoints[0].ip"=172.30.0.247 \
	--set "endpoints[0].zone"=eu-central-1a \
	--set "endpoints[1].ip"=172.30.1.54 \
	--set "endpoints[1].zone"=eu-central-1b
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example:

```bash
helm install airs r-airs-cni/airs-cni -f values.yaml
```

## SubnetInfo Resources

This chart also deploys `SubnetInfo` Custom Resources which are used by the CNI to identify traffic that should bypass the security inspection. By default, the following are created:

- `bypass-metadata`: Bypasses the cloud metadata service IP (`169.254.169.254/32`).
- `bypass-internal-traffic`: Bypasses standard RFC1918 internal IP ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`). If `deployTo` is set to `gke`, it also includes the GKE service IP range.

## Supported Environments

The chart supports different cloud environments via the `deployTo` parameter:
- `aks` (Azure Kubernetes Service)
- `gke` (Google Kubernetes Engine)
- `k3s-flannel` (k3s with default flannel CNI) - experimental
- `eks` (Amazon Elastic Kubernetes Service)
- `openshift` (Red Hat OpenShift)
