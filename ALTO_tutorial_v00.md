# ALTO Tutorial

## Table of Contents

TODO

## Installing and Starting ALTO Server

### Build from source code

You can get ALTO server from the Github.

#### Preparations

Before install ALTO server you need the preparations below, including Java JDK, maven, Git and some configurations.

* [Development Environment Setup for ODL](https://wiki.opendaylight.org/view/GettingStarted:Development_Environment_Setup).

  **Important:** Don’t forget to edit your ~/.m2/settings.xml file otherwise you will not be able to download any ODL Java dependencies.

#### Building and Installing

* Enter your ALTO server project root directory.
* Run `mvn install` to build ALTO project.

#### Replacing L2 Switch

* You can get the modified l2switch code from [Github](https://github.com/cs512/l2switch).

  ``` bash
  git clone https://github.com/cs512/l2switch
  ```

* Move the l2switch folder into ~/.m2/repository/org/opendaylight/ and replace the original `l2switch` folder.

### Running ALTO Server

* Enter alto-karaf/target/assembly/ directory in your ALTO server project.
* Run command ./bin/karaf to start the karaf.
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

## Usage

### Managing and Querying Data in ALTO

You can manage and query the data in ALTO. See [Administering or Managing ALTO](https://wiki.opendaylight.org/view/ALTO:Lithium_User_Guide#Administering_or_Managing_ALTO) for details.

### Use Cases

Two use cases are showed below: querying the cost and server selection. See [ALTO Client](#alto-clinet) for details.

## Connecting the Network to the Controller

### Using Mininet for Emulation

You can use Mininet to emulate the real networks.

#### Installing Mininet

There are four ways to install Mininet.

* [Option 1: Mininet VM Installation (easy, recommended)](http://mininet.org/download/#option-1-mininet-vm-installation-easy-recommended)

* [Option 2: Native Installation from Source](http://mininet.org/download/#option-2-native-installation-from-source)

* [Option 3: Installation from Packages](http://mininet.org/download/#option-3-installation-from-packages)

* [Option 4. Upgrading an existing Mininet Installation](http://mininet.org/download/#option-4-upgrading-an-existing-mininet-installation)

#### Using Mininet to build the networks

```
$ sudo mn --controller remote,<controller_IP> --topo tree,3 --switch ovsk,protocols=OpenFlow13
```
where `controller_IP` is the IP address of the controller and the default port is 6633.

If you want to use self-defined port, you can run the following command.

```bash
$ sudo mn --controller=remote,ip=<controller_IP>,port=<controller_port>
```

##### Changing Topology Size and Type

The default topology is a single switch connected to two hosts. You could change this to a different topology with ```--topo```, and pass parameters for that topology’s creation.

For example, to create a topology with one switch and three hosts:

```$ sudo mn --topo single,3```

Another example, with a linear topology (where each switch has one host, and all switches connect in a line):

```$ sudo mn --topo linear,4```

##### Custom Topologies

Custom topologies can be easily defined as well by using a simple Python API. An example is provided in [custom/topo-2sw-2host.py](http://mininet.org/walkthrough/#custom-topologies).

When a custom mininet file is provided, it can add new topologies, switch types, and tests to the command-line. For example:

```
$ sudo mn --custom ~/mininet/custom/topo-2sw-2host.py --topo mytopo --test pingall
```

<span id="alto-clinet">
## ALTO Client
</span>

### Querying the Properties

If you simply want to query the properties between a pair of nodes. You can use `curl` to query the properties.

#### Sending ECS request

ALTO client send the ECS request to the ALTO server.

``` bash
curl -l -H "Content-type: application/alto-endpointcostparams+json" -X POST -d 'cat example_input.json' <controller_IP>:8080/controller/nb/v2/alto/endpointcost/lookup -v
```
where `controller_IP` is the IP address of the controller and  `example_input.json` contains the information of the request including cost type, sources and destinations.

**example_input.json:**

``` json
{"cost-type":{"cost-mode":"numerical","cost-metric":"bandwidth"},"endpoints":{"srcs":"ipv4:10.0.0.1","dsts":"ipv4:10.0.0.5"}}
```

*note*: There should be no space and newline in you JSON file.

**optional parameters:**

cost-mode: numerical/ordinal

* numerical: This mode indicates that it is safe to perform numerical operations on the returned costs. The values are floating-point numbers.
* ordinal: This mode indicates that the cost values in a cost map represent ranking, not actual costs. The values are non-negative integers, with a lower value indicating a higher preference.


cost-metric: hopcount/routingcost/bandwidth

* hopcout: get the number of the hops between src and dst.
* routingcost: get the routing cost between src and dst.
* bandwidth: get the available bandwidth between src and dst.


#### Getting the result

* ALTO client will get the response from the ALTO server with the corresponding results.

``` json
{"meta":{"cost-type":{"cost-mode":"numerical","cost-metric":"bandwidth"}},"endpoint-cost-map":{"ipv4:10.0.0.1":{"ipv4:10.0.0.5":10000000.0}}}
```
From the result you can see that the available bandwidth between two endpoints is 10GB.

#### Changing of the network states

Use iperf to generate the traffic between two hosts in mininet.

TODO

### Server Selection

TODO

## Appendix

### Appendix A: Common Problem List

N/A

*Note:*

If you meet any problem while deploying and using, you can email ***wangjunzhuo200@gmail.com***, ***jingxuan.n.zhang@gmail.com*** or ***92yichenqian@tongji.edu.cn*** for help.
