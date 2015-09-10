# User Guide for Endpoint Cost Service(ECS) 
----

## Introduction
ECS is the abbreviation of **Endpoint Cost Service**, which provides the routing cost between endpoints. ECS accepts the pair of endpoints and returns the routing cost of the pair. It provides the routing cost by collecting raw networking data from OpenDayLight with different metrics such as hop counts, bandwidth and user-defined routing cost. This design document contains the information about our latest ECS implementation.

## Features
The features that ECS provides are showed below:

* Hop count:

	Count the number of hops between the given source and destination based on the routing path.
	
* Routing cost: 

	Compute the routing cost between the given source and destination based on the routing path.
	
* Available bandwidth: 

	Compute the available bandwidth between the given source and destination based on the routing path.

## Getting Started

### Install ALTO Project

#### Prerequisites
* [Development Environment Setup for ODL](https://wiki.opendaylight.org/view/GettingStarted:Development_Environment_Setup).
	* **Important:** Don’t forget to edit your ~/.m2/settings.xml file otherwise you will not be able to download any ODL Java dependencies.

#### Get ALTO project
TODO

#### Build and Install ALTO
* Enter your ALTO project root directory.
* Checkout to feature/ecs branch and run `mvn install` to build ALTO project.

### Replace L2 Switch
* Get modified l2switch code.(TODO)
* Put the l2switch folder into ~/.m2/repository/org/opendaylight/ and replace the original l2switch folder.

## Using Endpoint Cost Service in ALTO

### Run ALTO in Karaf
* Enter into alto-karaf/target/assembly/ directory.* Run command ./bin/karaf to start the karaf.	* It will take some time to install features, initialize services and download dependencies. Be patient and check ./data/log/karaf.log file to track the latest status.

### Input 


### Hop count

### Routing cost

### Available bandwidth




## Appendix
Appendix A:

Appendix B:
