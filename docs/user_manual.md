# User Manual for [Project Name]

## Table of Contents
- [Purpose of the Software](#purpose-of-the-software)
- [Operations Environment](#operations-environment)
    - [Hardware Configuration](#hardware-configuration)
    - [Software Configuration](#software-configuration)
- [Operations Manual](#operations-manual)
    - [General](#general)
    - [Set-up and Initialization](#set-up-and-initialization)
    - [Getting Started](#getting-started)
    - [Normal Operations](#normal-operations)
    - [Normal Termination](#normal-termination)
    - [Error Conditions](#error-conditions)
    - [Recover Runs](#recover-runs)
- [Tutorial](#tutorial)

## Purpose of the Software

Argo Workflow is an open-source engine tailored for orchestrating workflows on Kubernetes, enabling users to define, manage, and execute tasks in containers.

## Operations Environment

Argo Workflow is executed on a Kubernetes cluster, on which it orchestrates the containers required fo executing the desired workflows.

### Hardware Configuration

**TBC**
Provide a block diagram illustrating the main hardware components of the system, including required memory, storage, and peripherals like printers.

> Note: section might be omitted or restricted to requirements

### Software Configuration
**TBC**
Outline the main software components through a block diagram, including operating systems, utilities, and other supporting systems.

> Note: might be restricted to description of docker environment

## Operations Manual

### General

Detail the operational organization, schedules, elementary operations at the site, and responsibilities.

> Note: Elementary operations

### Set-up and Initialization

Describe procedures for user identification, authorization, installation, configuration, and initial setup tasks.

The set-up procedures are described in the [Installation Manual](./installation_manual.md).

### Getting Started


1. Ensure Kubectl configuration points to the desired Kubernetes cluster.
2. Open a new terminal.


### Normal Operations

From ARGO CLI, a non-exhaustive list of operations are available. 

#### Workflow Template operations
- List Workflow templates: `argo template list`
- Create Workflow template: `argo -n <k8s-namespace> template create <workflow-template-definition.yaml>`
- Delete workflow template: `argo -n <k8s-namespace> template delete <workflow-template-name>`

#### Workflow operations
- List Workflows: `argo -n <k8s-namespace> list`
- Submit a workflow: `argo -n <k8s-namespace> submit <workflow-definition.yaml>`
- Get status of a workflow: `argo -n <k8s-namespace> get <workflow-name>`
- Retrieve logs of a workflow: `argo -n <k8s-namespace> logs <workflow-name>`
- Delete a workflow: `argo -n <k8s-namespace> delete <workflow-name>`

### Normal Termination

Describe how users can cease or interrupt software use and verify if the termination has been normal.

>NOTE: optional

### Error Conditions

Detail common error conditions, detection methods, and troubleshooting steps.

### Recover Runs

Provide procedures for restarting or recovering from errors and maintaining continuity during emergencies.

> NOTE: if relevant

## Tutorial

> NOTE: Introduce how to use the software and what it accomplishes, tailored for both novices and experts.
A welcoming introduction to the software, setting the stage for its use.
Describe a typical task using graphical illustrations and diagrams to show the actions performed by the user.

## Hello-World Tutorial
This tutorial covers how we can use ARGO CLI to register a workflow template, submit a workflow and monitor its status before retrieving logs and results.

### Register a Workflow template
As we want to register a Workflow template, the first step would be to defined one.
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

Once we have defined a workflow template in a YAML file, we can register this template by calling ARGO CLI:
```
agro -n <k8s-namespace> template create examples/hello-world-template/hello-world-wf-template.yml
```

After the workflow template registration, we should see the template in the list, with the following command:
```
agro -n <k8s-namespace> template list
```

## Artifact Tutorial
This tutorial covers how we can use and produce artifacts in a workflow.
**TBC**