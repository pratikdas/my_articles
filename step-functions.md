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
Our state machine definition in ASL contains a collection of `state` objects. It has the following mandatory fields:
* `States`: This field contains a set of `state` objects. Each element of the set is a key-value pair with the name of the state as `key` and an associated `state` object as the value. 
* `StartAt`: This field contains the name of one of the state objects in the `States` collection from where the state machine will start execution.

This structure has the `States` field containing the collection of all the state objects with names like: `check inventory`, `cancel order`, `update inventory`, etc. The value of the field: `StartAt` is `check inventory` which means that the state machine starts execution from the state named `check inventory`.

Each state object has a `Type` attribute for the type of the state and a `Next` attribute. The `Next` attribute contains the name of the next state that the state machine will execute. 

Other attributes of the state object are dependent on the type of the state. In this example, for each of the state of type `Task`, we have defined a attribute `Resource` with a value of `arn:aws:states:::lambda:invoke` to represent the API to be called. 

We have defined the parameters of the API in an attribute `Parameters` which takes a set of key-value pairs. 

We can see the keys: `FunctionName` and `Payload.$`. The `FunctionName` key has a value of the name of the Lambda function while the key `Payload.$` contains an expression to determine the input to be passed to the Lambda function during execution of the state machine. 

We will understand passing and manipulation of the input and output by the various states during execution of the state machine in the next section.

## Data Manipulation with Input and Output Filters
We can invoke state machines asynchronously or synchronously depending on whether the type of workflow is `standard` or `express`. 
Step Functions receive input in JSON format which is then passed to the different states in the state machine. We can configure different kinds of filters to manipulate data in each state both before and after the task processing as shown in this diagram:
![Input and Output Filters](images/in_out_filters.png)

Let us understand how we can use these filters by applying them to the different states of the state machine of our order fulfillment process. Let us suppose that our order processing workflow takes the following input:

```json
{
    "order_processing_request": {
        "customer": {

        },
        "item": {
                "item_no": "I1234",
                "num_of_items": 5,
                "shipping_date": "23/12/2022",
                "shipping_address": "address_1"
        },
        "order_details" : {
                "order_id": "ORD345567",
                "order_date": "15/12/2022"
        }    
    }
}
```
The input data consists of information of the customer who has placed the order, the item for which the order is placed, and the order details. This input is fed to the first state: `check inventory`. The state machine will execute the lambda function: `check inventory` associated with this task.

Here is the code for the Lambda function for `check inventory`:

```js
exports.handler = async (event, context, callback) => {
    const item_no = event.item_no
    const num_of_items = event.num_of_items
    console.log(`item::: ${item_no} ${num_of_items}`)
    
    const items_in_inventory = genItemsFromInventory(item_no)
   
   // TODO fetch inventory info from database
    const inventory = {item_no: item_no, num_of_items_in_inventory: items_in_inventory}
    
    
    callback(null, inventory)
}

function genItemsFromInventory(item_no) {  
    var rand = Math.random()*100
    var power = Math.pow(10, 0)
    return Math.floor(rand*power)
}

```
It takes the `item` information as input.
We will prepare the input for the Lambda function using two filters:
1. `InputPath`
We have set this filter as `$.item`
This filter will extract the `item` attribute from the input of the state machine. Here is the result of applying this filter:

```json
{
    "item_no": "I1234",
    "num_of_items": 5,
    "shipping_date": "23/12/2022",
    "shipping_address": "address_1"
}
```
2. `Parameter`: This filter will prepare the input required by the Lambda function.
```json
{
  "item_no.$": "$.item_no",
  "num_of_items.$": "$.num_of_items"
}

```

Here is the result of applying the filters `InputPath` and `Parameter` to the state input:

```json
{
    "item_no": "I1234",
    "num_of_items": 5
}
```

This is the input payload which will be used by the state machine to execute the task associated with this state.
The result of the execution of Lambda function: `checkInventory` is:
```json
{
  "sku": "S0001",
  "quantity_in_stock": 84,
  "warehouse_no": "W001",
  "age_of_stock_in_days": 98
}
```
If we do not apply any more filters, this payload will be passed on to the next state. In that case we will lose the original input data containing `customer` and `order` information which will be required to execute the remaining states of the state machine. 
For preserving the original input and extract the relevant fields from the task result, let us add some more filters:

1. `ResultSelector`:

2. `ResultPath`:

3. `OutputPath`:

The state `check inventory` after adding these filters looks like this:

```json
{
  "Comment": "order processing",
  "StartAt": "check inventory",
  "States": {
    "check inventory": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "OutputPath": "$",
      "Parameters": {
        "FunctionName": "arn:aws:lambda:us-east-1:926501103602:function:checkInventory:$LATEST",
        "Payload": {
          "item_no.$": "$.item_no",
          "num_of_items.$": "$.num_of_items"
        }
      },
      "Next": "items available?",
      "InputPath": "$.order_processing_request.item",
      "ResultSelector": {
        "num_items_in_inventory.$": "$.Payload.quantity_in_stock",
        "item_sku.$": "$.Payload.sku"
      },
      "ResultPath": "$.task_result"
    },
```

## Handling Errors in Step Function Workflows
In absence of any error handling, the execution of a state machine will fail whenever a state reports an error. States of type: `Task` provides options for configuring a retry and fallback for handling errors.

### Retrying on Error
We configure retry by defining one or more retry rules, called "retriers". This will allow the task to be retried for execution when errors occur during execution of the task.

{{% image alt="Error handling Retry" src="images/posts/aws-step-function/retry-error.png" %}}
Coming to our example, our Lambda function can encounter errors of type: `Lambda.ServiceException`, `Lambda.AWSLambdaException`, or `Lambda.SdkClientException`.
To retry the task, when these errors occur, we have defined a `retrier` with the following retry settings:
* **Interval**: The number of seconds before the first retry attempt. It can take values from `1` which is default to `99999999`.
* **Max Attempts**: The maximum number of retry attempts. The task will not be retried after the number of retries exceeds this value. `MaxAttempts` has a default value of `3` and maximum value of `99999999`.
* **Backoff Rate**: It is the multiplier by which the retry interval increases with each attempt.

### FallBack to a Different State on Error
We can catch and revert to a fallback state when errors occur by specifying one or more catch rules, called "catchers".
{{% image alt="Error handling Retry" src="images/posts/aws-step-function/catch-error.png" %}}
In this example, we are defining a `prepare error` state to which the `process payment` state can fall back if it encounters an error of type: `States.TaskFailed`.

The state machine with a `catcher` defined for the `process payment` step looks like this in the visual editor.
{{% image alt="Error handling catcher in workflow" src="images/posts/aws-step-function/workflow-with-catch.png" %}}

### Handling Lambda Service Exceptions
As a best practice, we should proactively handle transient service errors in AWS Lambda functions that result in a `500` error, such as `ServiceException`, `AWSLambdaException`, or `SdkClientException`. We can handle these exceptions by retrying the Lambda function invocation, or by catching the error.

