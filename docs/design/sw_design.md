# Software Design Document for Argo-Workflow

## Table of Contents

- [General](#general)
- [Component Identifier](#component-identifier)
- [Type](#type)
- [Purpose](#purpose)
- [Function](#function)
- [Subordinates](#subordinates)
- [Dependencies](#dependencies)
- [Interfaces](#interfaces)
- [Resources](#resources)
- [Data](#data)

### General
This section provides a detailed look at the design aspects of each component used in the software. 

### Component Identifier

- **Description**: Argo Workflows enables the chaining of container-based modules and workflows.
- **Naming Convention**: Argo-Workflows

### Type
 
- **Logical Characteristics**: Argo Workflows is the main functional component implementing the Data Processing Environment sub-system of the OHDSA platform.
- **Physical Characteristics**: Argo Workflows is a Commercial-Of-The-Shelf (COTS) Kubernetes-native workflow engine that orchestrates parallel jobs on a Kubernetes cluster. It uses Kubernetes resources like custom resources, pods, jobs, ConfigMaps, and Secrets to implement and manage workflows. Each workflow is defined as a Workflow object, which dictates the sequence of tasks to be executed. These tasks are encapsulated as Kubernetes Jobs or Pods, making each step of the workflow a discrete, schedulable unit under Kubernetes management.

### Purpose

Argo Workflows component is designed to fulfill a pivotal role in managing and executing complex workflows that integrate various modules within Docker containers. The purpose of Argo Workflows is to enable users to submit both simple and elaborate workflows with a versatile Workflow Definition Language that accommodates Directed Acyclic Graph models, dynamic resource allocation, retry strategies, the utilisation of specialized hardware, and completion notifications. 

### Function

Argo Workflows is a container-native workflow engine for Kubernetes, designed to orchestrate parallel jobs in a cloud environment. Here are its major functions:

- **Workflow Orchestration**: Executes workflows where each step is a container, handling sequential, parallel, or conditionally executed steps.
  Each Step and Directed Acyclic Graph (DAG) Task triggers the creation of a Pod. 
  Each Pod comprises three containers:
  - Main Container: This container executes the user-specified image. The argoexec utility is volume-mounted within this container and functions as the primary command, invoking the configured command as a subprocess.
  - Init Container: Known as an InitContainer, this container is responsible for fetching artifacts and parameters, making them accessible to the main container.
  - Wait Container: This container is tasked with performing necessary cleanup operations, including the preservation of parameters and artifacts.
  
  **TBD**: add diagram to represent container usage

  For more information, see: https://argo-workflows.readthedocs.io/en/stable/architecture/

- **DAG Execution**: Argo Workflows can manage task dependencies using Directed Acyclic Graphs to ensure optimal execution order.In a DAG template, tasks without any dependency will be run immediately.

[//]: # (- **Event-driven Execution**: )
[//]: # (  Argo Events is an event-driven workflow automation framework capable of triggering Argo Workflows based on external events &#40;for reactive workflow scenarios&#41;. it supports a significant list &#40;more than 20&#41; of event sources, including AMQP and Minio.)
[//]: # (  Agro Evetns support of Minio enables to automate workflow execution based on changes within a bucket. )
[//]: # (  ![]&#40;D:\env\gitprojects\OHDSA\argo-worfklows\docs\design\sw_design_resources\argo-workflow-trigger.png&#41;)
[//]: # (  Argo Events does not only support the 'Submit' operation but also supports the following operations: Submit,Submit --from,Resubmit, Resume, Retry,Suspend,Terminate,Stop.)
[//]: # (  For more information see: https://argoproj.github.io/argo-events/sensors/triggers/argo-workflow/)

- **Resource Optimization**: Dynamically allocates resources based on task demands, optimizing cluster resource use.
  To achieve resource usage optimization, one can leverage Kubernetes resource mechanisms (limits and requests). 

- **Error Handling and Retry Logic**: Argo Workflow provides built-in mechanisms for error recovery and task retries.


- **Monitoring**: Argo produces metrics that provide information on the controller's status. As those metrics follow the same format as required by prometheus, those can be integrated with Prometheus.
  Two types of metrics are emitted by Argo: 
  - Controller metrics: concerns the state of the controller;
  - Custom metrics: regards the state of a Workflow, or a series of Workflows. Those metrics can be defined on the Workflow/Step/Task emitting the metric. using the same name and help string, is a required by prometheus to track the metrics over time. 
  
  For more information, see: See: https://argo-workflows.readthedocs.io/en/stable/metrics/

- **Logging**: 
It is officially not recommended to rely on Argo to archive logs as it is a naive solution, not designed for indexing, searching, and storing logs (see: https://argo-workflows.readthedocs.io/en/stable/configure-archive-logs/). 
In a Kubernetes environment, logs can be forwarded by an agent running on the node (see: https://kubernetes.io/docs/concepts/cluster-administration/logging/#using-a-node-logging-agent). 
This agent can forward logs to be saved and indexed for a future usage. Such solution can be provided by Fluentd (acting as the agent forwarding logs). Such logs can be forwarded to ElasticSearch (ELK) which supports storing, indexing and searching capabilities.

  **TBD**: add diagram to represent logging workflow

[//]: # (- ![]&#40;D:\env\gitprojects\OHDSA\argo-worfklows\docs\design\sw_design_resources\logging-with-node-agent.png&#41;)
- **Role-Based Access Control (RBAC)**: Utilizes Kubernetes RBAC to control access to workflow execution and management.
  All users of the Argo Server must use a service account in order to interact with the Argo Controller. A single service account can be shared by multiple users, as it is used to list  possible actions a user can do.
  Rules  defined in Argo can associate a user (using their OIDC group) to a service account in the same namespace as Argo server by annotating the desired service account. By using such rules, users from the OIDC provider are associated to the appropriate service account, with which they can interact with Argo Workflow server to manage workflows.
  For more information, see: https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/keycloak/

- **Artifacts**
Argo supports staging-in, staging-out and passing artifacts between steps thanks to artifact repositories.
An artifact repository can be used with any S3 compatible API (like Minio). Artifact repositories are defined in Kubernetes configmaps and referenced in workflow templates in order to reduce information duplication and simplify artifact repository usage.
It also supports other interfaces such as GIT. For more information,see: https://argo-workflows.readthedocs.io/en/stable/configure-artifact-repository/
  
- **Notification**
A default Exit handler can be configured in order to send notifications (such as emails or anything) to notify the completion of a workflow execution (failure/success). The exit handler is defined as a container executing a command, which permits to do anything with the end of the workflow, making it possible to communicate over any protocol.
  https://argo-workflows.readthedocs.io/en/stable/workflow-notifications/

For more detailed information, visit the [official Argo Workflows documentation](https://argoproj.github.io/argo-workflows/).

The following subsections focus on the implementation of the **OHDSA specific capabilities** supported by the workflow engine.

#### API

Argo Workflow provide a REST API endpoint with plenty operations, in which we can find the following ones:
- Workflow Template creation
- List available Workflow Templates 
- Describe Workflow Template
- List Workflows
- Submit Workflow
- Delete Workflow
- Retrieve Workflow Status
- Retrieve Workflow Logs

For more information on those operations,see API design documentation: [API](api_design.md)

#### Reusable Workflow Templates

> TBD: Workflow designer prepares reusable workflow templates for standardizing and streamlining reusable steps

Argo Workflow supports defining workflow templates reused in workflow definitions. The REST API provide an operation to register such workflow template. For more information on this, see section 'Workflow Template creation' in the API Design document: [API](api_design.md)
The following example represents a workflow template definition:
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
See: [Example](../../examples/hello-world-template/hello-world-wf-template.yml)

#### DAG Workflow Creation

Workflow designer creates workflows using a Directed Acyclic Graph (DAG) model.
Workflow templates can be referenced in DAG workflows in order to define workflow's tasks.

The following example represents a DAG workflow definition:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: dag-test-workflow-
spec:
  entrypoint: whalesay
  templates:
    - name: whalesay
      dag:
        tasks:
          - name: call-wf-template-1
            templateRef:
              name: wf-template-1
              template: whalesay-workflow-template
            arguments:
              parameters:
                - name: message
                  value: "hello world"
```


#### Data Artefacts

> For managing input and output data, the following principles are applied:
1. Data Inputs as Artefacts: In a WorkflowTemplate, input data should be provided as an artifact, which is submitted as an argument by the workflow calling this template. See: [Example](../../examples/artifacts/input-artifact/artifact-consumer-wf.yml)
2. Intermediate Artefacts: In a workflow template with multiple steps, intermediate artifacts are stored in the default repository (fast storage). See: [Example](../../examples/artifacts/intermediate-steps-artifacts/demo-wf.yml)
3. Creating Artefacts: Workflows (which can call the templates) can create artifacts from:
A key in the default S3 repository. See: [Example](../../examples/artifacts/input-artifact/artifact-consumer-wf.yml)
A key in a specified internal repository. SeeSee: [Example](../../examples/artifacts/specific-artifact-repository/artifact-consumer-wf.yml)
Remote sources such as S3 or HTTP, offering flexibility. See: [Example](../../examples/artifacts/external-http-artifact/artifact-consumer-wf.yml)
4. Credential Agnosticism: WorkflowTemplates do not handle credentials. External artifacts (HTTP, S3) are managed by a workflow generated by the Processing Gateway.

> TBD interfacing with fast / slow storage areas on S3 and management of artefacts
> Show how to define 2 multiple artifact repository (fast & slow) +configure one as the default one.

Artifact repositories are configured within a Kuebrnetes configmap within the same namespace as the workflow or within the management namespace.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: artifact-repositories
  annotations:
    workflows.argoproj.io/default-artifact-repository: slow-s3-artifact-repository
data:
  slow-s3-artifact-repository: |
    s3:
      bucket: argo-bucket
      endpoint: slow-minio:9000
      insecure: true
      accessKeySecret:
        name: my-minio-cred
        key: accesskey
      secretKeySecret:
        name: my-minio-cred
        key: secretkey
  fast-s3-artifact-repository: |
    s3:
      bucket: argo-bucket
      endpoint: fast-minio:9000
      insecure: true
      accessKeySecret:
        name: my-minio-cred
        key: accesskey
      secretKeySecret:
        name: my-minio-cred
        key: secretkey
```

- **TBD**:Steps I/O Mappings: Workflow designer maps intermediary steps data outputs to inputs steps of the workflow.
- **TBD**:Data Transfer Steps: Workflow designer integrates data stage-in and stage-out steps managing interactions between persistent storage system (for external interfaces), intermediate data storage system (for intermediate steps).


#### Retry Strategy

> TBD Workflow designer implements retry strategies for failed or errored workflow steps
> List the different strategies +show examples (retry number, backoff,...)

The retry strategy is used to decide in which case a step is retried.

**Limit parameter:**
A limit parameter san be specified in order to limit the number of reties.

**Retry Policy:**
In addition to this limit parameter, one can define a retry policy.
Retry policies in Argo Workflows are configured using the retryPolicy parameter defined in Workflow specification. It determines the conditions under which failed steps should be retried.
The available retry policies include:
- Always: Retries all failed steps, regardless of the failure type.
- OnFailure: Retries steps where the main container is marked as failed by Kubernetes.
- OnError: Retries steps that encounter errors related to the Argo controller, or steps where the init or wait containers fail.
- OnTransientError: Retries steps that encounter transient errors or errors that match the pattern specified in the TRANSIENT_ERROR_PATTERN environment variable.

**Expressions**:In addition to retry policies, retries in Argo Workflows can be controlled using expressions. These expressions have access to the following variables:
- lastRetry.exitCode: The exit code of the last retry, or "-1" if not available. 
- lastRetry.status: The phase of the last retry, which can be "Error" or "Failed".
- lastRetry.duration: The duration of the last retry, measured in seconds. 
- lastRetry.message: The message output from the last retry.

If the expression evaluates to false, the step will not be retried. The result of the expression is logically combined (using a logical AND) with the retryPolicy. Both the expression and the retry policy must evaluate to true for a retry to occur.

**Backoff**: The backoff parameter can be used to avoid too frequent retries too soon by introducing a delay between retries, thereby preventing immediate subsequent retry attempts.

**TBD**: add example of retry strategy

For more information, see: https://argo-workflows.readthedocs.io/en/stable/retries/

#### Notification

> **TBD**: describe how exit handler can be used to send notification of any type (using a container to send a AMQP message)

#### Hardware Constraints

> TBD: Resource requirements/limits and affinities
> Describe how to use affinities to deploy pods on nodes with specific harware
> Describe how to define resource requests/limits to deploy on node with sufficient resource (optimize resource usage).

When defining a workflow template, resource requirements can be specified within the 'container' section. The 'podSpecPatch' field in either the workflow or workflow template allows for overriding the resource limits and requests for a container. See: [Example](https://github.com/argoproj/argo-workflows/blob/main/examples/pod-spec-patch.yaml)

  **TBD**: add request/limits example

To ensure a container runs on a Kubernetes node with specific hardware configurations, a nodeSelector can be defined in the workflow specification or workflow template. This nodeSelector should reference labels that match those defined on the desired node. 
It is essential that Kubernetes nodes with specific hardware configurations use labels matching those used in the workflow/template nodeSelector parameter.

**TBD**: add nodeSelector example

#### Artefact Persistence

> TBD:  Workflow designer mark specific artefacts for persistence, overriding their default temporary status


#### Retention Policies

> TBD:  Workflow engine operator configure retention policies to remove expired workflows automatically
> Archiving workflows: https://argo-workflows.readthedocs.io/en/stable/workflow-archive/

The workflow archive stores information about the workflows such as the status, pods executed, results and more. This information is stored in a database such as PostgreSQL.

**Archive TTL**:
Specifies the time period to keep archived workflows before they will be deleted by the archived workflow garbage collection function. The default is forever.

Example:
```yaml
persistence:
  archiveTTL: 10d
```



### Subordinates

Argo Workflows encompasses the various components and resources it manages and interacts with to orchestrate and execute workflows.

- **Workflow Templates** : These are predefined configurations that describe specific workflows. They serve as blueprints from which instances of workflows are generated and executed. Workflow templates define the sequence of tasks, their dependencies, and the resources needed for each task.
- **Tasks/Pods** - In the context of Argo Workflow, each step in a workflow is executed as a separate pod within the Kubernetes environment. These pods can be considered subcomponents of the workflow they belong to, executing specific actions defined by the workflow and then reporting back their status upon completion or failure.

> Note: identify the DB, message broker, prometheus, etc.

### Dependencies
- **Description**: Detail any operations or conditions that must be met before this component can function properly, including exclusions during its operation.

> Note: if relevant, express dependency with other components (e.g. workflow needs module registry)
> Reflect the config of charts 

### Interfaces

- **Control Flow**: Describe how the component starts and terminates, including any interactions during execution (such as interrupts).
- **Data Flow**: Explain the input and output data flows, ensuring data structures are linked with control flows and interface components through common data areas or files.

Documentation detailing the API specifications and usage is provided in teh [API Design](./api_design.md)

> NOte: sequence diagram and link to the api-design document.


### Resources
- **Requirements**: Itemize what the component needs from its environment to perform its function, excluding items that are part of the component interface.

> NOTE: include docker dependencies, might include maven dependencies (libs)


### Data

Argo Workflows utilises complex internal data structures to orchestrate and manage workflows within Kubernetes. Here's a summary of these structures:

- **Workflow Definitions**: Defined as custom resources in YAML, these include:
  - **Element Descriptions**: Identifiers (names), types (e.g., script, container), and dimensions (step hierarchies and dependencies).
  - **Relationships**: Defines parent-child relationships among steps using dependencies that structure the workflow as a Directed Acyclic Graph (DAG).
  - **Value Range**: Includes everything from static strings to dynamic outputs from previous steps.
  - **Initial Values**: Elements such as parameters may have default values specified in the workflow template.

- **Workflow Status**: This structure tracks the real-time state of each step within the workflow, indicating whether a step is pending, running, succeeded, or failed.

- **Artifacts**: Used to manage data inputs and outputs for each step. Artifacts can be files stored in various storage backends (e.g., S3, Artifactory).

For more detailed information about the internal data structures used by Argo Workflows, including their specifications and how they are utilised within workflows, refer to the [official Argo Workflows Field Reference](https://argo-workflows.readthedocs.io/en/stable/fields/). This documentation provides comprehensive guides on defining and managing workflows, with specific sections on workflow specifications, status monitoring, and artifact handling.
