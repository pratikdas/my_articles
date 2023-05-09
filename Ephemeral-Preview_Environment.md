Building Ephemeral Test Environments for concurrent Testing
## Key Takeaways

* An Ephemeral environment is used to test our applications by spinning up the environment at the time of testing and destroying it thereafter.
* The environment is a complete stack of applications and fully functional consisting of the application under test along with all its dependencies: upstream services, databases, queueing systems, etc
* They replace traditionally used static shared environments like integration or staging environments which worked well for monolithic applications. 
* They are easier to set up when all our applications are on containers
* They should also include spinning up database and populating with synthetic data.

In this article, we will understand the concepts of ephemeral environments, when they are most useful and the different ways of setting them up.

## A typical workflow Traditional Static Environments: Dev, SIT, UAT, Prod
- Developers checkout code from source control repository
- They write code and test in a local dev environment which is usually their desktops where the application runs
- Once they are happy with their changes they promote it to a series of higher environments like SIT, UAT/pre-prod and eventually to production.
- The process of promotion is usually creating a Pull request which is reviewed and merged to a release branch. 

Let us look at few of the challenges with these environments.

## Challenges of Static Environments: Dev, SIT, UAT, Prod
Ephemeral environments replace traditionally used static shared environments like integration or staging environments. Shared environments worked well in the past when usually 1 big monolithic application was present with probably a database. Once we broke down this monolith into several microservices and multiple teams wait for the other team to finish testing their changes.  working on those  But teams often face challenges of instability and limited availability due to the deployment of breaking changes by other teams or if a release is already being tested.

- **Test multiple versions in parallel**: I can test 1 version at a time. If two teams are working in parallel on the same source code on multiple features, only one team can deploy their changes and the other team has to wait for the environment to be available.
- **Environment Stability**: Due to multiple teams deploying at the same time, the changes deployed by one team might inadvertantly break the dependent application.

These are some of the challenges we try to address by adopting ephemeral or dynamic environment.

## Introducing Ephemeral Environment
An ephemeral environment as the name suggests is a temporary short lived environment. It is also called dynamic or preview environments. It consists of infrastructure and a full stack of applications required for .  These are similar to docker containers but extended to a collection of applications. And instead, it only exists for a defined period of time. An environment is a collection of infrastructure or services to enable someone whether it be a development team or otherwise to deploy their components or applications into. 

Some of the features of an ephemeral environment are:
- **Tied to a branch**: They are tied to a feature branch
- **Isolation**: The environment is isolated from other set of changes
- **Infrastructure**: The infrastructure used for running the environment is also spun up on demand.
- **Applications**: The applications in our dependency graph are present

## Setting up Ephemeral Environment


## When do we spin up an Ephemeral Environment
- Provision a lighter development environment: Spun up by developers with only the dependent systems required by the application they need to work on. Advantage of solving "it runs on my machine" syndrome. Similar to Docker containers but extened to a collection of systems.

- Provision an integration environment: Spun up by developers for testing in a production like environment
- Provisioned by QA teams: To test a feature for functional testing.
- Business teams
- Performance testing

## Tools & Techniques
IaC:
CI/CD Pipelines:
Kubernetes Operator:

Completely Isolated stack
Sharing common dependencies. Traffic rerouting
## Complexities
Dependencies:

## Ephemeral Environment can be extended to Production
These environments are not limited to non-prod but can be adopted for production also

## Conclusion
In this article we made a case for moving away from traditional static environments like SIT, UAT, and pre-prod to ephemeral environments which can be created on the fly. They are an essential component in the toolkit of modern devops teams which are constantly looking out for ways to increasing the velocity of building and rolling out changes to production. However setting up ephemeral environments is complex. It needs well defined vision and adoption plan by teams working on the ground to change their ways of working.
