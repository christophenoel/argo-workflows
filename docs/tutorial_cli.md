### Argo CLI

This tutorial demonstrates how to use the Argo CLI to register a workflow template, submit a workflow, monitor its status, and retrieve logs and results.

#### Installing Argo CLI

The Argo CLI is a command-line interface tool that allows users to interact with Argo Workflows. Follow the steps below to install the Argo CLI:

1. **Download the latest Argo CLI release**:

   Visit the [Argo Workflows GitHub releases page](https://github.com/argoproj/argo-workflows/releases) and download the appropriate binary for your operating system.

   `Example: curl -sLO https://github.com/argoproj/argo-workflows/releases/latest/download/argo-linux-amd64.gz`

2. **Unzip the downloaded binary**:

   Unzip the downloaded binary.

   `Example: gunzip argo-linux-amd64.gz`

3. **Make the binary executable**:

   Make the unzipped binary executable.

   `Example: chmod +x argo-linux-amd64`

4. **Move the binary to your PATH**:

   Move the executable binary to a directory included in your system's `PATH`.

   `Example: mv argo-linux-amd64 /usr/local/bin/argo`

5. **Verify the installation**:

   Confirm that the Argo CLI has been installed correctly by running the version command.

   `Example: argo version`



#### Workflow Template operations

The main operations required to manage Workflow Templates are the following:

- List Workflow templates: `argo template list`
- Create Workflow template: `argo -n <k8s-namespace> template create <workflow-template-definition.yaml>`
- Delete workflow template: `argo -n <k8s-namespace> template delete <workflow-template-name>`

#### Workflow operations

The main operations required to manage Workflows are the following:

- List Workflows: `argo -n <k8s-namespace> list`
- Submit a workflow: `argo -n <k8s-namespace> submit <workflow-definition.yaml>`
- Get status of a workflow: `argo -n <k8s-namespace> get <workflow-name>`
- Retrieve logs of a workflow: `argo -n <k8s-namespace> logs <workflow-name>`
- Delete a workflow: `argo -n <k8s-namespace> delete <workflow-name>`

#### Register a Workflow template

To register a workflow template, we first need to define one.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
name: hello-world-wft
spec:
entrypoint: whalesay
templates:
- name: whalesay
inputs:
parameters:
- name: message
container:
image: docker/whalesay
command: [ cowsay ]
args: ["{{inputs.parameters.message}}"]
```

See: [hello-world-wf-template.yml](../examples/hello-world-template/hello-world-wf-template.yml)

After defining a workflow template in a YAML file, we can register it using the Argo CLI. To proceed, open a new terminal window at the root of this project and execute the following command:

```
argo -n <k8s-namespace> template create examples/hello-world-template/hello-world-wf-template.yml
```

After registering the workflow template, verify its registration with the following command:
```
argo -n <k8s-namespace> template list
```

#### Submit a workflow

Define the workflow in a YAML file as shown below.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-workflow-
spec:
  arguments:
    parameters:
      - name: message
        value: This is a whale message # Message to pass to the template
  # Reference the WorkflowTemplate
  workflowTemplateRef:
    name: hello-world-wft
```
See: [hello-world-wf.yml](../examples/hello-world-template/hello-world-wf.yml)

Once we have defined a workflow in a YAML file, we can submit it by calling ARGO CLI. To proceed, open a new terminal windows at the root of this project. Then execute the following command:

```
argo -n <k8s-namespace> submit examples/hello-world-template/hello-world-wf.yml
```

#### Monitor a workflow

In order to monitor a submitted workflow, we can retrieve its status with the following command:
```
argo -n <k8s-namespace> get <workflow-name>
```
Note that `<workflow-name>` correspond to the name of the workflow (available in the response to the submit workflow operation). In this example, it corresponds to: hello-world-workflow-qltqd

In the response, we can observe that `Status` is `Succeeded`. meaning that the execution of this workflow terminated successfully.

To retrieve the logs associated with this workflow, one can use the following command:
```
argo -n <k8s-namespace> get <workflow-name>
```
