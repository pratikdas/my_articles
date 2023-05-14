## Key Takeaways

* Service discovery is the mechanism of locating the endpoints of services present within a network.
* Consul is a product from HashiCorp with service discovery as one of its main capabilities
* Datacenters, Clusters, and Agents are some key concepts of Consul architecture. 
* A datacenter is the smallest unit of infrastructure that can perform basic Consul operations.Agents are daemons created when we run the consul binary. They can be run as server and client. A collection of server and client agents form a cluster.
* Client and server agents participate in a LAN gossip pool so that they can distribute and perform node health checks.
* We define services in a configuration file and upload that to an agent to register the services.
* The Consul DNS is the primary interface for discovering services registered in the Consul catalog.
* We can use DNS to reach services registered with Consul or modify your application to natively consume the Consul service discovery HTTP APIs.

In this article, we will understand the concepts of service discovery using HashiCorp Consul with the help of some examples.
## Why do we need Service Discovery
A typical infrastructure consists of resources of all kinds
We rarely work with fixed IPs these days. Resources spun up dynamically and have an IP allocated. Traditionally we used Load Balancer with a DNS which was front ending a collection of resources. Using a Load Balancer is costly in terms of money, configuration and performance. Consul takes a different appproach. It uses looks up a registry to find the service endpoint and makes a direct call using that endpoint address. Consul also has service mesh capabilities. It can be injected as a side car in kubernetes pod and allows the microservice to communicate to other microservices using the side car proxy. In this article we will look at how consul works with services deployed in VMs.
## Our Discoverable Assets
VMs
Services running on VMs
Microservice apps in Kubernetes cluster

All of these have endpoints. Each endpoint has a IP or DNS name and a port.
In the next section we will see how Consul helps to discover these endpoints 
## How is Consul used for Service Discovery 
Our 
The Consul infrastructure is set up in a Cluster as a collection of nodes. For the timebeing consider a node as a VM for simplicity. Each node runs a process called Consul agent. When we running consul, we actually run the consul agent in a VM. The Consul can span across multiple data centers and for delivering a scalable, highly available, abstract, and resilient service mesh infrastructure. A Consul deployment may span multiple zones or even regions in a public cloud environment.
## Spining up a Consul Infrastructure
## Introducing Consul Agent- the Core Process of Consul
Consul agent is the core process of Consul. The main functions of an agent are:  
- maintaining membership information, 
- registering services 
- running health checks 
- responding to DNS queries

The agent must run on every node that is part of a Consul cluster.

Agents run in either client or server mode. Client nodes are lightweight processes that make up the majority of the cluster. They interfa


This encryption key generated is set as the value for the "encrypt" key in the server and client configuration files.

## Starting the Consul agent
Let us create 2 directories:
Directory for config: consul/config
Directory for data: consul/data
Name of datacenter
Name of domain: 
Https port:8443
DNS Port: 8600
DNS Recursor: 1.1.1.1

The config file will look like this:

```hcl
datacenter = "my-dc1"
data_dir = "~/consul/data"
log_level = "INFO"
node_name = "node-alpha"
server = true
encrypt = ""
ui_config {
  enabled = true
}
# Addresses and ports
addresses {
  grpc = "127.0.0.1"
  https = "0.0.0.0"
  dns = "0.0.0.0"
}

ports {
  grpc  = 8502
  http  = 8500
  https = ${HTTPS_PORT}
  dns   = ${DNS_PORT}
}
```

Start a Consul agent with the consul command and agent subcommand using the following syntax:

```shell
$ consul agent -dev
```

Consul ships with a -dev flag that configures the agent to run in server mode and several additional settings that enable you to quickly get started with Consul. The -dev flag is provided for learning purposes only. We strongly advise against using it for production environments.

```shell
==> Starting Consul agent...
              Version: '1.15.2'
           Build Date: '2023-03-30 17:51:19 +0000 UTC'
              Node ID: '2ac840b1-b0fb-60bd-ef7d-3c53058cc04a'
            Node name: 'ip-172-31-57-153.eu-central-1.compute.internal'
           Datacenter: 'dc1' (Segment: '<all>')
               Server: true (Bootstrap: false)
          Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: 8502, gRPC-TLS: 8503, DNS: 8600)
         Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
```
They also report that the agent is running as a server and has claimed leadership

## Gossip Protocol
Consul uses a shared secret for implementing the gossip protocol. The shared secret is generated by running the command:
```shell
consul keygen
```
## Registering Services in Consul

## Join the Agent to the Cluster

Consul is available as a binary. Let us install consul in 3 aws ec2 instances.

Start the agent

Launch the UI on port 8500.


## Registering Services in Consul
Registering services on startup because the service persists if the agent fails. Specify the directory containing the service definition with the -config-dir option on startup. When the Consul agent starts, it processes all configurations in the directory and registers any services contained in the configurations. 
```shell
consul agent -config-dir configs
```
In the following example, the Consul agent starts and loads the configurations contained in the configs directory.

## Discovering Services From Consul
Services can be discovered either by using the HTTP API or using DNS.

### Look up via HTTP

### Look up via DNS
DNS is a canonical way to discover services. While Consul provides basic DNS services, the integration with other DNS services like NS1 allows for service discovery to occur over their advanced DNS network. 
The DNS interface allows applications to make use of service discovery without any high-touch integration with Consul.

For example, instead of making HTTP API requests to Consul, a host can use the DNS server directly via name lookups like redis.service.us-east-1.consul. This query automatically translates to a lookup of nodes that provide the redis service, are located in the us-east-1 datacenter, and have no failing health checks.
There are a few ways to use the DNS interface:
1. Using a custom DNS resolver library: point it at Consul.
2. Set Consul as the DNS server for a node:Provide a recursors configuration so that non-Consul queries can also be resolved.
3. Forward all DNS queries for the "consul." domain to a Consul agent from the existing DNS server
## Conclusion
In this article, we looked at the main concepts of HashiCorp Consul. We understood how to set up the cluster, register services and discover service endpoints.


