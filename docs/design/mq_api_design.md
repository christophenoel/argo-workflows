# Argo Workflows Message Queue Interface

## Table of Contents
- [Interface Overview](#interface-overview)
- [Interface Type](#interface-type)
- [Producers and Consumers](#producers-and-consumers)
- [Operations](#operations)
- [Data Model](#data-model)

## Interface Overview

The Argo Workflows Message Queue Interface provides a robust platform for defining, managing, and executing complex workflows on Kubernetes through message-driven interactions. It allows users to trigger workflows, receive updates, and manage workflows asynchronously using message queues, integrating seamlessly with other systems and services that support message queuing protocols.

The interface supports the automation of tasks such as data processing, CI/CD pipelines, and machine learning workflows by allowing event-driven execution and management. It is designed to be highly scalable and flexible, supporting a wide range of use cases and providing extensive customization options through message-based communication.

## Interface Type

- **Description**: Message Broker Interface
- **Communication Protocols**: AMQP, MQTT

## Producers and Consumers
- **Producers**:
    - Event sources that trigger workflow executions (e.g., CI/CD events, data processing events, IoT sensors).
    - Users and systems that send messages to manage workflows.
- **Consumers**:
    - Workflow engines that execute and manage workflows based on received messages.
    - Monitoring systems that track the status and progress of workflows through message subscriptions.
    - Notification services that alert users about workflow events and statuses.
    - Reporting tools that generate insights based on workflow executions received via messages.

## Operations
Detail each operation provided by the message queue interface, including method names, request and response formats, and a brief description of their functionality.

### Example Operation: Trigger Workflow
- **Message Topic**: `argo/workflows/trigger`
- **Description**: Triggers the execution of a workflow.
- **Message Payload**:
    ```json
    {
      "namespace": "example-namespace",
      "workflow": {
        "metadata": {
          "name": "example-workflow"
        },
        "spec": {
          "entrypoint": "main",
          "templates": [
            {
              "name": "main",
              "container": {
                "image": "example/image",
                "command": ["echo"],
                "args": ["hello world"]
              }
            }
          ]
        }
      }
    }
    ```
- **Response**:
    - **Success (200 OK)**:
      ```json
      {
        "status": "Workflow triggered successfully",
        "workflowId": "example-workflow-id"
      }
      ```
    - **Error (400 Bad Request)**:
      ```json
      {
        "error": "Invalid workflow definition"
      }
      ```

### Example Operation: Workflow Status Update
- **Message Topic**: `argo/workflows/status`
- **Description**: Provides updates on the status of a workflow.
- **Message Payload**:
    ```json
    {
      "namespace": "example-namespace",
      "workflowId": "example-workflow-id",
      "status": "Running",
      "progress": "50%",
      "message": "Workflow is halfway through execution"
    }
    ```
- **Response**:
    - **Success (200 OK)**:
      ```json
      {
        "status": "Status update received successfully"
      }
      ```
    - **Error (400 Bad Request)**:
      ```json
      {
        "error": "Invalid status update format"
      }
      ```

## Data Model

The full reference of the data model is provided on https://argo-workflows.readthedocs.io/en/latest/fields/
