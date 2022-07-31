## Key Takeaways

* Coordinating between granular services is one of the common challenges when building complex applications.
* Traditionally orchestration was done in code (imperative style) or declaratively mostly using proprietary solutions.
* Step Functions is a serverless orchestration service in the AWS Cloud.
* We can build a wide variety of applications involving workflows like: orchestrate microservices, automate both IT and business processes, and build data and machine learning pipelines.
* Step Functions is a low-code integration service with buiding blocks for actions, flows, and filters for input and output data.
* Step Functions workflows also provide blocks for configuring failure management with retries and fallback, parallelizing more than one actions, integrate AWS services, and setting up observability mechanisms.
* We define the workflows in JSON format using a schema called ASL
* Step Functions also provide a visual interface: Studio to define, view, debug, and run workflows

In this article, we will understand the core concepts of AWS Step Functions and apply those to solve some commonly encountered use cases.

## AWS Step Function Implement a State Machine

AWS Step functions use a State Machine to represent the workflow. A workflow consists of a set of tasks, each of which represents a discrete activity to be performed.
 A State Machine is an abstract concept of representing the state of a system at a particular point of time.

We define the workflows in JSON format using a schema called ASL

```

```

Step Functions also provide a visual interface: Studio to define, view, debug, and run workflows.

We can create two types of state machine. State machine executions differ based on the type. The type of state machine cannot be changed after the state machine is created.
1. **Standard**: State machine of type: `Standard` should be used for long-running, durable, and auditable processes.
2. **Express**: State machine of type: `Express` is used for high-volume, event-processing workloads such as IoT data ingestion, streaming data processing and transformation, and mobile application backends. They can run for up to five minutes. 


## States in a State Machine

Each step of the orchestration is represented by a state in the state machine and connected to one or more states through transitions. A state takes an input, performs some action depending on the type of the state and emits an output to be passed on to the next state.

States of type `Task` perform the work in a state machine. They are configured as an API call to one of the AWS services. The parameters to the API are either specified in the definition of the state machine or supplied at runtime.

A Task state ("Type": "Task") represents a single unit of work performed by a state machine.

All work in a state machine is done by tasks. A task performs work by using an activity or an AWS Lambda function, or by passing parameters to the API actions of other services.



We define the orchestration logic in JSON syntax using a schema called Amazon States Language (ASL). The main element of the schema is the State object with the following structure:

```json
 {
 	"name" : "",
 	"type" : "",
 	"next" : ""
 }
```
Each The `next` field 


Each state can have a child state.

Some states perform work, others help navigate...

## Creating the State Machine
We can define a state machine from the Workflow Studio or the Amazon States Language (ASL) for defining our state machine. 
{{% image alt="checkout process" src="images/posts/aws-step-function/create_sm_1.png" %}}

Here we have selected Workflow Studio to author our state machine. We have also selected the type of state machine as `standard` because our order fulfillment process is a long running lasting more than 5 minutes. 

Let us also give a name: `checkout` to our state machine and assign an IAM role that defines which resources our state machine has permission to access during execution. Our IAM policy definition is associated with the following policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lambda:InvokeFunction"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
This policy will allow the state machine to invoke any Lambda function. 

## State Machine for Order Fulfilment

Let us look at different types of states with an example of order fulfilment process. A typical order fulfilment process uses a workflow similar to this:

![Order Fulfillment Workflow](images/order_fulfillment_workflow.png)

This workflow is triggered ofter order is placed by the customer and consists of the steps shown in the diagram. A Step Functions state machine to represent this workflow will look like this in the Workflow Studio:

![Order Fulfillment State Machine](images/order_fulfillment_sm.png)

As we can see, we have used the following states to define this state machine:

1. `Check Inventory` : This is a state of type: `Task` and invokes a Lambda function `check inventory`
2. `Cancel Order`: This is a state of type: `Task` and invokes a Lambda function `update order`
3. `Mark Order as complete`: This is a state of type: `Task` and invokes a Lambda function `update order`
4. `Update Inventory`: This is a state of type: `Task` and invokes a Lambda function `update inventory`
5. `items available`: This is a state of type: `Choice` with 2 branches. If items are not available the order is cancelled
6. `Parallel`: This is a state of type: `Parallel` with 2 branches which the state machine executes in parallel
7. `Success`: End the execution with a success
8. `Fail`: End the execution with a failure

We can also see a start and end indicator to define the start and end positions for the execution of the state machine.

The corresponding definition of the state machine in the Amazon States Language (ASL) looks like this:

```json
{
  "Comment": "order processing",
  "StartAt": "check inventory",
  "States": {
    "check inventory": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "OutputPath": "$.Payload",
      "Parameters": {
        "FunctionName": "arn:aws:lambda:us-east-1:**********:function:checkInventory:$LATEST"
      },
      "Next": "items available?"
    },
    "items available?": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.item.num_of_items",
          "NumericGreaterThanPath": "$.inventory.num_of_items",
          "Next": "cancel order"
        }
      ],
      "Default": "Parallel"
    },
    "Parallel": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "mark order as complete",
          "States": {
            "mark order as complete": {
              "Type": "Task",
              "Resource": "arn:aws:states:::lambda:invoke",
              "OutputPath": "$.Payload",
              "Parameters": {
                "Payload.$": "$",
                "FunctionName": "arn:aws:lambda:us-east-1:**********:function:updateOrder:$LATEST"
              },
              "End": true
            }
          }
        },
        {
          "StartAt": "update inventory",
          "States": {
            "update inventory": {
              "Type": "Task",
              "Resource": "arn:aws:states:::lambda:invoke",
              "OutputPath": "$.Payload",
              "Parameters": {
                "Payload.$": "$",
                "FunctionName": "arn:aws:lambda:us-east-1:**********:function:updateInventory:$LATEST"
              },
              "End": true
            }
          }
        }
      ],
      "Next": "Success"
    },
    "Success": {
      "Type": "Succeed"
    },
    "cancel order": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "OutputPath": "$.Payload",
      "Parameters": {
        "Payload.$": "$",
        "FunctionName": "arn:aws:lambda:us-east-1:**********:function:updateOrder:$LATEST"
      },
      "Next": "Fail"
    },
    "Fail": {
      "Type": "Fail"
    }
  }
}
```
This structure has the `States` field containing the collection of all the state objects with names like: `check inventory`, `cancel order`, `update inventory`, etc. The value of the field: `StartAt` is `check inventory` which means that the state machine starts execution from the state named `check inventory`.

Each state object has a `Type` attribute for the type of the state and a `Next` attribute. The `Next` attribute contains the name of the next state that the state machine will execute. 

Other attributes of the state object are dependent on the type of the state. In this example, for each of the state of type `Task`, we have defined a attribute `Resource` with a value of `arn:aws:states:::lambda:invoke` to represent the API to be called. 

We have defined the parameters of the API in an attribute `Parameters` which takes a set of key-value pairs. 

We can see the keys: `FunctionName` and `Payload.$`. The `FunctionName` key has a value of the name of the Lambda function while the key `Payload.$` contains an expression to determine the input to be passed to the Lambda function during execution of the state machine. 

We will understand passing and manipulation of the input and output by the various states during execution of the state machine in the next section.

## Data Manipulation with Input and Output Filters

