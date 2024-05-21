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
  Each Step and Directed Acyclic Graph (DAG) Task triggers the creation of a Pod. Each Pod comprises three containers:
  - Main Container: This container executes the user-specified image. The argoexec utility is volume-mounted within this container and functions as the primary command, invoking the configured command as a subprocess.
  - Init Container: Known as an InitContainer, this container is responsible for fetching artifacts and parameters, making them accessible to the main container.
  - Wait Container: This container is tasked with performing necessary cleanup operations, including the preservation of parameters and artifacts.
  ![](D:\env\gitprojects\OHDSA\argo-worfklows\docs\design\sw_design_resources\argo-workflow-overview.jpeg)
    For more information, see: https://argo-workflows.readthedocs.io/en/stable/architecture/

- **DAG Execution**: Argo Workflows can manage task dependencies using Directed Acyclic Graphs to ensure optimal execution order.In a DAG template, tasks without any dependency will be run immediately.

- **Event-driven Execution**: 
  Argo Events is an event-driven workflow automation framework capable of triggering Argo Workflows based on external events (for reactive workflow scenarios). it supports a significant list (more than 20) of event sources, including AMQP and Minio.
  Agro Evetns support of Minio enables to automate workflow execution based on changes within a bucket. 
  ![](D:\env\gitprojects\OHDSA\argo-worfklows\docs\design\sw_design_resources\argo-workflow-trigger.png)
  Argo Events does not only support the 'Submit' operation but also supports the following operations: Submit,Submit --from,Resubmit, Resume, Retry,Suspend,Terminate,Stop.
  For more information see: https://argoproj.github.io/argo-events/sensors/triggers/argo-workflow/

- **Resource Optimization**: Dynamically allocates resources based on task demands, optimizing cluster resource use.
  To achieve resource usage optimization, one can leverage Kubernetes resource mechanisms (limits and requests). 
When defining a workflow template, within the 'container' section, resourceRequirements can be defined: https://argo-workflows.readthedocs.io/en/stable/fields/#resourcerequirements
By specifying 'podSpecPatch' field in either the workflow or workflow template, one can override the resource limits and requests associated with a container: https://argo-workflows.readthedocs.io/en/stable/fields/#fields_123
To ensure a container is executed on a Kubernetes node with a specific hardware configuration, a nodeSelector can be defined both in the workflowSpecification or Workflow Template. The node selector should reference labels matching the labels defined on the desired node. On the other hand, Kubernetes nodes with specific hardware must use labels to advertise those specificities and leverage nodeSelector capabilities( see: https://argo-workflows.readthedocs.io/en/stable/fields/#template)

- **Error Handling and Retry Logic**: Argo Workflow provides built-in mechanisms for error recovery and task retries.
 This mechanism is called the 'retryStrategy' and can be defined in Workflow specification. The strategy is used to decide in which case a step is retried. In addition to the retry decision, it is possible to configure a 'backoff' mechanism in order to delay the retry and thus avoid retrying too soon or too frequently. https://argo-workflows.readthedocs.io/en/stable/retries/

- **Monitoring**: Argo produces metrics that provide information on the controller's status. As those metrics follow the same format as required by prometheus, those can be integrated with Prometheus. See: https://argo-workflows.readthedocs.io/en/stable/metrics/#default-controller-metrics

- **Logging**: 
It is officially not recommended to rely on Argo to archive logs as it is a naive solution, not designed for indexing, searching, and storing logs (see: https://argo-workflows.readthedocs.io/en/stable/configure-archive-logs/). 
In a Kubernetes environment, logs can be forwarded by an agent running on the node (see: https://kubernetes.io/docs/concepts/cluster-administration/logging/#using-a-node-logging-agent). 
This agent can forward logs to be saved and indexed for a future usage. Such solution can be provided by Fluentd (acting as the agent forwarding logs). Such logs can be forwarded to ElasticSearch (ELK) which supports storing, indexing and searching capabilities.
![](D:\env\gitprojects\OHDSA\argo-worfklows\docs\design\sw_design_resources\logging-with-node-agent.png)
- **Role-Based Access Control (RBAC)**: Utilizes Kubernetes RBAC to control access to workflow execution and management.

For more detailed information, visit the [official Argo Workflows documentation](https://argoproj.github.io/argo-workflows/).

The following subsections focus on the implementation of the **OHDSA specific capabilities** supported by the workflow engine.

#### API

> Note: TBD reference to API

#### Reusable Workflow Templates

> TBD: Workflow designer prepares reusable workflow templates for standardizing and streamlining reusable steps

#### DAG Workflow Creation

Workflow designer creates workflows using a Directed Acyclic Graph (DAG) model.

#### Data Artefacts

> TBD interfacing with fast / slow storage areas on S3 and management of artefacts 

- Steps I/O Mappings: Workflow designer maps intermediary steps data outputs to inputs steps of the workflow.
- Data Transfer Steps: Workflow designer integrates data stage-in and stage-out steps managing interactions between persistent storage system (for external interfaces), intermediate data storage system (for intermediate steps) and databases.


#### Retry Strategy

> TBD Workflow designer implements retry strategies for failed or errored workflow steps

#### Hardware Constraints

> TBD: Resource requirements/limits and affinities

#### Artefact Persistence

> TBD:  Workflow designer mark specific artefacts for persistence, overriding their default temporary status


#### Retention Policies

> TBD:  Workflow engine operator configure retention policies to remove expired workflows automatically


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
