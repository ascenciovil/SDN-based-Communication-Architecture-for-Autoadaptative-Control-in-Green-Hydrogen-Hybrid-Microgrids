Instructions of Use
Repository Contents

This repository contains the following files:

ODL-QoS.gns3project – GNS3 project file containing the network topology used for testing.
README.txt – Instructions for configuring and running the tests.

The ODL-QoS.gns3project file is an importable GNS3 project where the testing environment was created. To perform your own tests, import this project into GNS3 and follow the configuration steps below.

1. OpenDaylight Bridge Configuration

After importing and opening the project in GNS3, open a terminal on the opendaylight-odl-1 PC and configure a bridge interface using the following commands:

ip link add name br0 type bridge
ip link set br0 up
ip link set eth0 master br0
...
ip link set eth10 master br0
ip addr add 192.168.100.2/24 dev br0

This bridge connects all OpenDaylight interfaces so the controller can communicate with the OpenFlow switches.

2. Install OpenDaylight Features

Open the OpenDaylight console by running:

/opt/opendaylight/bin/karaf

Then install the following features in this exact order:

odl-restconf
odl-mdasl-apidocs
odl-openflowplugin-flow-services
odl-dlux-core
odl-dluxapps-nodes
odl-dluxapps-topology
odl-dluxapps-yangui
odl-dluxapps-yangvisualizer
odl-dluxapps-yangman

Installing the features in this order is important, as changing the order may produce different results from those presented in the testing.

3. Flow Configuration

After installing the OpenDaylight features, configure the flow rules in the virtual switches to allow communication between the following devices:

Electrolyzer-Agent ↔ Microgrid-Orchestrator
Full-Cell-Agent ↔ Load-Agent-3
BESS-Agent ↔ Load-Agent-5

Make sure communication between these pairs is working before proceeding with testing.


4. Testing Procedure

Once connectivity is established, begin the network performance tests.

Step 1 – Background Traffic

Run iperf3 tests between:

Full-Cell-Agent ↔ Load-Agent-3
BESS-Agent ↔ Load-Agent-5

Step 2 – Measurement Traffic

Run iperf3 and ping between:

Electrolyzer-Agent ↔ Microgrid-Orchestrator

The performance measurements used in the study were taken from this connection.

5. Test Scenarios

The experiments were performed under three different network configurations:

1. Standard TCP/IP Network

A duplicate of the same topology was used, but with:

Standard switches
No SDN controller
No QoS configuration

This scenario serves as the baseline network.

2. SDN Network Without QoS

The topology was used with:

OpenDaylight controller
OpenFlow switches
Flow rules configured
No QoS configuration

3. SDN Network With QoS

Same configuration as above, but with QoS applied using meter rules to limit bandwidth to 5 Mbit/s for the following traffic:

Full-Cell-Agent ↔ Load-Agent-3
BESS-Agent ↔ Load-Agent-5

This bandwidth limitation allows prioritization of traffic between:

Electrolyzer-Agent ↔ Microgrid-Orchestrator


