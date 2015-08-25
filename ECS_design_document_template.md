#ECS Design and Implementation Document
----

##Branch
Now we have two branches of our ECS implementation. One is totally compatible with OpenDayLight, which located in `master`. The other branch `feature/ecs` has the latest code. These codes can still work in original ODL but it works well on our modified L2Switch module beacuse OpenDayLight couldn't provide enough information for ECS.

##Introduction
ECS is the abbreviation of `Endpoint Cost Service`, which provides the routing cost between endpoints. This design document contains the design and implementation information of our latest ECS implementation. 

##Purpose
State the purpose of the document; something like: this is functional specificationS of feature "..." which has Jira ID CS-xyzw

##References
* [RFC7285](https://tools.ietf.org/html/rfc7285): Application-Layer Traffic Optimization (ALTO) Protocol
* [L2 Switch](https://wiki.opendaylight.org/view/L2_Switch:Main): Dependency of ECS 


##Glossary
* Switch:
* Switching:
* Route:
* Routing:
* Layer 2 Switch:
* Layer 3 Switch:
* Endpint:
* 

##Feature Specifications
* put a summary or a brief description of the feature in question 
* list what is deliberately not supported or what the feature will not offer - to clear any prospective ambiguities
* list all open items or unresolved issues the developer is unable to decide about without further discussion
* quality risks (test guidelines)
	* functional
	* non functional: performance, scalability, stability, overload scenarios, etc
	* corner cases and boundary conditions
	* negative usage scenarios
* specify supportability characteristics:
	* what new logging (or at least the important one) is introduced
	* how to debug and troubleshoot
	* what are the audit events 
	* list JMX interfaces
	* graceful failure and recovery schoolenarios
	* possible fallback or work around route if feature does not work as expected, if 
those workarounds do exist ofcourse.
	* if feature depends other run-time environment related requirements, provide sanity check list for support people to run
* explain configuration characteristics:
	* configuration parameters or files introduced/changed
	* branding parameters or files introduced/changed
	* highlight parameters for performance tweaking
	* highlight how installation/upgrade scenarios change
* deployment requirements (fresh install vs. upgrade) if any
* system requirements: memory, CPU, desk space, etc
* interoperability and compatibility requirements:
	* OS
	* xenserver, hypervisors
	* storage, networks, other
* list localization and internationalization specifications 
* explain the impact and possible upgrade/migration solution introduced by the feature 
* explain performance & scalability implications when feature is used from small scale to large scale
* explain security specifications
	* list your evaluation of possible security attacks against the feature and the answers in your design* *
* explain marketing specifications
explain levels or types of users communities of this feature (e.g. admin, user, etc)

##Structure

##Use Cases
put the relevant use case/stories to explain how the feature is going to be used/work

##Architecture and Design description
* discussion of alternatives amongst design ideas, their resources/time tradeoffs and limitations. Explain why a certain design idea is chosen over others
* highlight architectural patterns being used (queues, async/sync, state machines, etc)
* talk about main algorithms used
* explain what components are being changed and what the dependent components are
* regarding database: talk about tables being added/modified
* performance implications: what are the improvements or risks introduced to capacity, response time, resources usage and other relevant KPIs
* preferably show class diagrams, sequence diagrams and state diagrams
* if possible, publish signatures of all methods classes and interfaces implement, and the explain the object information of different classes

##Services APIs/User Interface
* how to use the services

##Usage Impact
* Are there any entities being created that require usage reporting for billing purposes? 
* Does this change any existing entities for which usage is being tracked already?

##TODO List
* A general routing service in ODL: OpenDayLight provides insufficient details of routing information. It even DO NOT have a general routing service. So current we manage to modify the original l2switch module to get actually routing between endpoints.
* Make ECS independenly: Now ECS depend on L2 Switch and OpenDayLight. It's not a good design because ECS may gather data from mutliple information sources. The ECS should only depend on the data not an module.

##Appendix
Appendix A:

Appendix B:
