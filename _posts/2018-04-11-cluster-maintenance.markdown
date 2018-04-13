---
layout: single
title: "Cluster Maintenance"
date: "2018-04-11 16:42:53 +0100"
excerpt: "Monitor and Manage Cluster, Disk and Node Maintenance ; Troubleshooting in MapR Converged Data Platform "
tags: [MapR, Hadoop, Administration]
categories: [ MapR ]
comments: true
published: true
share: true
toc : true
toc_label : "Maintenance"
toc_icon : "wrench"  # corresponding Font Awesome icon name (without fa prefix)
---

# Intro
This post teaches you how to monitor the cluster’s health, manage cluster resources, and perform ongoing maintenance and troubleshooting to keep the cluster in top working order.

#  Monitor and Manage the Cluster

## Monitor the Cluster
### Monitoring Tools
#### Tools
The Cluster can be monitored using MapR tools like MCS, CLI and external tools like Ganglia and Nagios.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1111_monitoringtools3.png)

#### [Nagios](https://www.nagios.org/)

* Nagios can monitor network and host resources
* Configuration script generator
* Conduit for alarms to go through standard escalation
* Integrates other sources of alarms aside from MapR builtin alarms.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1111_nagios2.png)

#### [Ganglia](http://ganglia.sourceforge.net)

* Monitors Clusters and grids
* Scalable, distributed system monitoring tool
* Unlike Nagios, Ganglia is not used to raise alarms but to report cluster health.
* Show cluster status in a given time period
* Install on CLDB nodes to gather metrics from CLDB like cpu, memory, active server nodes and volumes created on the node and overall cluster.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1111_ganglia2.png)


### Maintenance Tasks
1. Overview
* Regular maintenance
  * Replace disk(s)
  * Maintain nodes
  * Upgrade software
  * Clean out log files
* Design changes
  * Make cluster HA
  * Expand cluster
  * Balance services

2. Sample Schedule
Shown here is a possible Monitoring and Maintenance Schedule :
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1112_maintenancetasks_sample.png)


## Configure and Respond to Alarms

### MapR Alarms
#### About Alarms
* Alarms are raised to notify users of a problem
* When an alarm is raised alerts are posted to the MCS
  * Can also be sent via email
* Configure custom email addresses for specific alerts to accountable entities.

#### Alarm Classifications
Alarms can be classified into :
1. Cluster Alarms :
  * Cluster Space
  * Licensing Issues
  * CLDB problems.
2. Volume Alarms :
  * Specific to a Volume.
  * Snapshots and Mirroring failures
  * Under replicated and Unavailable data
  * Volume topology problem.
3. Node Alarms :
  * Specific to a Node
  * Disk failures
  * Time-skew problems
  * Failed Services

``` bash
$ maprcli alarm names
```
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1121_alarmclassifications.png)

#### View Alarms
Alarms can be viewed either in Dashboard or MapR CLI. To see all alarms:
``` bash
$ maprcli alarm list
```
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1121_viewalarm.png)

#### View Alarm Details
Vie alarm details in Dashboard and investigate the problems , restart services or nodes affected.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1121_viewalarmdetails.png)

### Configure Alerts
#### Overview
Before alarms are set configure SMTP and user email addresses.
1. Set up SMTP
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1122_configurealerts_smtp.png){: .align-center}

2. Configure email addresses of cluster users
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1122_configurealerts_email.png){: .align-center}

3. Set up custom alerts
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1122_configurealerts_alerts.png){: .align-center}

#### MCS and CLI

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1122_configurealertsmcsandcli2.png)
Standard alerts send email to the entity involved like Volume Quota Alarms to Volume Accountable Entity.
Additional address can be defined for alternative notifications to other target users.

#### Test Alerts
It's a good idea to test alarms because Email address and SMTP are not configured when alarms were defined.
* Raises the specified alarm
* Description appears in maprcli alarm list output and MCS
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1122_testalerts.png)

#### Verify Configuration
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1122_verifyconfiguration2.png)

### Respond to Alarms

#### Determine Urgency

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1123_determineurgency2.png){: .align-right}

* Alarms differ in urgency
  * Some resolve on their own
  * Some are early warnings
* Determine:
  * Underlying cause
  * What action is needed
  * How soon action should be taken
* Many alarms are self-explanatory
* Refer to [documentation](http://doc.mapr.com/display/MapR/Alarms+Reference) for details

####  Example: Normal Operation
_CLUSTER_ALARM_UPGRADE_IN_PROGRESS_
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1123_examplenormaloperation.png){: .align-right}
* Normal operation
* Alarm will clear when upgrade is done

####  Example: Schedule Action
_ALARM_ADVISORYQUOTA_EXCEEDED_
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1123_examplescheduleaction.png){: .align-right}
* Urgency depends on:
  * Rate of data growth
  * Differential between advisory and hard quotas
  * Which volume or AE

####  Example: Degraded Performance
_NODE_ALARM_SERVICE\_*\_DOWN_
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1123_exampledegradedperformance.png){: .align-right}
* May impact a single node, or entire cluster
* Criticality varies by service

####  Example: Urgent Situation
_NODE_ALARM_TIME_SKEW_
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1123_exampleurgentsituation.png){: .align-right}
* Alarm raised for time skew
  * Maximum differential: 20 seconds
  * Recommended differential: <500 ms
* System time critical for ZooKeeper functionality
  * Use NTP


## Balance Cluster Resources
As the cluster keeps growing and changing  it is important to balance location and replication of data. There are two balancers available :
* Disk balancer
  * Redistributes data
* Role balancer
  * Changes container replication role

Balancing starts 30 minutes after CLDB startup, by default
  * change with `cldb.balancer.startup.interval.sec`

### Disk and Role Balancers
#### Disk Balancer
* Moves containers between storage pools  
* Ensures disk usage on nodes is similar

Disk balancer is particularly useful when new nodes or rack is added to matured cluster.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1132_diskspacebalancer.gif)

#### Disk Balancer Details
There are two primary disk balancer settings :
1. Threshold defines a percentage of utilization at which a storage pool is eligible for rebalancing. Can be set between 10%-99%
2. Concurrent disk rebalancers define percentage of data can be actively rebalanced at a point of time. This prevents rebalancing from disrupting normal cluster operations. Can be set from 2%-30%

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1132_diskbalancerdetails.png)

#### Role Balancer
* Changes replication roles of containers to ensure all nodes have equal share of master containers.
* Distributes master containers among nodes
* Evenly distributing replication roles balances network bandwidth and also spreads load for writes across cluster.
* Write operation on MapRFS and Read on MapRDB are performed on master container.So role balancing also distributes table reads.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1132_rolebalancer.gif)

#### Role Balancer Details
There are two primary role balancer settings
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1132_rolebalancerdetails.png)


### Enabling Balancers Through the CLI
#### Balancing: CLI
* Turn on:
```bash
maprcli config save -values {"<parameter>" : "<value>"}
```
* Adjust:
```bash
maprcli config save -values { "cldb.balancer.disk.paused" : "0"}
maprcli config save -values { "cldb.balancer.role.paused" : "0"}
```
* Check values:
```bash
maprcli config load -json | grep balancer
```
### Balancer Parameters
List of some of the disk and role balancing parameters
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1133_balancerparameters.png)

## Manage Logs and Snapshots
### Manage Snapshots
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1141_managesnapshots.png){: .align-right}

* Scheduled snapshots expire automatically
* Manual snapshots must be deleted
  * Check regularly (at least monthly)
* Use schedules if possible

### Remove Snapshots
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1141_removesnapshots.png)
``` bash
maprcli volume snapshot list -columns snapshotid -filter [expirytime=="0"]
maprcli volume snapshot remove -snapshots <ID1, ID2, ID3 ...>
```

### Manage Log Files
1. Overview
* Some managed automatically
  * Job logs
  * Data audit logs (if enabled)
* Others must be cleaned out manually
  * `maprcli` audit logs
  * Cluster audit logs (if enabled)

2. Retention Times
* Retain longer if needed for troubleshooting
  * example: job logs with local logging deleted after 3 hours by default
* Shorten retention time if logs get too large

## Add and Remove Services
### Removing Services
The following is the procedure for removing a service from a node.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1151_removeroles.png)
### Adding services
Adding services is even easier than removing as follows :
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1151_addroles.png)


# Disk and Node Maintenance
## Replace a Failed Disk
### Disk Failure
Replacing failed disks is a normal part of cluster maintenance.
Factors impacting disk failure rate:
1. Number of Disks
2. Disk Reliability
3. Data Center Environment

### What Happens When a Disk Fails?
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1212_diskfails.gif)

### Disk Failure Information
Look at the Disk Failure Report and perform recommeded recovery methods.
1. Disk Failure Report
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1213_diskfailurereport.png)

2. Recovery Methods
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1213_recoverymethods.png)

3. Alarms Raised
* VOLUME_ALARM_DATA_UNDER_REPLICATED
  * data under available disk is replicated but doesn't meet minimum replication requirements.
  * automatically re-replicated from another copy and alarm cleared
* VOLUME_ALARM_DATA_UNAVAILABLE
  * If the only copy of data was on failed disk or all disks with that data failed.
  * run `/opt/mapr/server/fsck` to attempt to recover
  * Can bring storagepool online if `fsck` clears disk issues.
  * risk of data loss

**Caution** : Risk of data loss when running `fsck -r`
{: .notice--warning}

### Replace a Failed Disk: MCS
To replace a failed disk in MCS navigate as follows :
1. Locate the Failed Disk
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1215_locatethefaileddisk.png)

2. Remove the disk from MapR-FS and physically replace failed disk
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1215_removefrommapr-fs.png)

3. Add New/Repaired Disks
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1215_addnew-repaireddisks.png)

4. Remove the failed disk report when replacement is complete.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1215_faileddiskreport.png)

5. Verify New Storage Pool generated for new disks.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1215_verifynewstoragepool.png)

### Replace a Failed Disk: CLI
To replace a failed disk in CLI perform following commands :
1. Run `$ maprcli disk remove`
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1216_replaceafileddiskcli1.png)
2. Note drives removed; physically replace failed drive
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1216_replaceafileddiskcli2.png)
3. Remove failed disk report log
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1216_replaceafileddiskcli3.png)
4. Run `$ maprcli disk add` with list of disk being replaced.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1216_replaceafileddiskcli4.png)

## Add and Remove Disks
### Best Practices
* Add disks in groups to form a storage pool
  * If you add a single disk  a storage pool of single disk is created.
  * If you add multiple disks MapR will automatically generated storagepool(s) from them.
* Add disks with homogenous sizes and speeds
  * Optimize performance
  * Maximize capacity
* If you add heterogenous disk the performace and capacity is limited by slowest and lowest of the pool(randomly grouped).
* Add disks in groups by size ( or performance like SSD vs HDD); this ensures that only disks of the same size go into a storage pool

### Procedures: Adding and Removing Disks
#### Add Disks
After physically installing the new disks we can add them to MapRFS using MCS or CLI as follows :

![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1222_adddisks2.png)

#### Remove Disks
* Removing disk removes the entire storage pool(all the other disks in the pool too)
* Removed disks no longer contain usable data
* Make sure data exists elsewhere (storagepool replicated). GUI will warn if the data is not replicated elsewhere.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1222_removedisks2.png)

## Perform Node Maintenance
### Decommissioning vs. Maintenance

1. ​Decommissioning : Permanently remove from cluster
* Failed services
* Disks unrecoverable
* corrupted filesystems or OS
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1231_decommissioning.png)

2. Maintenance : Plan to perform an action but keep the node as part of the cluster
* Upgrade the OS
* Add or replace disk drives
* Diagnose problems
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1231_maintenance.png)

### Decommission a Node

#### Decommission a Node
1. Move node to the `/decommissioned` topology
  * Doesn't need to be named that but move the node to a topology outside of `/data`
  * Wait for replication to finish
  * Verify volume information removed
2. Stop Warden and ZooKeeper
3. Run `maprcli remove node`
4. Remove installed packages

#### Clean Up
1. Remove `/opt/mapr` directory
2. Remove MapR core files in `/opt/cores`
3. If removed node has CLDB or ZooKeeper, run `configure.sh` on all other nodes to maintain HA
  *  `configure.sh –C` if CLDB node changed
  *  `configure.sh –Z` if ZooKeeper node changed

### Perform Maintenance on a Node
#### Maintenance Overview
* No need to decommission
* Run `maprcli node maintenance`
* Replication starts after specified `–timeoutminutes`
* If multiple nodes, perform sequentially
* Don't offline more than half of the ZooKeepers
* Don't offline all CLDB or ResourceManager nodes

#### Perform Maintenance
1. Issue node maintenance command:
```bash
$  maprcli node maintenance -nodes <node> –timeoutminutes <minutes>
```
2. Stop Warden
3. Stop ZooKeeper (if installed on node)
4. Perform maintenance and reboot

## Add Nodes
### Expand a Cluster
When a new node is added to existing cluster you need to consider
* Node Topology
* How Management services are dispersed throughout the cluster?
* How data populates?

####  Node Topology
* Rack is a typical failure domain as some outages can take down entire rack.
* Rack can support 15+ nodes, you should consider spreading nodes evenly across three racks.
* By default three replicas of each data container, having three racks allow three sub-topology and containers can be replicated across three racks.
* Use fewer nodes per rack to distribute cluster nodes

#### Management Services
* If you have multiple management services on a node, it's recommended to move them to the newly added node
* Recommended to put NFS service on all the nodes.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1241_managementservices.gif)

#### Data Considerations
1. New data written randomly, weighted toward new nodes so gradually data is distributed.
2. Alternatively distribute data more quickly with disk balancer after a new node is added.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1241_dataconsiderations.gif)

### Add Nodes
Here is the overview of steps performed when adding a new node. Refer MapR [documentation](http://doc.mapr.com/display/MapR/Adding+Nodes+to+a+Cluster) for more details

#### Prepare
1. Set default topology to /newnodes
  *  Optional
2. Verify node specifications
3. Create MapR user
4. Add MapR repository
#### Install and Configure
5. Add MapR package key
6. Install MapR service packages
7. Run configure.sh
```bash
$  configure.sh -C <cldb nodes> -Z <zk nodes> -N <cluster name> -M7
```
8. Start Warden on new node
#### Finish Up
9. Format and add disks
10. Set node topology
11. Mount cluster
  * Optional
  * Only if NFS installed

# Troubleshooting

## Troubleshooting Different Problem Types
### How Do You Know There's a Problem ?
The three common give-aways are :
* Alarm raised
* Performance issues
* Jobs failing

### Resources
#### Resources
There are several resources to consult during troubleshooting.
* Log files
* [MapR documentation](maprdocs.mapr.com)
* [MapR Answers](community.mapr.com/community/answers)
* Apache forums and websites
* MapR Support (with Converged Enterprise Edition)

#### Log File Locations
This table lists some of common log types and their default locations. Some are found in local filesystem and while others in MapRFS
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1312_logfilelocations.png)

#### Log File Retention
Many logs expire automatically, so its possible for long running jobs logs are deleted before completion. You can change the retention times.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_maintain/adm203_1312_logfileretention.png)

### Troubleshooting Scenarios

#### Service Not Running
* Typically corresponds to a raised alarm
* Warden will attempt start or restart 3 times
* Each service has its own logs in `/opt/mapr/logs`
  * Don't forget to check Warden logs

#### Service Problems
1. Open two terminal windows
2. Window 1:
``` bash   
$ service mapr-warden stop
```
3. Window 2:
``` bash   
$ tail –f /opt/mapr/logs/warden.log
```
4. Window 1:   
``` bash
$ service mapr-warden start
```

####  Cluster Startup Issues
1. ZooKeepers up?   
``` bash
$ service mapr-zookeeper qstatus
```
2. Review `warden.log` and `cldb.log`
3. Get `maprcli node cldbmaster` to succeed
4. Troubleshoot individual services

#### Data Not Available
* Is affected volume node-specific?
  * `/var/mapr/local/<node name>`
  * Is node offline?
* Failed disks and nodes
* Permission problems

#### Jobs Not Running
* Start with the YARN job logs
  * command line
  * ResourceManager page (`<IP address>:19888`)
* Permissions problems
* Resource problems
* Recommended to run Yarn Log Aggregator to retain logs for long time centrally in MapRFS.

#### Degraded Performance
* Check `/opt/mapr/logs/mfs.log-3`
  * Failures communicating with the CLDB nodes or other servers?
* Check node resources
  * CPU, memory, swap space, disk space, network
* Isolate management services

## Use Support Utilities
### Tools

#### Core Files
* Copies of contents of memories when certain anamolies are detected.
* Located in `/opt/cores`
  * Name includes service involved
* An alarm is raised when a core file is created
* No immediate action required
  * Service will automatically restart
* Forward to MapR Support when possible

#### `fsck`
* Find/fix file system inconsistencies
  * `/opt/mapr/server/fsck`
* Repair storage pools after disk failure
* Verification or repair
  * Use `-r` for repair

**Caution**:
Using `fsck -r` can result in loss of data. Contact support first!
{: .notice--warning}

####  `gfsck`
* Find/fix cluster/volume/snapshot inconsistencies
  * `/opt/mapr/bin/gfsck`
* Scans and repairs a cluster, volume, or snapshot
* Typical process
  * Take storage pool offline
  * Run `fsck` on storage pool
  * Bring storage pool back online
  * Run `gfsck` on affected cluster/volumes/snapshots

####  `mapr-support-dump.sh`
* Collects information/logs from a single node
```bash
$  /opt/mapr/support/tools/mapr-support-dump.sh
```
* Output at `/opt/mapr/support/dump`
  * Creates a `.tar` file

####  `mapr-support-collect.sh`
* Collects information/logs from entire cluster
```bash
$  /opt/mapr/support/tools/mapr-support-collect.sh
```
* Output at `/opt/mapr/support/collect`
  * Uses SSH and SCP
  * Creates a `.tar` file
