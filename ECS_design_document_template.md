# ECS Design and Implementation Document

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [ECS Design and Implementation Document](#ecs-design-and-implementation-document)
    - [Branch](#branch)
    - [Introduction](#introduction)
    - [Features](#features)
    - [References](#references)
    - [Glossary](#glossary)
    - [Limitations](#limitations)
    - [Architecture and Design description](#architecture-and-design-description)
        - [Working flow](#working-flow)
        - [Overview of our ECS implementation](#overview-of-our-ecs-implementation)
        - [Dependencies](#dependencies)
        - [`alto-provider` Overview](#alto-provider-overview)
        - [`alto-network` Overview](#alto-network-overview)
    - [Services APIs/User Interface](#services-apisuser-interface)
        - [Northbound APIs defined by RFC7285](#northbound-apis-defined-by-rfc7285)
        - [User defined routing cost computation](#user-defined-routing-cost-computation)
        - [User defined routing service](#user-defined-routing-service)
    - [TODO List](#todo-list)

<!-- markdown-toc end -->

---

## Branch

Now we have two branches of our ECS implementation. One is totally compatible with OpenDayLight (ODL), which located in `master`. The other branch `feature/ecs` has the latest code. These codes can still work with original ODL but it works well with our modified L2Switch module because OpenDayLight couldn't provide enough information for ECS.

## Introduction

ECS is the abbreviation of `Endpoint Cost Service`, which provides the routing cost between endpoints. ECS accepts the pair of endpoints and returns the routing cost of the pair. It provides the routing cost by collecting raw networking data from OpenDayLight with different metrics such as hop counts, bandwidth and user-defined routing cost. This design document contains the information about our latest ECS implementation.


## Features

The features that provides now:

* Hop count:

    Count the number of hops between the given source and destination based on the routing path.

* Routing cost:

    Compute the routing cost between the given source and destination based on the routing path.

* Bandwidth:

    Compute the available bandwidth between the given source and destination based on the routing path.
	

## References

* [RFC7285](https://tools.ietf.org/html/rfc7285): Application-Layer Traffic Optimization (ALTO) Protocol
* [L2 Switch](https://wiki.opendaylight.org/view/L2_Switch:Main): Dependency of ECS 

## Glossary

* Switching: 
	The packets are moved within the same network and are forwarded based on the destination MAC address.
	
* Routing: 
	The packets can be transferred between networks using IP address to determine the destination.
	
* Layer 2 Switch:
	A L2 switch does switching only. It uses mac address to switch the packets from a port to the destination port.
	
* Layer 3 Switch:
	A L3 switch also does switching exactly like a L2 switch, while it is capable of having IP address and doing routing.
	
* Endpoint: 
	An endpoint is an application or host that is capable of communicating (sending and/or receiving messages) on a network. An endpoint is typically either a resource provider or a resource consumer.

## Limitations

* OpenDayLight
  * ODL doesn't provide general routing service to handle global routing or switching. So ECS relies on an extra module to gather routing information.

* OpenFlow
  * Packet counter

* ALTO
  * Can't handle multi-path or load balancing scenario.
  * Do not have fail-safe mechanism to get path.

## Architecture and Design description

### Workflow

The general method we use in ECS is:
1. Find the actual routing paths which packet will go through;
2. Compute the routing cost by given paths and link properties.

The fist step in our implementation is that ECS acquires the actual routing path between each endpoint pair. We use the algorithm shown as below to calculate paths. Notice that the paths may be multi-path for some reason such as load balancing.

```
IF find a existing routing service in ODL:
    compute routing path by invoke routing service;
ELSE
    compute routing path by looking up flow table;
```

Following is the algorithm of computing routing path by looking up flow tables of each switch:

```
current_sw_id = get_attachment_point(src_ip)
dst_sw_id = get_attachment_point(dst_ip)
while (current_sw_id != dst_sw_id) {
  r <- loop_up_flow_table(sw_id, src_ip, dst_ip)
  if (!r) {
    force_compute_routing_path(sw_id, src_ip, dst_ip)
    r <- loop_up_flow_table(sw_id, src_ip, dst_ip)
  } 
  next_sw_id = get_next_sw_id(r)
  current_sw_id = next_sw_id
}
```

_We haven't completed force_compute_routing_path(sw_id, src_ip, dst_ip) because of the ODL's limitations._

Then we compute the routing cost.

```
SWITCH cost metric:
    CASE hopcounts: RETURN the length of shortest path in given multi-paths;
    CASE bandwidth: RETURN the maximal bandwidth in given multi-paths;
    CASE routingcost:
        IF user provides routing cost function RETURN the function's result;
        ELSE RETURN our default function's result;
    DEFAULT:
        RETURN null;
```

### Overview of our ECS implementation

ECS has two modules in current ALTO implementation, `alto-provider` module, which contains ECSImplementation, and `alto-network` module, which contains the supported codes of ECS. And this ECS depends on a modified L2 Switch in OpenDayLight, which will be discussed in next section. 

### Dependencies

* A modified L2 Switch:

	We use a modified L2 Switch from ODL because two reason. First is that ODL didn't provide a routing service or another switching module. So ODL's default selection is L2 Switch. Second is that the L2 Switch from ODL use Spain Tree Protocol (STP) as default switching protocol. It's not good enough because STP cause flooding. Each packet in the network managed by L2 Switch would forward to all hosts so the network will be inefficient. Since L2 Switch has legacy codes from Hydrogen release, which use shortest path algorithm to switching. We modify the L2 Switch to enable Dijkstra algorithm using the legacy codes. And we also provide a routing service in L2 Switch to calculate the real path between endpoints.

### `alto-provider` Module Overview

The alto-provider module implements the ECS RPC defined in alto-model. The core functions of computing endpoint cost between a endpoint pair are in class ```org.opendaylight.alto.provider.ecs.basic.BasicECSImplementation```. 

To compute the endpoint cost, it will first invoke the routing path query service provided by alto-network module to get the routing path, and then compute the endpoint cost according to the routing path, link properties and other metrics such as cost type and cost metric given by user. Currently we support the following cost mode and cost metrics:

```json
{"cost-mode": "numerical", "cost-metric":"hopcount"}
{"cost-mode": "numerical", "cost-metric":"bandwidth"}
{"cost-mode": "numerical", "cost-metric":"routingcost"}
```

Both routing path query service and link property query service are defined in module alto-network which will be described detailed in later section.

### `alto-network` Module Overview

alto-network module provide two services, the `__Network Element Service__` and the `__Routing Path Query Service__`.

#### Network Element Query Services

Network Element Query Services support queries of three different network element, including host nodes, flow capable nodes and links, as well as properties of them.

##### Interface

Following are the interfaces defined for the three network element service:

__NetworkLinkService__

```java
public interface NetworkLinkService {

    public Link getLinkByLinkId(String linkId);

    public List<Link> getLinksBySourceNodeId(String srcNodeId);

    public List<Link> getLinksByDestinationNodeId(String dstNodeId);

    public Link getLinkBySourceTpId(String srcTpId);

    public Link getLinkByDestinationTpId(String dstTpId);
}

```
__NetworkHostNodeService__

```java
public interface NetworkHostNodeService {

    public HostNode getHostNodeByHostIP(TypedEndpointAddress ip);

    public HostNode getHostNodeByHostId(String hostId);

    public boolean isValidHost(TypedEndpointAddress ip);
}

```
__NetworkFlowCapableNodeService__

```java
public interface NetworkFlowCapableNodeService {

	public FlowCapableNode getFlowCapableNode(String nodeId);

    public FlowCapableNodeConnector
    	getFlowCapableNodeConnector(String tpId);

    public FlowCapableNodeConnectorStatistics
    	getFlowCapableNodeConnectorStatistics(String tpId);

    public Long getAvailiableBandwidth(String tpId);

    public Long getCapacity(String tpId);

    public Long getConsumedBandwidth(String tpId);
}

```

##### Implementation

All implementations are in package ```org.opendaylight.alto.network.topology.impl```. We registered listeners for each link, host node and flow capable node and maintained our own data structures in memory in order to query the network elements faster.


#### Routing Path Query Service

With the help of APIs defined above and their implementations, we defined and implemented following routing path query service. Routing Path Query Service aims to provide routing path calculation given some packet fields. 

##### Interface

Since the path calculated could be a graph instead of a line, we define class LinkNode to represent the node in the graph. Packets fields are wrapped in instance of class __MatchFields__.

```java
public interface RoutingPathService {

	public LinkNode buildRoutePath(MatchFields matchFields);

}
	
```

The interface for RoutingPath query is quite generic now and we are going to extend it in the future to support varies customized routing services. e.g. We will add buildRoutingPathForLayer3(MatchFields matchFields) to get layer3 routing path.

##### Implementation

We implemented the routing path query service by implementing the algorithm we described in __Workflow__ section above.

## Services APIs/User Interface

### Northbound APIs defined by RFC7285

Please see details from [RFC7285](https://tools.ietf.org/html/rfc7285)

## TODO List

* User defined routing cost computation

	Make the routing cost computation flexible enough to support custom cost types and cost metrics.

* A general routing service provider module in ODL: 

	OpenDayLight provides insufficient details of routing information. It even DO NOT have a general routing service. So currently we manage to modify the original l2switch module to get actual routing between endpoints.
  
* Make ECS independently: 

	Now ECS depend on L2 Switch and OpenDayLight. It's not a good design because ECS may gather data from multiple information sources. The ECS should only depend on the data not on the module.
