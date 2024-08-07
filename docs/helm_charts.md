# Helm Charts for [Project Name]

## Overview

The official chart for Argo Workflows is used to deploy Argo Workflows. It is described and detailed in [Argo Workflows Helm Documentation](https://github.com/argoproj/argo-helm/tree/main/charts/argo-workflows) with a list of all properties.

The present repository provides the specific parameters used for the DimSum platform. This document only details these parameters and those that need to be provided during installation (depending on the environment).

The exact configuration and prerequisites required for the DimSum platform are detailed in the [installation manual](installation_manual.md).


## Prerequisites

Before you can use this Helm chart, ensure you meet the following prerequisites:
* S3 Bucket must be deployed and a `secret` providing the credentials.
* Target cluster with Kubernetes 1.23+ (with PV support)
* Helm 3.8.0+ installed locally

## Installing the Helm Chart

1. Clone locally the present GitHub repository, which contains the DimSum `charts/values.yaml` with some overridden values.

    ```bash
    git clone https://github.com/argoproj/argo-helm.git /charts
    ```

2. Add the Helm chart repository:

    ```bash
    helm repo add argo https://argoproj.github.io/argo-helm
    helm repo update
    ```

3. Edit the necessary parameters in [charts/values.yml](../charts/values.yml) (see further section)

4. Install the official Argo Workflows Helm chart with:

    ```bash
    helm install [release-name] argo/argo-workflows --namespace [namespace] -f /charts/values.yaml 
    ```
   Replace `[release-name]` with a name for your Helm release, and `[namespace]` with the Kubernetes namespace where you want to deploy.

## Uninstalling the Chart

To remove the deployed Helm chart:
```bash
helm delete [release-name] --namespace [namespace]
```

## Configuration and installation details

This section details the parameters that need to be adapted for the environment on which Argo Workflows is deployed. 

### Artifact Repository Configuration


To configure the artifact repository, you need to set up the `artifactRepositoryRef` in the `values.yaml` file. This configuration defines where Argo Workflows will store its artifacts. 

By default, the workflow uses the artifact repositories defined by the config map `artifact-repositories` (specifying one or more repositories). Within this ConfigMap, the  default repository must be designated using the annotation workflows.argoproj.io/default-artifact-repository. 

The following configuration defines the details of the default repository.

You need to specify the bucket name, endpoint, and the secrets for the access key and secret key. Here's how it works:

- **Bucket**: The name of the bucket where artifacts will be stored.
- **Endpoint**: The endpoint URL for the S3-compatible storage service.
- **Access Key and Secret Key**: These are used to authenticate with the storage service. You need to provide the secrets for these keys. The `accessKeySecret` and `secretKeySecret` fields specify the names and keys of the Kubernetes secrets containing the access key and secret key, respectively.


```yaml
artifactRepositoryRef:
  artifact-repositories:
    annotations:
      workflows.argoproj.io/default-artifact-repository: default-artifact-repository
      default-artifact-repository:
        s3:
            bucket: test
            insecure: true
            endpoint: l-k8s01-master.spb.spacebel.be:30901 # TBD create an additional property
            accessKeySecret:
              name: minio-credentials
              key: accessKey
            secretKeySecret:
              name: minio-credentials
              key: secretKey
```

### Service Account Parameters

>IMPORTANT: These parameters should only be modified by experienced users for advanced configurations. 

Each workflow is associated with a service account that dictates the permissions and actions the workflow can perform within the cluster. 

It is recommended to create a default service account `argo-workflow`, assign all required roles for workflows, and bind the role to that default service account.

```yaml
workflow:
  serviceAccount:
    create: true
    name: "argo-workflow"
  rbac:
    create: true
server:
  rbac:
    create: true
controller:
  workflowDefaults:
    spec:
      serviceAccountName: argo-workflow
```

### Authentication Parameters

Authentication parameters define the authentication modes and settings for Argo Workflows. These parameters are specified in the values.yaml file.


### Ingress Parameters

```yaml
server:
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      # Add other necessary annotations here
    hosts:
      - argo.your-domain.com
    paths:
      - /
    tls: 
      - secretName: argo-tls
        hosts:
          - argo.your-domain.com
```


## Parameters

The present section provides the definition of the properties overridden in the yaml file [../charts/values.yaml](../charts/values.yaml).

### Artefacts Repository Parameters

| Key                   | Type | Default           | Description |
|-----------------------|------|-------------------|-------------|
| artifactRepositoryRef | object | See example above | Store artifact in a S3-compliant object store |


### Service Account Parameters

| Key | Type | Value                                          | Description |
|-----|------|------------------------------------------------|-------------|
| workflow.serviceAccount.create | bool | `true`                                         | Specifies whether a service account should be created |
| workflow.serviceAccount.name | string | `"argo-workflow"`                              | Service account which is used to run workflows |
| workflow.serviceAccount.pullSecrets | list | `[]`  (TBD)                                    | Secrets with credentials to pull images from a private registry. Same format as `.Values.images.pullSecrets` |
| controller.workflowDefaults | object | `{    spec.serviceAccountName: argo-workflow}` | Default values that will apply to all Workflows from this controller, unless overridden on the Workflow-level. Only valid for 2.7+ |


### Authentication Parameters

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| server.authModes | list | `[]` | A list of supported authentication modes. Available values are `server`, `client`, or `sso`. If you provide sso, please configure `.Values.server.sso` as well. |

### Ingress Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
server.serviceType | string | `"ClusterIP"` | Service type for server pods |
server.serviceNodePort | string | `nil` | Service node port |


## Additional Resources

- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

