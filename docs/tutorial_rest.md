### REST Tutorial

In this section, we will see how to use the REST API for managing workflows.
For more information on those operations,see API design documentation: [API](./design/rest_api_design)

> Note: Authentication aspects are not covered in this tutorial. See https://argo-workflows.readthedocs.io/en/latest/argo-server-auth-mode/

Postman collection of requests used in this tutorial: [Postman DEMO Collection](images/argo_workflow_demo.postman_collection.json)

Open the Postman collection HELLOWOLRD-TEMPLATE-EXAMPLE

#### Register a Workflow template

In this tutorial, we will use the template located at [hello-world-wf-template.yml](../examples/hello-world-template/hello-world-wf-template.yml)

Once we have defined a workflow template in a YAML file, we can register this template by calling the REST API.

Open the Postman collection and select the request located in Template/ Create Workflow Template:

![img.png](images/tutorial-rest-register-template.png)

#### Submit a Workflow

Open the Postman request located in Workflow / Submit Workflow:

![img.png](images/tutorial-rest-submit-workflow.png)


#### Monitor a workflow

The REST API provide an operation to retrieve the status of a Workflow including the status of the pods deployed on the Kubernetes cluster.

Open the Postman collection and select the request located in Workflow/ Get Status:

![](D:\env\gitprojects\OHDSA\argo-worfklows\docs\design\api_design_resources\workflow-status.png)

To retrieve the logs associated with a workflow, a REST Operation also exists.

Open the Postman collection and select the request located in Workflow/ Get LOGS:

![](D:\env\gitprojects\OHDSA\argo-worfklows\docs\design\api_design_resources\workflow-logs.png)
