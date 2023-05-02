Building Ephemeral Test Environments for concurrent Testing
## Key Takeaways

* An Ephemeral environment is used to test our applications by spinning up the environment at the time of testing and destroying it thereafter.
* The environment is a complete stack of applications and fully functional consisting of the application under test along with all its dependencies: upstream services, databases, queueing systems, etc
* They replace traditionally used static shared environments like integration or staging environments which worked well for monolithic applications. 
* They are easier to set up when all our applications are on containers
* They should also include spinning up database and populating with synthetic data.

In this article, we will understand the concepts of ephemeral environments, when they are most useful and the different ways of setting them up.

## What do we aim for in our Test Environments
Some of the things we aim for in our test environment are:
**Isolation**: The environment is isolated from other set of changes
**Applications**: The applications in our dependency graph are present
**Tied to a feature branch**: They are tied to a feature branch

## Challenges of Static Environments
Advantage: Production like
Shared

## Introducing Ephemeral Environment

## Setting up Ephemeral Environment

## When do we spin up a Ephemeral Environment


## Conclusion
