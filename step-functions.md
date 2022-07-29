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


## Order Fulfilment
Let us look at different types of states with an example of order fulfilment process. A typical order fulfilment process uses a workflow similar to this:

![Order Fulfillment Workflow](images/order_fulfillment_workflow.png)
Let us define this workflow with a Step Functions state machine:

the first step of the state machine is `check inventory`. We will add this as a state type: `Task`. This will be a Lambda function.

We will add the first step of the state machine 



## Data Manipulation with Input and Output Filters

