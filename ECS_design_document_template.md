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
        - [Northbound APIs defniened by RFC7285](#northbound-apis-defniened-by-rfc7285)
        - [User defiened routing cost computation](#user-defiened-routing-cost-computation)
        - [User defiened routing service](#user-defiened-routing-service)
    - [TODO List](#todo-list)

<!-- markdown-toc end -->

---

## Branch

Now we have two branches of our ECS implementation. One is totally compatible with OpenDayLight (ODL), which located in `master`. The other branch `feature/ecs` has the latest code. These codes can still work with original ODL but it works well with our modified L2Switch module beacuse OpenDayLight couldn't provide enough information for ECS.

## Introduction

ECS is the abbreviation of `Endpoint Cost Service`, which provides the routing cost between endpoints. ECS accepts the pair of endpoints returns the routing cost of the pair. It provide the routing cost, by collecting raw networking data from OpenDayLight, whith different metrics such as hop counts, bandwidth and user defiened routing cost. This design document contains the information about our latest ECS implementation.


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

* Switch: 
* Switching: The packets are moved within the same network and are forwarded based on the destination MAC address.
* Route: 
* Routing: The packets can be transfered between networks using IP address to determine the destination.
* Layer 2 Switch: 
* Layer 3 Switch: 
* Endpoint: An endpoint is an application or host that is capable of communicating (sendind and/or receiving messages) on a network. An endpoint is typically either a resource provider or a resource consumer.

## Limitations

* OpenDayLight
  * ODL doesn't provide general routing service to handle global routing or switching. So ECS relaies on an extra module to gather routing information.

* OpenFlow
  * Packet counter

* ALTO
  * Can't handle multi-path or load banlancing scenario.
  * Do not have fail-safe mechanism to get path.

## Architecture and Design description

### Working flow

The general method we use in ECS is:
1. Find the actual paths which packet vias;
2. Compute the routing cost by givend paths.

The fist step in our implementation is that ECS acquire the actual path between endpoints to the routing cost. We follow the algorithm shown as below to calculate paths. Notice that the paths may be multi-path for some reason such as load balancing.

```
IF found a routing service in ODL:
    use that service to get paths;
    RETURN the paths;
ELSE IF lookup flow table entries to compute paths successfully:
    RETURN the paths;
send a fake packet in the network to simulate the communication between endpoints;
IF lookup flow table entries to compute path successfully:
    RETURN founded path;
ELSE RETURN null;
```

_We didn't complete line (6) to (8) because of the ODL's limitations._

Then we compute the routing cost.

```
SWICH cost metric:
    CASE hopcounts: RETURN the length shortest path in gavin paths;
    CASE bandwidth: RETURN the path with minimal bandwidth;
    CASE routingcost:
        IF user privide routing cost function RETURN the function's result;
        ELSE RETURN our default function's result;
    DEFAULT:
        RETURN null;
```

### Overview of our ECS implementation

ECS has two module in current ALTO implementation, `alto-provider` module, which contains ECSImplementation, and `alto-network` module, which contains the supported codes of ECS. And this ECS depend on a modified L2 Switch in OpenDayLight, which will discussed in next section. 

### Dependencies

* A modified L2 Switch:
  We use a modified L2 Switch from ODL because two reason. First is that ODL didn't provide routing service or another switching module. So ODL's default selection is L2 Switch. Second is that the L2 Switch from ODL use Spain Tree Protocol (STP) as default switching protocol. It's not good enough because STP cause flooding. Each packet in the network managed by L2 Switch would fowrarded to all hosts so the network will be inefficiency. Since L2 Switch has legacy codes from Hydrogen release, which use shortest path algorithm to switching. We modified the L2 Switch to enable Dijkstra algorithm using the lagacy codes. And we also privide a routing service in L2 Switch to calculate the real path between endpoints.

### `alto-provider` Overview

TODO

### `alto-network` Overview

TODO

## Services APIs/User Interface

### Northbound APIs defniened by RFC7285

Please see details from [RFC7285]

### User defiened routing cost computation

TODO

### User defiened routing service

Now we define the routing service interface by java:

```java
public interface NetworkGraphService {
  public List<Link> getPath(NodeId sourceNodeId, NodeId destinationNodeId);
}
```

## TODO List

* A general routing service in ODL: OpenDayLight provides insufficient details of routing information. It even DO NOT have a general routing service. So current we manage to modify the original l2switch module to get actually routing between endpoints.
* Make ECS independenly: Now ECS depend on L2 Switch and OpenDayLight. It's not a good design because ECS may gather data from mutliple information sources. The ECS should only depend on the data not an module.
