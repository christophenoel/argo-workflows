# Installation Manual

## Overview

This installation manual provides detailed instructions for the installation and configuration of the Argo Workflows component as part of the DimSum platform. 

The DimSum platform's comprehensive setup is documented in the related platform installation guide. This manual focuses specifically on Argo Workflows, facilitating its reuse as an independent component or enabling advanced configuration within the system.


## Prerequisites

Before proceeding with the installation, ensure that the following prerequisites are met and properly configured on your target Kubernetes cluster:

- **Helm:** Helm is required for deploying the Argo Workflows component on Kubernetes. Ensure that Helm is installed and configured on your system.
- **kubectl:** The `kubectl` command-line tool must be installed and configured with access to your target Kubernetes cluster (`kubeconfig`).
- **S3 Storage:** The DimSum platform relies on S3 storage to store the artifacts used by the workflows. Ensure that your S3 storage is configured and accessible.
- **Domain Name or External IP:** The Argo Workflows component requires a domain name or an external IP address that can be bound to the Argo Workflows server.
- **Argo CLI:** The Argo Command Line Interface (CLI) is useful for interacting with Argo Workflows. It can be downloaded from the official Argo Workflows GitHub repository: https://github.com/argoproj/argo-workflows/releases/


## Build Procedure

No build process is required.

## Deployment

This project uses Helm for deployment on Kubernetes. 

Make sure Helm is installed and set up correctly then follow the instructions in [Helm Charts Manual](./helm_charts.md)

You can ensure that the installation was successful, by submitting a workflow to Argo:

```
argo -n argo-helm submit --serviceaccount executor https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml --watch
```

## Configuration

This section provides guidelines for advanced setup related to Argo Workflows.

### Label Kubernetes nodes

> NOTE: TBD detail this section

Labels associated with nodes are used by Argo Workflows and Kubernetes to choose on which node a pods must be scheduled.
Here is an example of how to associate a label with a node:
```
kubectl label nodes <node-name> accelerator=<my-label>
```

To list labels associated with nodes:
```
kubectl get nodes --show-labels=true
```


## Uninstall
```
helm uninstall --namespace argo-helm argo-workflows
kubectl delete namespace argo-helm
```