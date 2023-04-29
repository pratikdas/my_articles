## Key Takeaways

* Service discovery is the mechanism by which a service finds out the address of dependent services and uses that to invoke those services.
* Consul is a product from Hashicorp with service discovery as one of its main capabilities
* Datacenters, Clusters, and Agents are some key concepts of Consul architecture. 
* A datacenter is the smallest unit of infrastructure that can perform basic Consul operations.Agents are daemons created when we run the consul binary. They can be run as server and client. A collection of server and client agents form a cluster.
* Client and server agents participate in a LAN gossip pool so that they can distribute and perform node health checks.
* We define services in a configuration file and upload that to an agent to register the services.
* The Consul DNS is the primary interface for discovering services registered in the Consul catalog.
* We can use DNS to reach services registered with Consul or modify your application to natively consume the Consul service discovery HTTP APIs.

In this article, we will understand the concepts of service discovery using Hashicorp Consul and apply those to build a service to service communication between 3 microservices running in VMs.
## Architecture of Consul on VMs


## Introducing Consul Agent- the Core Process of Consul
Consul agent is the core process of Consul. The agent maintains 
- membership information, 
- registers services, runs checks, responds to queries, and more.
The agent must run on every node that is part of a Consul cluster.

Agents run in either client or server mode. Client nodes are lightweight processes that make up the majority of the cluster. They interfa

## Starting the Consul agent

Start a Consul agent with the consul command and agent subcommand using the following syntax:

```shell
$ consul agent <options>
```

Consul ships with a -dev flag that configures the agent to run in server mode and several additional settings that enable you to quickly get started with Consul. The -dev flag is provided for learning purposes only. We strongly advise against using it for production environments.

## Join the Agent to the Cluster

Consul is available as a binary. Let us install consul in 3 aws ec2 instances.

Start the agent

Launch the UI on port 8500.


## Registering Services
 registering services on startup because the service persists if the agent fails. Specify the directory containing the service definition with the -config-dir option on startup. When the Consul agent starts, it processes all configurations in the directory and registers any services contained in the configurations. 
```shell
consul agent -config-dir configs
```
In the following example, the Consul agent starts and loads the configurations contained in the configs directory.

## Discovering Services
DNS is a canonical way to discover services. While Consul provides basic DNS services, the integration with other DNS services like NS1 allows for service discovery to occur over their advanced DNS network. 

## Conclusion
In this article, we looked at the main concepts of AWS Step Function and used those to define a workflow for an Order Fulfillment process. Here are the main points from the article for quick reference:


