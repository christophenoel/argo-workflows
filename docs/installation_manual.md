# Installation Manual

## Overview

This manual provides comprehensive guidance on how to install and set up Argo Workflow. Ensure all prerequisites are met before proceeding with the installation.

## Prerequisites

Before you begin the installation, ensure that the following software is installed and configured on your system:

- Docker
- Kubernetes
- Helm
- Argo CLI

NB: Argo CLI can be downloaded from here: https://github.com/argoproj/argo-workflows/releases/

## Build Procedure

NO Build process required.

## Deployment

This project uses Helm for deployment on Kubernetes. Make sure Helm is installed and set up correctly then follow the instructions in [Helm Charts Manual](./helm_charts.md)

### Verifying the Installation

Ensure the installation was successful, by submitting a workflow to Argo:

```
argo -n argo-helm submit --serviceaccount executor https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml --watch
```

### Addition configuration

#### Label Kubernetes nodes

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