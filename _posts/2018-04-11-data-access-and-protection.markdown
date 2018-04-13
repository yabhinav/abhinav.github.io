---
layout: single
title: "Data Access and Protection"
date: "2018-04-11 10:55:14 +0100"
excerpt: "Access Cluster Data, Snapshots and Mirrors in MapR Converged Data Platform "
tags: [MapR, Hadoop, Administration]
categories: [ MapR ]
comments: true
published: true
share: true
toc : true
toc_label : "Access and Secure"
toc_icon : "lock"  # corresponding Font Awesome icon name (without fa prefix)
---


# Access Cluster Data

## Access Data
### Access Data with HDFS

{% include video id="163743203" provider="vimeo" %}

### Access Data with MapR-FS and NFS
#### Features
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_821_features.png){: .align-right}

* Files in MapR-FS are fully read-write
  * Read files as they are written
  * Overwrite files
  * Modify files in place
* Direct-access NFS
  * Easily access data with Linux commands

#### Direct-Access NFS
* Read Performance
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_821_direct-access-nfs.png)

####  Mounting the Cluster
* Mount your cluster file system locally
  * `/mapr/my.cluster.com/` by default
* Use Hadoop jobs or standard Linux commands
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_821_mounting-the-cluster.png)

#### Handling Data
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_812_handling-data.png)


## Set up Client Access
## Types of Client Access
There are several ways to setup client access with MapR using :
1. Direct Access NFS™
2. MapR Client
3. MapR POSIX Client

### Direct Access NFS
1. Overview
* Mount the cluster via NFS
* Read and write cluster data directly
* NFS mounting models:
  * Gateway
  * Collocation
  * Self-mounting
2. Gateway
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_822_directaccessnfs_gateway.png)

3. Collocation
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_822_directaccessnfs_collocation.png)

4. Self-Mounting
* Run Direct Access NFS on cluster node and mount via localhost
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_822_directaccessnfs_selfmounting.png)

5. Mounting Procedure
* From the client machine:
``` bash
$ mkdir /mapr
$ mount -o tcp,ver=3,nolock  <MapR NFS_node>:/mapr /mapr
$ ls /mapr/<my cluster name>/<path_from_mfs_root>
```
* With local NFS server:
``` bash
$ mkdir /mapr
$ mount -o tcp,ver=3,nolock  localhost:/mapr /mapr
$ ls /mapr/<my cluster name>/<path_from_mfs_root>
```

When mounting an local NFS server on a cloud based cluster use `localhost` or loopback address `127.0.0.1`.
Don't use the nodes ipaddress as that actually routes the network out and back in.

### MapR Clients
#### MapR Client
MapR client is available for non-cluster nodes.Instructions differ based on OS.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_823_maprclients.png)

#### MapR POSIX Client
Along with regular access , Direct posix access is supported only in linux.Compared to native NFS posix
* Data is compressed on client side before sending to cluster.
* Uses a secure direct connection between client and cluster.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_823_maprposixclient.png)

###  MapR POSIX Client Types
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_823_maprposixclienttypes2.png){: .align-right}

* Starting MapR5.1 FUSE-based posix client are released instead of NFS loopback.
* Basic
  * Up to 1 GB/sec
  * 10 free
* Platinum
  * Up to 5 GB/sec (hyperthreading disabled)
  * Paid

## Configure Virtual IP Addresses
### What is a VIP?
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_831_vippool.png){: .align-right}

* A virtual IP address
* A "pool" of static IP addresses
* If the connection to the static IP address goes down, the VIP switches to another address in the pool

### Static IP versus Virtual IP
#### Static IP

A client connects directly to the IP of the node.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_832_static_ip.gif)

#### VIP

Alternatively a client can connect to a virtualIPaddress which has access to a pool of nodes.
When the node1 goes down the client will automatically connected to another node.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_832_vip.gif)

### Making NFS Highly Available

{% include video id="163767849" provider="vimeo" %}

### Configure VIPs

1. Configure VIPs through the MCS or through the CLI: `$ maprcli virtualip add`
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_835_configurevips1.png)

2. To create a VIP pool with desired subnet of network interface
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_835_configurevips2.png)

3. Add or remove a node to VIP pool
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_835_configurevips3.png)

### Review VIPs
1. Review or Modify VIP Pools
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_836_reviewvips_review.png)

2. View VIP Assignments
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_836_reviewvips_viewvipassignments.png)

3. ifconfig for VIPs (we can see virtual cards created)
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_836_review-vips.gif)

### Summary
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_831_vippool.png)

* Each VIP will route connections to a single node
  * Client connections will remain on the same node until failover
* VIPs are not load balancing
  * Set up round-robin DNS on your DNS server
  * Or use third-party load balancers instead
* You can select which NICs are assigned to each VIP
  * If not all NICs are in the same subnet
  * If you wish to restrict VIPs assignments to specific NICs


## Control Access to the Cluster

> Who Can Do What?

Depends on permissions granted:
* At the cluster level
  * MapR cluster operation permissions
* At the volume level
  * MapR volume operation permissions
  * Root directory permissions (POSIX)
  * Access Control Expressions (ACEs) to access volume data
* At the file/directory level
  * UNIX mode bits (POSIX)
  * MapR ACEs

### Access Control Expressions (ACEs)
{% include video id="165036308" provider="vimeo" %}

### Working with Whole Volume ACEs
{% include video id="165107168" provider="vimeo" %}


# Snapshots

## Understand Snapshots
### Overview
#### What is a Snapshot?
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_911_what_is_a_snapshot.png){: .align-right}

* A view of a source volume at a specific point in time
* Use to:
  * Recover from user errors, data corruption, or program errors
  * Create static data sets for queries or auditing
  * better than replication or mirroring.

#### Characteristics of a Snapshot
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_911_characteristics_snapshot.png){: .align-right}

* Read-only point-in-time image of a volume
* Doesn't copy dat but simply create pointers to the data
  * Tiny space penalty
  * Very quick (seconds)
* Can scheduled or taked on demand
  * Persists until user-set expiration

#### Where Do Snapshots Go?
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_911_wheredosnapshotsgo.png){: .align-right}

* Top level of every volume
  * `.snapshot` directory
  * Exists even if empty
* Not visible to `ls –ltarh` command
* Accessed via NFS or Hadoop shell possibe if we include `.snapshot` in the query path.

### Snapshot Architecture

{% include video id="157616758" provider="vimeo" %}


### Use Case: Protection From Human Error
#### Situation
You know human error is one of the most common causes of lost or corrupted data, and you want to protect against loss of time and data.
#### Solution
Make frequent snapshots of key data. To control space requirements, set reasonable retain times.


## Configure and use Snapshots
### Take a Snapshot
#### Manual Snapshot: MCS
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_921_manualsnapshotmcs.png)

#### Manual Snapshot: CLI
* Command line:
```bash
$ maprcli volume snapshot create -volume <vol name> -snapshotname <snapshot name>
```
* Manual snapshots do not expire automatically


### Snapshot Schedules
Scheduled snapshots expire automatically. Normal, Important and Critical are pre-defined schedules.

1. Normal
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_922_scheduledsnapshots_normal.png)

2. Important
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_922_scheduledsnapshots_important.png)

3. Critical
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_922_scheduledsnapshots_critical.png)

4. Custom Schedules
* You can create custom schedules through
  * the MCS
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_922_scheduledsnapshots_custom2.png)
  * or by using the CLI:
  ``` bash
  maprcli schedule create -schedule '{"name":"Schedule-1","rules":[{"frequency":"once","retain":"1w","time":13,"date":"12/4/2018"}]}'
  ```

### Schedule a Snapshot
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_923_schedule_a_snapshot.png)

### Display Schedules
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_924_display_schedules.png)

### Recover Data From a Snapshot
1. Create a Recovery Directory
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_926_createrecoverydirectory.png)

2. Copy Files With Hadoop Commands
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_926_copyfileswithhadoopcommands.png)

3. Use Linux commands, if the cluster file system is mounted.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_926_copyfileswithlinuxcommands.png)

### Preserve a Snapshot
Preserving a snapshot removes its expiration date
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_927_preserve_a_snapshot.png)

### Snapshot Maintenance
* Scheduled snapshots are deleted automatically
* Manual snapshots must be deleted manually (through the CLI or MCS)
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_928_snapshotmaintenance.gif)



# Mirrors
## Describe Mirroring

> About Mirror Volumes

{% include video id="162485040" provider="vimeo" %}

> How Mirrors Are Created

{% include video id="162485900" provider="vimeo" %}

## Configure and Use Local Mirrors
### Create a Local Mirror Volume
#### Create a Mirror Volume
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1021_createamirrorvolume2.png)

#### Set a Mirror Schedule
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1021_setamirrorschedule2.png)

#### Manually Start the Mirror
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1021_manuallystartthemirror2.png)

### Promote a Mirror
* Promoting a mirror changes it from Read-Only to a standard Read-Write Volume.
* Typically a mirror is promoted when the source volumes remains unavailable.
* Once a mirror is promoted any mirroring schedule associated with the volume is diabled.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1022_promoteamirror.png)

### Rules for Promoted Mirrors
**Important!**
* To use promotable mirrors, the source and destination volumes must have the same name and mount point
* Mirror volumes that are promoted to standard volumes cannot be written to until they are explicitly mounted

```bash
$ maprcli volume modify -name <name> -type rw
$ maprcli volume mount -name <name> -path <path>
```

## Use Cascading and Remote Mirrors
### Cascading Mirrors
* A cascading mirror is a mirror of a mirror of source volumes.
* Cascading is useful for deployment
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1031_cascadingmirrorsoverview.png)

* Deployment Without Cascading Mirrors :
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1031_withoutcascadingmirrors.png)

* Deployment With Cascading Mirrors
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1031_withcascadingmirrors.png)


### Remote Mirrors
1. Local vs. Remote Mirrors
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1032_remotemirrors_localmirrors.jpg)

2. Remote Mirrors and Data Protection
![]({{ site.url }}{{ site.baseurl }}/images/mapr_config/adm202_1032_remotemirrors2.gif)


## Review of Mirrors
* Local mirrors
  * Source and destination volumes in the same cluster
* Cascading mirrors
  * Source and destination volumes are both mirrors
* Remote mirrors
  * Source and destination volumes in different clusters
