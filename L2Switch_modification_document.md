# L2Switch modifation document

## Introduction

Since L2 Switch is the default routing service in OpenDayLight. And OpenDayLight didn't provides any other module to handle routing. But ALTO needs the routing information from the network. The dummy way to get path of two endpoints is looking up FRM in ODL then generate the path by flow table entry. A better way is that OpenDayLight can provides a routng service to give us a unified view of network. So we modified the L2 Switch in OpenDayLight to privide layer 2 routing service. Then our ECS implementation could query the path between endpoints from OpenDayLight.

## Difference from original L2Switch

### The issues that original L2Switch has

- Original flow table writer doesn't work.
- Using STP as default routing method caused low performance.
- Didn't provide a routing query service for two hosts.

### The modified parts

#### `l2switch-main`

- `FlowWriterServiceImpl`:
change the addFlows method to add multiply flow table into switches.

- `ReactiveFlowWriter`:
add writeFlowsDijkstra method to connect two hosts by shortest path.

- `inventoryReader`:
add getNodeConnectorByMac method.

#### `l2switch-loopremover`

- `NetworkGraphService`:
Deannotated getPath method.

- `NetworkGraphImpl`:
Implemented the getPath method. getPath will return the shortest path between two nodeId.

#### Version of L2Switch

Because the ALTO now depends on a modified L2Switch. We change the original L2Switch's version from `0.2.1-SNAPSHOT` to `0.2.1-ALTO-SNAPSHOT`.
