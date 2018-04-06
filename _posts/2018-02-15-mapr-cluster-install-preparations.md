---
title: Preparation for Installing a MapR Cluster.
excerpt: "Identify Node Types, Prepare and Verify Cluster Hardware, Test Nodes and Plan the Service Layout"
tags: [MapR, Hadoop, Adminstration, Preparation]
categories: [ MapR ]
comments: true
published: true
share: true
---

# Prepare for Installation
## Intro
When you have finished with this section, you will be able to:
  * [Identify Node Types](#Identify Node Types)
  * [Prepare and Verify Cluster Hardware](#Prepare and Verify Cluster Hardware)
  * [Test Nodes](#Test Nodes)
  * [Plan the Service Layout](#Plan the Service Layout)

## Identify Node Types
### Node Types
#### Data Nodes:
![Datanodes]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_111_datanodes.png)

  * Store data
  * Run application jobs
  * Process table data

#### Control Nodes:
![Controlnodes]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_111_controlnodes.png)

  * Manage the cluster
  * Establish communication between cluster and client nodes

### Client Nodes:
![Clientnodes]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_111_clientnodes.png)

  * Submit jobs
  * Retrieve data

### Control-As-Data Nodes:
Smaller clusters may combine control and data nodes

![Control-As-Data Node]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_112_control-as-datanodes2.png)


## Prepare and Verify Cluster Hardware
### Prepare Cluster Hardware
  * Critical! Verify all nodes meet requirements
  * Failure to verify all nodes is a common cause of failure during installation
  * Visit [maprdocs.mapr.com/51/#AdvancedInstallation/PreparingEachNode.html](http://maprdocs.mapr.com/51/#AdvancedInstallation/PreparingEachNode.html) for the latest information

### Cluster requirements
  * Minimum requirements are just that: minimum
      * Not necessarily recommended for production
  * Optimum specifications vary depending on:
      * Size of the cluster
      * The node type
      * Types of jobs that will run
      * How busy the cluster will be
  * Refer to [maprdocs.mapr.com](http://maprdocs.mapr.com/) for details

#### Basic Requirements for All Nodes
  * 64-bit processor(s)
  * Supported OS
    * RHEL or CentOS 6.1 or later
    * SUSE 11 SP2 or later
    * Ubuntu 12.04 or later
    * Oracle Enterprise Linux 6.4 or later
  * Java installed (requires JDK, not just JRE)
    * Sun Java JDK 1.7 or 1.8
    * OpenJDK 1.7 or 1.8
  * Sync to an internal NTP server
  * Unique hostname for each node
    * `hostname –f` returns appropriate values
  * Resolvable with all other nodes with both forward and reverse DNS lookup
  * Identical credentials and UID/GID on each node, for any cluster user
    * MapR administrative user (`mapr` by default) is created during install if it does not exist
  * Common unified authentication on all nodes

![Unified Authentication]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_122_basicrequirementsallnodes.png)

#### Memory Requirements
  * Other recommendations:
    * Do not use numad (Non-Uniform Memory Access Daemon)
    * Set ```vm.overcommit_memory=0 in /etc/sysctl.conf```

#### Cluster Storage
  * Raw, unformatted disks
  * Best: three or more disks per data node
    * Same size and speed
  * Recommendations( due to unnecessary overhead on cluster performance):
    * Do not use RAID
    * Do not use LVM (Logical Volume Manager)

##### Cluster Storage Requirements
  * Determine size of data
  * Multiply by replication factor
    * Default is `3`
  * `25%` overhead
    * Log data, MapReduce data, temp files

![Determine data size]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_124_determinesizeofdata.png)

#### Local File System Requirements
##### Local Storage
  * Put some directories on their own partition
    * `/opt` (at least `128` GB)
    * `/tmp` (at least `10` GB)
    * `/opt/mapr/zkdata` (about `500` MB)
  * Swap space: `24` to `128` GB
    * `~110%` of physical memory

![Local File System]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_125_localfilesystemrequirements.png)

##### Boot Drive
  * Use LVM; its a good idea to use LVM on boot drive, but it should not be used on cluster storage
    *  Easily grow file systems later
  * Mirror the boot disk
​    *  Facilitate recovery in case of failure

#### Network Ports Used by MapR
  * Port `9443`:  UI installer
  * Port `8443`:  MapR Control System
  * Full list in online documentation

### Pre- and Post-Install Tools
MapR tools used for pre-install and post-install tests available [here](https://github.com/MapRPS/cluster-validation)

### Audit the Cluster
  * Ensure prerequisites are met
  * Identify component disparities:
    *  Disk size, cache, rpm
    *  Disk controller issues/bottlenecks
    *  Number of drives per controller

## Test Nodes
### Pre-Installation Testing
  * Start with subsystem components in good order
    *  One slow node can drag down performance
  * Establish performance reference values
  * Identify component disparities
  * End goal: get as much work through cluster as you can with minimal downtime

### Pre-Install Tests
#### stream
  * Tests memory and gives clues to CPU performance
  * Measures sustainable bandwidth, not peak performance

#### IOzone
  * Destructive read/write test
  * Run multiple times to verify repeatability
  * Look for bottlenecks

![IOzone test]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_132_preinstalltestsiozone.png)


#### rpctest
  * MB/sec network can handle
    *  Upper bound for the link
  * Test between pairs of nodes
    *  Point-to-point connections
  * Test multiple node pairs concurrently
    *  Look for same results as sequential test
    *  Indicates how well switches are working

## Plan the Service Layout
### Service Layout Considerations
  * MapR runs on clusters of one-two nodes
  * High Availability (HA) requires three or more, since ZooKeeper must be installed on an odd number of nodes
  * Rough size range:
    *  Small (`< 10 `nodes)
    *  Medium (`10-25` nodes)
    *  Large cluster (`25` up to `1000`s of nodes)
  * Not all services required
  * Layout will evolve over time
    *  Add/remove services
  * Plan initially, then monitor to see if service changes are needed
  * The role of each service
    *  How many of each are essential
    *  How many needed for High Availability (HA)
  * Resource requirements for services
  * The size of your cluster

![Service Layout]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_141_servicelayoutconsiderations.png)

### Overview of Services

![Overview of Services]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_142_overviewofservices.png)
To see the services configured for a node, view `/opt/mapr/roles`

#### Services: ZooKeeper
  * Coordinates other services
  * Provides:
    * Distributed and synchronized configuration
    * Storage and mediation of configuration information
    * Resolution of race conditions
  * Runs on an odd number of nodes
    * One, three, or five on separate nodes
    * A quorum (majority) must be up
  * Start before other services
  * Run on control nodes, or control-as-data nodes

#### Services: Warden
  * Coordinates cluster services
    * Light Java application
    * Runs on all nodes
  * A MapR-specific service
  * Job on each node is to:
    * Start, stop, restart services (except ZooKeeper)
    * Allocate memory to services

#### Services: CLDB
  * Tracks and dispenses container information
    * Replaces NameNode
  * Can be one to three
    * Two required for HA
  * Master CLDB accessed for writes
    * Other CLDBs on standby and accessed for reads
  * Automatically restarted on failure
    * No data or job loss
  * CLDB failover is automatic with Converged Enterprise Edition
    * Different procedure for Converged Community edition
  * Run on control nodes, or control-as-data nodes

#### Services: ResourceManager and NodeManager
##### ResourceManager
  * Manages cluster resources
  * Schedules applications for YARN
  * Two required for HA
    * Prefer three on large clusters
    * One active, others standby
  * Run on control nodes, or control-as-data nodes

##### NodeManager
  * Works with ResourceManager to manage YARN resource containers
  * One per data node and control-as-data node

#### Services: MapR File Server
  * MapR FileServer (MFS) manages disk storage
  * Run on all nodes that store data
  * Must run on any CLDB nodes
    * Even control nodes


#### More Services
  * NFS provides read/write Direct Access NFS™
    * Recommended for all nodes
  * HistoryServer archives MapReduce job metrics and metadata
    * One per cluster; typically on a control node
  * Webserver provides access to the MCS (MapR Control System)


### Running Services on the Same Node
####  CLDB and ZooKeeper
  * On small clusters, may need to run CLDB and ZooKeeper on the same node.
  * On medium clusters, assign to separate nodes.
  * On large clusters, put on separate, dedicated control nodes.

####  ResourceManager and ZooKeeper
  * Avoid running ResourceManager and ZooKeeper together
  * With more than 250 nodes, run ResourceManager on dedicated nodes

####  Service Guidelines for Large Clusters
  * Avoid running MySQL Server or webserver on a CLDB node

###  Sample Service Layouts
#### Small HA Cluster
![Overview of Small HA Cluster]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_1413_small-ha-cluster.png)

#### Large HA Cluster
![Overview of Large HA Cluster]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_1413_large-ha-cluster.png)
