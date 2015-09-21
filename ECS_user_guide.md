# User Guide for Endpoint Cost Service(ECS) 

----

## Introduction

ECS is the abbreviation of **Endpoint Cost Service**, which provides the routing cost between endpoints. ECS accepts the pair of endpoints and returns the routing cost of the pair. It provides the routing cost by collecting raw networking data from OpenDayLight with different metrics such as hop counts, bandwidth and user-defined routing cost. This document contains the information about how to use ECS in ALTO.

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

#### Get ALTO server

* Get the ALTO server project. (TODO)

#### Build and Install ALTO server

* Enter your ALTO project root directory.

* Checkout to feature/ecs branch and run `mvn install` to build ALTO project.

### Replace L2 Switch

* Get modified l2switch code from [GitHub](https://github.com/cs512/l2switch.git).

* Put the l2switch folder into ~/.m2/repository/org/opendaylight/ and replace the original l2switch folder.

## Using Endpoint Cost Service in ALTO

### ALTO Server

#### Run ALTO Server in Karaf

* Enter into alto-karaf/target/assembly/ directory.

* Run command `./bin/karaf` to start the karaf.

* It will take some time to install features, initialize services and download dependencies. Be patient and check ./data/log/karaf.log file to track the latest status.

``` bash
$ cd alto-karaf/target/assembly
$ ./bin/karaf

    ________                       ________                .__  .__       .__     __
    \_____  \ ______   ____   ____ \______ \ _____  ___.__.|  | |__| ____ |  |___/  |_
     /   |   \\____ \_/ __ \ /    \ |    |  \\__  \<   |  ||  | |  |/ ___\|  |  \   __\
    /    |    \  |_> >  ___/|   |  \|    `   \/ __ \\___  ||  |_|  / /_/  >   Y  \  |
    \_______  /   __/ \___  >___|  /_______  (____  / ____||____/__\___  /|___|  /__|
            \/|__|        \/     \/        \/     \/\/            /_____/      \/
 
 
 Hit '<tab>' for a list of available commands
 and '[cmd] --help' for help on a specific command.
 Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown OpenDaylight.

 opendaylight-user@root>
```

#### Connect to the alto server
Connect the network to the alto server.

* For emulation: you can use mininet to emulate a network environment.

```
sudo mn --controller remote,192.168.0.24 --topo tree,3 --switch ovsk,protocols=OpenFlow13
```

### ALTO Client

#### Send ECS request

ALTO client send the ECS request to the ALTO server.

```
curl -l -H "Content-type: application/alto-endpointcostparams+json" -X POST -d `cat XXX.json` 192.168.1.24:8080/controller/nb/v2/alto/endpointcost/lookup -v
```
where XXX.json contains the information of the request including cost type, sources and destinations.

**XXX.json:**

```
object {
	CostType	cost-type;
	[JSONString	constraints<0..*>;]
	EndpointFilter	endpoints;
} ReqEndpointCostMap;

object {
	[TypedEndpointAddr srcs<0..*>;]
	[TypedEndpointAddr dsts<0..*>;]
} EndpointFilter;
```

e.g. :

```
{
    "cost-type": {"cost-mode" : "numerical",
                  "cost-metric" : "routingcost"},
    "endpoints" : {
      "srcs": [ "ipv4:10.0.0.1" ],
      "dsts": [
        "ipv4:10.0.0.2",
        "ipv4:10.0.0.3",
        "ipv4:10.0.0.4"
      ]
    }
}
```

**optional parameters:**

cost-mode: numerical/ordinal

* numerical: This mode indicates that it is safe to perform numerical operations on the returned costs. The values are floating-point numbers.
* ordinal: This mode indicates that the cost values in a cost map represent ranking, not actual costs. The values are non-negative integers, with a lower value indicating a higher preference.

 
cost-metric: hopcount/routingcost/bandwidth

* hopcout: get the number of the hops between src and dst.
* routingcost: get the routing cost between src and dst.
* bandwidth: get the available bandwidth between src and dst.


#### Get the result

* ALTO client will get the response from the ALTO server with the corresponding results.

```
{"meta":{"cost-type":{"cost-mode":"numerical","cost-metric":"routingcost"}},"endpoint-cost-map":{"ipv4:10.0.0.1":{"ipv4:10.0.0.4":200.0,"ipv4:10.0.0.3":150.0,"ipv4:10.0.0.2":100.0}}}
```

### Note:

If you meet any problem while deploying and using, you can email ***wangjunzhuo200@gmail.com***, ***jingxuan.n.zhang@gmail.com*** or ***92yichenqian@tongji.edu.cn*** for help.

## Appendix

### Appendix A: Common Problem List

N/A

