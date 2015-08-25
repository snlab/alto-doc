# ECS Design and Implementation Document

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [ECS Design and Implementation Document](#ecs-design-and-implementation-document)
    - [Branch](#branch)
    - [Introduction](#introduction)
    - [Purpose](#purpose)
    - [References](#references)
    - [Glossary](#glossary)
    - [Features](#features)
    - [Use Cases](#use-cases)
    - [Architecture and Design description](#architecture-and-design-description)
    - [Services APIs/User Interface](#services-apisuser-interface)
    - [Usage Impact](#usage-impact)
    - [TODO List](#todo-list)
    - [Appendix](#appendix)

<!-- markdown-toc end -->

---

## Branch

Now we have two branches of our ECS implementation. One is totally compatible with OpenDayLight, which located in `master`. The other branch `feature/ecs` has the latest code. These codes can still work with original ODL but it works well with our modified L2Switch module beacuse OpenDayLight couldn't provide enough information for ECS.



## Introduction

ECS is the abbreviation of `Endpoint Cost Service`, which provides the routing cost between endpoints. ECS could provide the routing cost, by collecting raw networking data from OpenDayLight, whith different metrics such as hop counts, bandwidth and user defiened routing cost. This design document contains the information about our latest ECS implementation. 

## Purpose

State the purpose of the document; something like: this is functional specificationS of feature "..." which has Jira ID CS-xyzw

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

## Features

The features that provides now:

* Hop count:

    Count the number of hops between the given source and destination based on the routing path.

* Routing cost:

    Compute the routing cost between the given source and destination based on the routing path.

* Bandwidth:

    Compute the available bandwidth between the given source and destination based on the routing path.
	
The features that will provides in the future:

* 


## Use Cases

put the relevant use case/stories to explain how the feature is going to be used/work

## Architecture and Design description

* discussion of alternatives amongst design ideas, their resources/time tradeoffs and limitations. Explain why a certain design idea is chosen over others
* highlight architectural patterns being used (queues, async/sync, state machines, etc)
* talk about main algorithms used
* explain what components are being changed and what the dependent components are
* regarding database: talk about tables being added/modified
* performance implications: what are the improvements or risks introduced to capacity, response time, resources usage and other relevant KPIs
* preferably show class diagrams, sequence diagrams and state diagrams
* if possible, publish signatures of all methods classes and interfaces implement, and the explain the object information of different classes

## Services APIs/User Interface

* how to use the services

## Usage Impact

* Are there any entities being created that require usage reporting for billing purposes? 
* Does this change any existing entities for which usage is being tracked already?

## TODO List

* A general routing service in ODL: OpenDayLight provides insufficient details of routing information. It even DO NOT have a general routing service. So current we manage to modify the original l2switch module to get actually routing between endpoints.
* Make ECS independenly: Now ECS depend on L2 Switch and OpenDayLight. It's not a good design because ECS may gather data from mutliple information sources. The ECS should only depend on the data not an module.

## Appendix

Appendix A:

Appendix B:
