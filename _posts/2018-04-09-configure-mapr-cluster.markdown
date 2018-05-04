---
layout: single
title: "Configure MapR Cluster"
date: "2018-04-09 13:10:19 +0100"
excerpt: "Users, Groups and System Settings; Configure Topology and Volumes; Job Logs and Scheduling"
tags: [MapR, Hadoop, Administration]
categories: [ MapR ]
comments: true
published: true
share: true
toc : true
toc_label : "Configure MapR"
toc_icon : "flag-checkered"
---

# Intro
>
This post covers

1. How to configure cluster users, topology, and volumes once the cluster has been installed.
2. It also covers the basics of job logs and scheduling jobs.
3. Also learn cluster administration with real-world system administrator concepts and practices, including
  * Planning, Installation and Configuration, Load Balancing
  * Tuning diagnosing deployment issues and performance
  * Setting up a Hadoop cluster with direct access NFS, snapshots
  * Monitoring cluster health, resolving hardware issues and troubleshooting job errors.

# Manage Users and Groups
## Cluster Users and Groups

* MapR uses native OS configuration
  * users/groups must exist at the OS level with same uid and gid on all nodes
  * use LDAP to simplify administration
* Assign permissions
  * cluster operations
  * volume operations
* Set disk use quotas
* There are two users who are granted permissions by default when MapR was installed
  * `root` - MapR software is installed as the root user
  *  `MapR Administrative User` - has full privileges to administer cluster and runs cluster services. specified during install (default is mapr)

## Cluster Permissions

1. Users and groups can be assigned permissions at three levels:  the cluster level, the volume level, and for cluster files and directories.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s07_cluster_permissions2.png)

2. Assign Cluster Permissions: MCS
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_412_assignclusterpermissionsmcs.png)

3. Assign Cluster Permissions: CLI
* `maprcli acl set` (overwrites existing permissions)
``` bash
maprcli acl set -type cluster -user mark:fc sharon:login
```
* `maprcli acl edit` (edits existing permissions)
``` bash
maprcli acl edit -type cluster -group mark:login
```

4. View Cluster Permissions: CLI
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s10_view_permissions.png)

## Volume Permissions

1. Users and groups can be assigned permissions at three levels:  the cluster level, the volume level, and for cluster files and directories.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s14_volume_permissions.png)

2. Assign Volume Permissions: MCS
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_414_assignvolumepermissionsmcs.png)

3. Assign Volume Permissions: CLI
* `maprcli acl set` (overwrites existing permissions)
``` bash
maprcli acl set -type volume -name vol11 -user mark:fc sharon:dump,m
```
* `maprcli acl edit` (edits existing permissions)
``` bash
maprcli acl edit -type volume -name vol11 -group mkt:dump,m
```

4. View Volume Permissions
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s17_view-permission.png)

## MapR-FS Permissions

1. MapR-FS Permissions is
* POSIX permissions model
* Set read, write, and execute permissions for user (owner), group, and other
* Change with:
  * `hadoop fs –chmod`
  * `hadoop fs –chown`
  * Linux commands, if cluster is mounted
2. Examples
  * Full permissions for user; read and execute for group/others:
  ```bash
  $ hadoop fs -chmod 555 myfile
  $ hadoop fs -chmod u=rwx , g=rx, o=rx  myfile
  $ hadoop fs -chmod u=rwx , go=rx myfile
  ```
  * Recursively set directory permissions:
  ```bash
  $ hadoop fs -chmod -R 600  /user/myvolume
  $ hadoop fs -chmod -R u=rx /user/myvolume
  ```


## User and Group Quotas

1. User and Group Quotas
* Users and groups can be assigned as Accountable Entities (AEs)
* Quotas can be assigned to AEs
* Disk use for an accountable entity:
  * Sum of space used by all volumes for which the user/group is the AE
* Volumes can also have quotas
2. Advisory and Hard Quotas
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s25_advisory-and-hard-quotas.png){: .align-right}
* **Hard quota**:  no additional data will be written to any volume belonging to the Accountable Entity
* **Advisory quota**:  an alarm is raised but data will continue to be written

3. Set Default User and Group Quotas
```bash
$ maprcli config save -values '{"mapr.quota.group.advisorydefault":"1T"}'
```
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s26_set_default_user.png)

4. Set Specific User and Group Quotas
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s27_set_specific_user.png)
An individual quota always takes precedence over default AE quota.

# Configure System Settings
##  System Settings

###  Email Addresses
Configure Email Address with one of the following Options :
* Suffix usernames with @domain as emails
* Use LDAP to retrieve email address for user.
![Email]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s31_sys-setting-email.png)

###  SMTP
Configure SMTP dialogue to send notification via Email as MapR Cluster
![SMTP]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l04_s32_sys-setting-smtp.png)

###  MapReduce Mode
{% include video id="157625075" provider="vimeo" %}


# Configure Topology

A clusters topology :
* Logically groups nodes
* Guides placement of data
* Based on business requirements and data access strategies

## Default Topology
Topology paths are defined much like FS directories but they are not mount-points or directories, but they are simply labels for topologies that illustrate the relationship between group of nodes.
For Example `/data/default-rack` is sub topology of `/data/`; so anything inside `/data/default-rack` topology is part of `/data/` topology.
* Default topology for Node: ​/data/default-rack
* Default topology for Volume: /data

As volumes are created they are assigned to `/data`; this means data is replicated to any node under `/data/` topology.

![Default Topology]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_s5-1-2_default-topology2.png)

## Topology and Replication
###  Default Topology and Replication
* MapR-FS attempts to replicate data:
  * to different nodes
  * in different sub-topologies​
* Without sub-topologies, data may be replicated to nodes in the same rack

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_513_anm1.gif)

### Rack Topology and Replication
* Define a rack topology so data will be replicated to different racks
* Data is automatically replicated to different sub-topologies

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_513_anm2.gif)


![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_s5-1-2_default-topology3.jpg)

>
Both `/data/foo` and `/data/foo/bar` are part of `/data/foo` topology  and hence data is written/replicated on nodes 1-16

## Common Topology Strategies
{% include video id="157626742" provider="vimeo" %}

MapR considers each topology rack ( or sub-topology) as a different rack.

## Topology Examples

Topology can be very simple…
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_s5-1-6_topology-simple.png)

 …or very complex
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_516_complex.png)

## Topology for Decommissioned Nodes
{% include video id="157641875" provider="vimeo" %}

> What happens when you move a node to the /decommissioned topology?

When you move a node to the /decommissioned topology, or anywhere outside of the topology all the volumes are in, no new data is written to that node.  Also, data on the node in /decommissioned will be moved back into the appropriate topology to maintain the correct number of copies.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm200_519_knowledgecheck_answer.gif)


## Best Practices
* Do not assign volumes to the `“/”` topology
* Use high-level topologies, such as `/data`, for volumes
* Use lower-level topologies only if required to constrain data placement
  * isolate departmental data
  * isolate based on performance requirements

## Configure Node Topology
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_s5-2-1_configure-node.png){: .align-right}
1. Have a plan! based on department or requirements.
2. Assign a topology to each node according to the plan
* through MCS
* via CLI
* with a text file

### Assign Node Topology through MCS
1. From the dashboard, navigate to Cluster > Nodes
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_522_assignnodetopology_clusternodes.png)

2. Select node(s) and click Change Topology
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_522_assignnodetopology_changetopology.png)

3. Select an existing topology, or enter a new one
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_s5-2-2_3.png)

4. The topology column and topology tree are updated
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_522_assignnodetopology_update.png)

### Create the `/decommissioned `Topology
To create a topology we need assign a node, So
* Assign the `/decommissioned` topology to a node
* Move the node back to its appropriate topology
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_523_anm1.gif)
* `/decommissioned` topology still exists and can be selected later.

### Assign/Change Node Topology Through CLI
1. Determine the node ID:
```bash
$ maprcli node list --columns id
```
2. Set the new topology:
```bash
$ maprcli node move -serverids <ID> -topology <path>
```

### Assign Complex Topologies With a Text File
On large clusters you can specify topology in a text file. Each line in the textfile specified a singlenode and the full topology path for that node.

In `/opt/mapr/conf/cldb.conf`, set:
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l05_s5-2-5_asiign-complex.png)



# Configure Volumes

## Describe Volumes and Volume Properties
### Recap: Storage Architecture
{% include video id="157643540" provider="vimeo" %}

### What is a MapR Volume?
Logical unit of data organization and management
* Comprised of containers
  * Name containers : Each volume has one named contained. It has metadata and first 64kb of data.
  * Data containers
* Volumes may span the cluster or set of nodes depending on the topology.

### Volume Properties
1. Name and Mount Path
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_613a_name-n-mount-path.png)

2. Permissions
Volume permissions can be granted to users or groups
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_613_permissions.png)

3. Topology
* Volume topology set to /data
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_613d_topology1.png)
* Volume topology set to /data/r1
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_613d_topology2.png)

### Volume Quotas
1. Hard and Advisory Quotas
* Hard quota
  * No more data written to volume
* Advisory quota
  * Alarm is generated

2. What Counts?
Only original, post-compression data counts against a quota.
Example:
* 10 GB source file: 10 GB
* compressed to 8 GB: 8 GB
* replicated 3 times: 24 GB

 Only **8GB** counts against the quota. Only space occupied by first copy/container counts

### Accountable Entities
* Entity (user or group) accountable for a volume’s usage
* Only one AE per volume
* AEs may also have quotas
* Every volume assigned to an AE counts against the AE’s quota i.e Size of AE's quota is preferred over Volume Quotas

### AEs and Quotas
1. Accountable Entities and Quotas
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_617a_accountable-entities-n-quotas.png)
2. Volumes and Accountable Entities
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_617b_volumes-n-accountable-entities.png)
3. Volumes and Accountable Entities
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_617c_volumes-n-accountable-entities.png)
4. Volumes and Accountable Entities
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_617d_volumes-n-accountable-entities.png)
5. Volumes, Accountable Entities, and Quotas
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_617e_volumes-accountable-entities-n-quotas.png)
This means combined size of volumes is limited to AE's Quota **40TB**

### Replication Factor
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_619_replication-factor.png){: .align-right}

* Replication factor is set at the volume level (default = 3)
* Applies to all containers in the volume
* Separate replication factors for name containers and data containers is also possible
* Replication factor
  * Desired number of copies
* Minimum replication factor (default 2)
  * If actual replication count falls below this, an alarm is raised and data is re-replicated immediately

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adn201_l06_619_gif_1.gif){: .align-right}
* Disk failure triggers immediate replication
* Node failure triggers replication after a configurable window
  * Default: 1 hour
  * Set with `maprcli config save`

### Replication Type
#### High Throughput

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adn201_l06_6110_gif_1.gif){: .align-right}
* Once set, cannot change for the volume
* High throughput (chain)
  * Default
  * Appropriate for most volumes

#### Low Latency

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adn201_l06_6110_gif_2.gif){: .align-right}
* Once set, cannot change for the volume
* Low latency (star)
  * Best with small files
  * Can impact network with large files


## Configure Volumes
### Typical Volume Layout
Some system volumes like `/opt/` and `/var/`. Apart from them we need to create volumes specific to projects and users.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_621_typical-volume-layout.png)

### Best Practices
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_622_best-practices.png){: .align-right}
Create plan and configure volumes before we start loading data into the cluster; some of the best practices involve :
* Create multiple volumes
  * Efficiently use snapshots and mirrors (by creating at volumes level)
  * Separate and manage resources
  * Establish ownership and accountability
  * Spread the load of access requests
* Create volumes based on business requirements
  * For users, departments, projects, etc.

### Create Volumes with MCS
Navigate to **MapR-FS > Volumes**, and click **New Volume**
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_623a_creat-volumes-w-mcs.png)

### Volume Setup
#### Volume Name
Volume name should be unique if you intend to used same or different mount path.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_623_volumesetupvolumename.png)
#### Mount Path
The parent directory should exist but the mount directory shouldn't.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_623_volumesetupmountpath.png)
#### Topology
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_623_volumesetuptopology.png)
#### Read/Write Status
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_623_volumesetupreadwritestatus.png)
If you want to prevent data in a volume being modified (after loading ; not at creation) Check `Read-only`
#### Accountable Entity
Select Accountable Entity by selecting an User/Group from drop-down list.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_623_volumesetupaccountableentity.png)

### Volume Settings
1. Permissions
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_625_volumesettingspermissions.png)

2. Quotas
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_625_volumesettingsquotas2.png)
* `<units>` is:
  * `M` (Megabytes)
  * `G` (Gigabytes)
  * `T` (Terabytes)


3. Replication
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_625_volumesettingsreplication.png)

4. Scheduling
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_625_volumesettingsscheduling.png)

5. Data Access
* Create Access Control Expressions to define access to volume data
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_625_maprcli_dataaccess.png)

### Create Volumes with CLI
1. Syntax
Create a volume and mount it with:
```bash
maprcli volume create -name <name> -mount 1 -path <path>
```
Type the command with no options to see syntax:
```bash
maprcli volume create
```
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_624a_create-volumes-w-cli.png)

2. Options
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_624a_createvolumesoptionscli.png)
For a full list: [Visit MapR Documentation](maprdocs.mapr.com/51/#ReferenceGuide/volume-create.html)

3. Set Permissions
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_624c_setpermissionscli.png)

### Modify a Volume
1. MCS : Click a volume name to modify its properties
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l06_625_modify-a-volume-mcs.png)

2. With maprcli:
```bash
maprcli volume modify -name <name> [options]
```

# Job Logs and Scheduling
## Configure Logging Options
### Job and Application Log Options

Logs created by application in the cluster are by default are stored in local files. When `Centralised logging` is selected logs are written to local volumes on MapR-FS , you can then run a command that will create directory with links to all relevant logs. `Yarn Log Aggregator` will aggregate resource container logs from local filesystem and store into the cluster.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_711_job-application-log-options.png)

#### About Log Options
* Best practice: store logs on the MapR-FS
  * Prevent failures due to lack of space on the local file system
  * Access logs from one location
* Option: Configure centralized logging and YARN log aggregation
  * If only centralized logging is configured, YARN jobs will use local logging

####  Local Logging
1. Default
* `userlogs` has one subdirectory per job ID
* `<job ID>` has one subdirectory for each container ID the job used
* `<container ID>` has `stderr`, `stdout`, and `syslog` files
2. Log Directory Location
* To set base directory for storing logs, set: `yarn.nodemanager.log-dirs` in `yarn-site.xml`

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_713_localloggingdefault2.png)

#### Centralized Logging
1. Overview  
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_714_icon.png){: .align-right}
* MapR-specific
* Only for MapReduce jobs
* Log files written to local volume on MapR-FS
  * default:  `/var/mapr/local//logs/yarn/userlogs`
  * replication factor = 1
  * I/O confined to node

2. Details
* During or after a job, use:
``` bash
maprcli job linklogs -jobid <job> -todir <target-dir>
```
* `<target-dir>`
  * contains symbolic links to all related log files instead of copying files
  * has sub-directories for hosts, mappers, and reducers

3. Output
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_714_output.png)

4. Configuration
* Configure in: `/opt/mapr/hadoop/<version>/etc/hadoop/yarn-site.xml`
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_714_configuration_code2.png)
* Restart ResourceManagers if enabled while apps are running

5. Log Retention Time
* Configure in: `/opt/mapr/hadoop/<version>/etc/hadoop/yarn-site.xml`
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_714_log-retentiontime_code2.png)
* Set retention time, in seconds.
  * defaults to 30 days

6. View Logs in MCS
* Navigate to **JobHistoryServer**, click on job ID
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_714_view-logs-mcs.png)

7. View Logs with CLI
``` bash
maprcli job linklogs -jobid <job> -todir <target-dir>
```
* After linking, `<target-dir>` contains symbolic links organized by hostname and resource container ID
* View logs with:
  * `hadoop fs –cat`
  * `hadoop fs -tail`

#### YARN Log Aggregation
#### Overview
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_716_icon.png){: .align-right}

* Aggregates and moves logs onto the MapR file system
* Accessed via:
  * YARN command-line tools
  * The HistoryServer UI
  * The file system (with NFS)
* Logs owned by user who runs the application

#### Enabling
* Configure in: `/opt/mapr/hadoop/<version>/etc/hadoop/yarn-site.xml`

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_716_configure_code2.png)

* Also uses `yarn.log-aggregation.retain-seconds`
  * same as for centralized logging

#### Log Locations
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_716_yarn-log-log-locations.png)

#### Configuration Steps
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_716_icon2.png){: .align-right}

* Set `yarn.log-aggregation-enable` property to `true` on all NodeManager nodes
* Set other properties if desired
* Restart the NodeManager services
* Restart JobHistoryServer
  * to view aggregated logs through JobHistoryServer page

#### View Logs Through MCS
* Navigate to **JobHistoryServer**, click on job ID
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_716_view-logs-through-mcs.png)

#### View Logs Using CLI
* Determine application ID:
```bash
$ yarn application -list appStates FINISHED
```
* View logs for the application:
```bash
$ yarn logs -applicationID <ID>
```

#### View Other Users' Logs  

`<yarn logs path>/<user>/<log dir suffix>`

* Default: yarn logs command searches current `user` path
* To see logs for jobs submitted by another user:
```bash
$ yarn logs -applicationID <ID> -appOwner <user>
```

## Configure the Fair Scheduler
### Job Schedulers
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_721_job-schedulers.png){: .align-right}

* Used to prioritize jobs
* Fair Scheduler is the MapR default
* Others available:
  * FIFO queue-based scheduler
  * Capacity Scheduler
* Label-based scheduling
  * add-on to capacity or fair scheduler

### Fair Scheduler

#### Overview
Dynamically allocates equal share of resources over time
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_722.gif)

#### Behavior
Can be configured in: `/opt/mapr/hadoop/<version>/etc/hadoop/yarn-site.xml`

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_722_fairschedulerbehavior.png)

#### Illustration
Here is an Illustration on Fair Scheduler
{% include video id="157642362" provider="vimeo" %}

### The Allocation File
#### Overview
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_724a_overview.png){: .align-right}

* Modify on the ResourceManager node
* Configure Fair Scheduler policy defaults
* Configure queues and their properties
* `/opt/mapr/hadoop/hadoop-<version>/etc/hadoop/fair-scheduler.xml`
* Allocation is created during installation and reloaded every 10 seconds; so changes can be made dynamically.They are in XML format

#### Defining Queues
Queues are defined in a queue block.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_724b_defining-queues.png)
This is means only those users `joe,marta and dorian` can use marketing queue.

#### Queue Weights
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_724c_queue-weights.png)
This means at full capacity `development` will use twice more resources than `marketing`.

#### Global Parameters
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_724d_global-parameters.png)
These can be overwritten for specific users...

#### Nested Queues
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_724e_nested-queues.png)

#### Resource Allocation
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm201_l07_724f_resource-allocation.png)
Memory allocation is not additive , so manager get 15Gb while other users in marketing queue get only 5Gb memory for their current jobs.
