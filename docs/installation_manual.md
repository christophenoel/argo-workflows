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

This project uses Helm for deployment on Kubernetes. Follow the steps below to deploy the application using Helm charts.

### Step 1: Install Helm

Make sure Helm is installed and set up correctly. For installation instructions, see the [official Helm documentation](https://helm.sh/docs/intro/install/).

### Step 2: Deploy Using Helm


Those Helm charts are provided and maintained by the community. It is not an “official” Argo Helm chart, but it is officially recommended in Argo’s documentation.
```
helm repo add argo https://argoproj.github.io/argo-helm
helm install --namespace argo-helm argo-workflows argo/argo-workflows --version 0.41.4 -f charts/values.yaml
```

## Configuration

### Configuring the Application

The configuration applied when deploying the HELM chart is defined in the yaml file [values.yaml](../charts/values.yaml)


<table>
  <caption>
    configuration properties
  </caption>
  <thead>
    <tr>
      <th scope="col">Property</th>
      <th scope="col">Description</th>
      <th scope="col">Default Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">workflow.serviceAccount</th>
      <td>Service account used to run workflows on the cluster.</td>
      <td><code>workflow:
  serviceAccount:
    create: true
    name: "executor"
</code></td>
    </tr>
    <tr>
      <th scope="row">workflow.rbac</th>
      <td>RBAC permissions. This property is used to add Role and RoleBinding for the controller.</td>
      <td><code>workflow:
  rbac:
    create: true</code></td>
    </tr>
    <tr>
      <th scope="row">controller.workflowDefaults</th>
      <td>Specify workflow default values. Here, used to specify the default service account to use (when not provided in the workflow).</td>
      <td><code>controller:
  workflowDefaults:
    spec:
      serviceAccountName: executor</code></td>
    </tr>
    <tr>
      <th scope="row">server.authModes</th>
      <td><p>Authentication mode, used to provide the kubeconfig used in the web server to manage resources.

Possible values:
- server:  use kubeconfig from the service account
- client: client must provide Kubernetes bearer Token
- sso: uses single sign-on (using the same service account as in ‘server’ mode.
<br/>More info:https://argo-workflows.readthedocs.io/en/stable/argo-server-auth-mode/
</p> </td>
      <td><code>server:
  authModes: [ server ]</code></td>
    </tr>
    <tr>
      <th scope="row"> server.serviceType </th>
      <td> Specify the serviceType used to expose the Argo Web server. </td>
      <td><code>serviceType: NodePort</code></td>
    </tr>
    <tr>
      <th scope="row"> server.serviceNodePort </th>
      <td> When using server.ServiceType NodePort, specify the port used to expose the service. </td>
      <td><code>serviceNodePort: 32747</code></td>
    </tr>
    <tr>
      <th scope="row"> artifactRepositoryRef </th>
      <td> <p>Artifact Repository configuration 

Annotations: define the annotation used to identify the default artifact repository.

Configured to use S3 protocole without TLS (insecure: true)

Endpoint  matches the S3 API provided by Minio.

accessKeySecret and secretKeySecret provide credentials used to connect to the S3 server.</p></td>
      <td><code>artifactRepositoryRef:
artifact-repositories:
annotations:
workflows.argoproj.io/default-artifact-repository: default-artifact-repository
default-artifact-repository:
s3:
bucket: test
insecure: true
endpoint: l-k8s01-master.spb.spacebel.be:30901
accessKeySecret:
name: minio-credentials
key: accessKey
secretKeySecret:
name: minio-credentials
key: secretKey</code></td>
    </tr>
  </tbody>
 
</table>

### Verifying the Installation

Ensure the installation was successful, by submitting a workflow to Argo:

```
argo -n argo-helm submit --serviceaccount executor https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml --watch
```

## Troubleshooting

Offer common issues and solutions encountered during the installation process.

### Custom resource Definition (CRD) already defined
Custom resource Definition (CRD) are defined cluster wide and not bound to a namespace. This can cause a conflict when installing multiple instances of Argo Workflows within the same Kubernetes cluster.
If CRD are already installed on the Kubernetes cluster, and you don’t want/need to change those, you can install Argo and skip the CRD installation like this:
```
helm install --namespace argo-helm argo-workflows argo/argo-workflows --version 0.41.4 --set crds.install=false
```

