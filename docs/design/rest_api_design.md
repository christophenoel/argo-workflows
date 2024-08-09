# Argo Workflows REST API

## Table of Contents
- [Interface Overview](#interface-overview)
- [Interface Type](#interface-type)
- [Producers and Consumers](#producers-and-consumers)
- [Operations](#operations)
- [Data Model](#data-model)

## Interface Overview

The Argo Workflows API provides a robust platform for defining, managing, and executing complex workflows on Kubernetes. It is designed to enable users to automate the orchestration of containerized tasks, integrating seamlessly with other systems and services within a Kubernetes environment. 

The API allows for the creation, submission, monitoring, and management of workflows, facilitating tasks such as data processing, CI/CD pipelines, and machine learning workflows. Argo Workflows is designed to be highly scalable and flexible, supporting a wide range of use cases and providing extensive customization options through its API.

## Interface Type

- **Description**: REST API
- **Communication Protocols**: HTTP/HTTPS

## Producers and Consumers

Producers:

- CI/CD systems that need to automate deployment pipelines.
- Data processing systems that require complex task orchestration.
- Machine learning platforms that automate model training and deployment workflows.
- Users and developers who define and manage workflows through the API.

Consumers:

- Monitoring systems that track the status and progress of workflows.
Logging systems that aggregate and analyze workflow logs.
Notification services that alert users about workflow events and statuses.
Reporting tools that generate insights based on workflow executions.
Users and administrators who retrieve workflow status, logs, and results through the API.


## Operations

Detail each operation provided by the API, including method names, request and response formats, and a brief description of their functionality.

### Example Operation: GetItem
- **HTTP Method**: `GET`
- **Endpoint**: `/items/{id}`
- **Description**: Retrieves an item by its unique ID.
- **Request Parameters**:
    - `id` (path): The unique identifier for the item.
- **Response**:
    - **Success (200 OK)**:
      ```json
      {
        "id": "item1",
        "name": "Example Item",
        "description": "A sample item in the catalog."
      }
      ```
    - **Error (404 Not Found)**:
      ```json
      {
        "error": "Item not found."
      }
      ```
### Workflow Template creation 
The following operation is used to register a reusable workflow template by providing the target namespace as a path parameter and the workflow template in the request's body.

> <span style="background-color: #28a745; color: white; padding: 2px 6px; border-radius: 3px;">POST</span> /api/v1/workflow-templates/{namespace} 

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflow template.  |
| body          | body     | object     | true         | Workflow template object to create.  |


### List available Workflow Templates

The following operation list the reusable workflow templates already available. This request requires to provide the namespace as a path parameter, other parameters must be provided as query parameters and are optionals parameters used to filter the list returned by this operation.

> <span style="background-color: #007bff; color: white; padding: 2px 6px; border-radius: 3px;">GET</span> /api/v1/workflow-templates/{namespace}

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflow template.  |


### Describe Workflow Template

The following operation provide information about a specific workflow template.

> <span style="background-color: #007bff; color: white; padding: 2px 6px; border-radius: 3px;">GET</span> /api/v1/workflow-templates/{namespace}/{name}

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflow template.  |
| name          | path     | string     | true         | Name of the workflow template.       |
 |


### List Workflows

The following operation list all workflows submitted in a specific namespace.

> <span style="background-color: #007bff; color: white; padding: 2px 6px; border-radius: 3px;">GET</span> /api/v1/workflows/{namespace}

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflows.          |


### Submit Workflow

The following operation submit a workflow for execution in the namespace specified.

> <span style="background-color: #28a745; color: white; padding: 2px 6px; border-radius: 3px;">POST</span> /api/v1/workflows/{namespace}

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflows.          |
| body          | body     | object     | true         | Workflow object to submit.           |


### Delete Workflow

The following operation deletes a workflow in the namespace specified.

> <span style="background-color: #dc3545; color: white; padding: 2px 6px; border-radius: 3px;">DELETE</span> /api/v1/workflows/{namespace}/{name}

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflow.           |
| name          | path     | string     | true         | Name of the workflow.                |


### Retrieve Workflow Status

The following operation retrieves the status of the desired workflow running in the namespace specified.

> <span style="background-color: #007bff; color: white; padding: 2px 6px; border-radius: 3px;">GET</span> /api/v1/workflows/{namespace}/{name}/status

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflow.           |
| name          | path     | string     | true         | Name of the workflow.                |


### Retrieve Workflow Logs

The following operation retrieves the logs of the desired workflow running in the namespace specified. To retrieve logs from the desired step, specify the 'logOptions.container' query parameter with the container name corresponding to the desired step.

> <span style="background-color: #007bff; color: white; padding: 2px 6px; border-radius: 3px;">GET</span> /api/v1/workflows/{namespace}/{name}/logs

| **Parameter** | **In**   | **Type**   | **Required** | **Description**                      |
|---------------|----------|------------|--------------|--------------------------------------|
| namespace     | path     | string     | true         | Namespace of the workflow.           |
| name          | path     | string     | true         | Name of the workflow.                |



## Data Model

The full reference of the data model is provided on https://argo-workflows.readthedocs.io/en/latest/fields/
