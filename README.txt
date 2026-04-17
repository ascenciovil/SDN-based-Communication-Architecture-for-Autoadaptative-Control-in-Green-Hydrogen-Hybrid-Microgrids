================================================================================
SDN-Based Communication Architecture for Auto-Adaptive Control
in Green Hydrogen Hybrid Microgrids
================================================================================

REPOSITORY OVERVIEW
--------------------------------------------------------------------------------

This repository contains the simulation environment and configuration files
associated with the research paper:

  "SDN-Based Communication Architecture for Auto-Adaptive Control
   in Green Hydrogen Hybrid Microgrids"

The provided materials allow full reproduction of the network performance
experiments described in the paper, using GNS3 as the network emulation
platform and OpenDaylight (ODL) as the Software-Defined Networking (SDN)
controller.

--------------------------------------------------------------------------------
REPOSITORY CONTENTS
--------------------------------------------------------------------------------

  ODL-QoS.gns3project  -  GNS3 project file containing the complete network
                           topology used in the experimental evaluation.

  RTBox1.zip           -  Configuration files for Real-Time Box 1 (RTBox1).

  RTBox2.zip           -  Configuration files for Real-Time Box 2 (RTBox2).

  README.txt           -  This file. Setup instructions and reproduction guide.

--------------------------------------------------------------------------------
PREREQUISITES
--------------------------------------------------------------------------------

Before proceeding, ensure the following software is installed on your system:

  - GNS3 (Network Emulation Platform)
  - OpenDaylight (Karaf distribution)
  - iperf3 (Network performance measurement tool)

--------------------------------------------------------------------------------
SETUP AND CONFIGURATION
--------------------------------------------------------------------------------

Import the ODL-QoS.gns3project file into GNS3 to load the complete testing
environment. Then follow the steps below.

STEP 1 - OpenDaylight Bridge Configuration
------------------------------------------

After importing and opening the project in GNS3, open a terminal on the
"opendaylight-odl-1" node and configure a bridge interface to connect all
OpenDaylight interfaces, enabling the SDN controller to communicate with the
OpenFlow switches:

    ip link add name br0 type bridge
    ip link set br0 up
    ip link set eth0 master br0
    ...
    ip link set eth10 master br0
    ip addr add 192.168.100.2/24 dev br0

This bridge aggregates all controller-facing interfaces under a single
logical interface (br0), which is required for proper OpenFlow communication.

STEP 2 - Install OpenDaylight Features
---------------------------------------

Launch the OpenDaylight Karaf console with:

    /opt/opendaylight/bin/karaf

Then install the required features in the following exact order. The
installation sequence is critical -- deviating from it may produce results
inconsistent with those reported in the paper.

    feature:install odl-restconf
    feature:install odl-mdsal-apidocs
    feature:install odl-openflowplugin-flow-services
    feature:install odl-dlux-core
    feature:install odl-dluxapps-nodes
    feature:install odl-dluxapps-topology
    feature:install odl-dluxapps-yangui
    feature:install odl-dluxapps-yangvisualizer
    feature:install odl-dluxapps-yangman

STEP 3 - Flow Rule Configuration
----------------------------------

Once OpenDaylight is running and the features are installed, configure the
flow rules in the virtual OpenFlow switches to enable communication between
the following agent pairs:

    Electrolyzer-Agent  <-->  Microgrid-Orchestrator
    Full-Cell-Agent     <-->  Load-Agent-3
    BESS-Agent          <-->  Load-Agent-5

Verify that bidirectional connectivity is established between each pair
before proceeding to the testing phase.

--------------------------------------------------------------------------------
EXPERIMENTAL PROCEDURE
--------------------------------------------------------------------------------

The network performance tests are structured in two layers of traffic:

  Background Traffic
  Establish iperf3 flows between the following pairs to simulate network load:

      Full-Cell-Agent  <-->  Load-Agent-3
      BESS-Agent       <-->  Load-Agent-5

  Measurement Traffic
  Run iperf3 and ping tests between:

      Electrolyzer-Agent  <-->  Microgrid-Orchestrator

  This is the critical communication path evaluated in the study.
  Performance metrics (throughput, latency, jitter) are collected from
  this pair.

  iperf3 command used for measurement:

      iperf3 -c <Microgrid-Orchestrator IP> -n 100M

--------------------------------------------------------------------------------
TEST SCENARIOS
--------------------------------------------------------------------------------

The experiments were conducted under four distinct network configurations,
corresponding to the scenarios described in the paper:

  Scenario 1 - Standard TCP/IP (Baseline)
  ----------------------------------------
  Network type  : Standard TCP/IP
  Switches      : Open vSwitch
  SDN controller: None
  QoS           : Not configured

  This scenario establishes the performance baseline for comparison.

  Scenario 2 - SDN Without QoS
  ------------------------------
  Network type  : SDN (OpenFlow)
  Controller    : OpenDaylight
  Switches      : OpenFlow-enabled virtual switches
  Flow rules    : Configured
  QoS           : Not configured

  Scenario 3 - Standard TCP/IP With QoS
  ----------------------------------------
  Network type  : Standard TCP/IP
  Switches      : Open vSwitch
  SDN controller: None
  QoS           : Enabled (meter rules)

  Bandwidth limited to 5 Mbit/s for background traffic:
      Full-Cell-Agent  <-->  Load-Agent-3
      BESS-Agent       <-->  Load-Agent-5

  This limitation prioritizes throughput for:
      Electrolyzer-Agent  <-->  Microgrid-Orchestrator

  High-priority flow rules applied for packets originating from
  the Electrolyzer Agent.

  Scenario 4 - SDN With QoS
  ---------------------------
  Network type  : SDN (OpenFlow)
  Controller    : OpenDaylight
  Switches      : OpenFlow-enabled virtual switches
  Flow rules    : Configured
  QoS           : Enabled (meter rules)

  Bandwidth limited to 5 Mbit/s for background traffic:
      Full-Cell-Agent  <-->  Load-Agent-3
      BESS-Agent       <-->  Load-Agent-5

  Traffic prioritization active for:
      Electrolyzer-Agent  <-->  Microgrid-Orchestrator

  High-priority flow rules applied for packets originating from
  the Electrolyzer Agent.

================================================================================
