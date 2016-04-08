# Steth

A network inspection tool for OpenStack.


  * License: Apache License, Version 2.0
  * Source: https://git.openstack.org/cgit/openstack/steth
  * Bugs: https://bugs.launchpad.net/steth
  * Wiki: https://wiki.openstack.org/wiki/Steth


## Description

Steth is an inspection tool that can aid in pinpointing issues before deployment
and during operation of an OpenStack environment.

It is modelled as agent(s)/client in which a controller interacts with agents
deployed in your environment.


## Background

OpenStack networking can be deloyed as different architectures, such as ML2 with
OVS(legacy and DVR), Linux bridge, OVN, Dragonflow and so forth. However, they
all need enviromental prerequisites. For instance, VLAN needs to be configured
as we expect; bandwidth should meet our requirements; connection between nodes
should be active, etc.

Besides, with some well-deployed architectures, troubleshooting for VM
networking is difficult. For instance, why VM cannot get an IP address; or why
it cannot connect to Internet, etc. Steth integrates useful scripts and third
party tools(like iperf, tcpdump, etc.) to help operators keep tracking on VM
networking.


## Mission

Steth is an introspection tool for OpenStack networking. Only proved to be
working in ML2 with OVS for now.


## Multi-node Architecture

```
                                                                   note that steth does not save
                                                                   any state, it acts as a rpc
                                                                   client which makes requests to steth
                                    +--------------------------+   agent(s) and analyses the result.
                                    |                          |
                                    |   +------CLI---------+   |
                                    |   |                  |   |
             +--------------------------+     steth        +--------------------------+
             |                      |   |                  |   |                      |
             |                      |   +--------+---------+   |                      |
             |                      |            |             |                      |
             |                      +--------------------------+                      |
             |                                   |                                    |
             v                                   v                                    v
+-------+port:9698---------+        +-------+port:9698---------+         +-------+port:9698---------+
|            ^             |        |            ^             |         |            ^             |
|            |             |        |            |             |         |            |             |
| +----------+-------+     |        | +----------+-------+     |         | +----------+-------+     |
| |                  |     |        | |                  |     |         | |                  |     |
| |   steth-agent    |     |        | |   steth-agent    |     |         | |   steth-agent    |     |
| |                  |     |        | |                  |     |         | |                  |     |
| +-----------+------+     |        | +-----------+------+     |         | +-----------+------+     |
|             |            |        |             |            |         |             |            |
|             |            |        |             |            |         |             |            |
|  +----------v----------+ |        |  +----------v----------+ |         |  +----------v----------+ |
|  | run command like:   | |        |  | run command like:   | |         |  | run command like:   | |
|  | ping, iperf, tcpdump| |        |  | ping, iperf, tcpdump| |         |  | ping, iperf, tcpdump| |
|  | or use scapy to send| |        |  | or use scapy to send| |         |  | or use scapy to send| |
|  | packet              | |        |  | packet              | |         |  | packet              | |
|  +---------------------+ |        |  +---------------------+ |         |  +---------------------+ |
|                          |        |                          |         |                          |
|                          |        |                          |         |                          |
+--------------------------+        +--------------------------+         +--------------------------+
```

In a scenario using multiple nodes, Steth is a stateless CLI and controller.
It knows each steth agent and will read config files, interact with OpenStack,
and sending instructions to agents when needed. 

Steth Agent is introduced to manage processes or run commands. It should be
installed in each compute and network node, and their IPs should be specified
in the config file of steth controller.


## Steth Agent

Listening on 0.0.0.0:9698 and waiting for the rpc request.

Note: for get_interface() agent API, we use ifconfig to get full information.
However, the output of ifconfig varies from a Linux distribution to another.
The API has only been tested on CentOS 6.5 and 7.0. Any other distribution has
not been tested. If it works, please let us know.
