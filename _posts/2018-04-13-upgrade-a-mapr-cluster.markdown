---
layout: single
title: "Upgrade a MapR Cluster"
date: "2018-04-13 14:50:55 +0100"
excerpt: "Upgrade a MapR Converged Data Platform"
tags: [MapR, Hadoop, Administration]
categories: [ MapR ]
comments: true
published: true
share: true
toc : true
toc_label : "Upgrade Plan"
toc_icon : "download"  # corresponding Font Awesome icon name (without fa prefix)
---

# Overview
This post takes you through the process of upgrading a MapR cluster, beginning with what to include in a cluster upgrade plan, how to perform pre-upgrade testing, and then to upgrading MapR and patching core software, ecosystem components, and MapR clients.


# Plan the Upgrade

## Identify Upgrade Methods
### Software Releases and Patches

* Each MapR release has a version number written in the format X.Y.Z
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_111_software1.png)

* Upgrade the software to move from one version to the next
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_111_software2.png)

* Patch the software to update to the current version. It is advisable to install patches after upgrade, when the patch is installed the MapR software version doesn't change.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_111_software3.png)

### Select an Upgrade Method
* Upgrades can be performed **Offline/Rolling** and **Manual/Scripted**. Choosing an Upgrade method will involve assessing the risks and availability requirements of the cluster.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_112_upgrade_method_1.png)
* An Offline upgrade poses least risk since it's simpler where the complete cluster is offline and upgrade are performed manually or scripted in a short time.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_112_upgrade_method_2.png)
* Rolling upgrades are complex and performed in stages (a few nodes at a time) while the cluster remains available; which requires _Highly Availability_ features so that critical services running on a node that is taken offline will automatically startup in an another node.
  * Group nodes for upgrades so that critical services are available
  * Plan when there is very low cluster activity
  * Cluster users should expect performance segregation and its best to limit cluster operations to only critical functions during rolling upgrade.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_112_upgrade_method_3.png)


### Community Edition and Rolling Upgrades
* You can perform Rolling Upgrade on Community Edition with single services.
* Expect interruptions when nodes running the only copy of a service are upgraded
* Minimize cluster access
* With 10 or fewer nodes, an offline upgrade probably makes the most sense

![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_113_community_edition.png)

### Supported Upgrade Methods (to v5.2)
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_114_support_upgrade.png)

\* Only supported for versions of MapR that were installed using the MapR Installer

### Method One: Offline Upgrade
An offline installer follows the following steps.
#### Administrator:
MapR Installer cannot stop non-MapReduce jobs, so `mapr` admin should perform following steps :
1. Stop non-MapReduce jobs
2. Start MapR Installer (if using)

#### Administrator or Installer:
Remaining steps completed by Installer or Manually by adminstrator are :
3. Stop MapReduce jobs
4. Stops cluster services
5. Upgrades packages on all nodes
6. Brings the cluster back online
7. Upgrades ecosystem components

### Method Two: Scripted Rolling Upgrade
* Upgrades MapR software one node at a time
   * Cluster remains operational
* MFS service on each node goes down while packages are upgraded
   * Does not raise the under-replication alarm since time required for upgrade is short enough.
* No ecosystem components are upgraded
* Process can be monitored from MCS from any node that is not currently being upgraded.
* After upgrade, Warden restarts services
   * Including ecosystem components if compatible with the new release.

### Method Three: Manual Rolling Upgrade
* Upgrade MapR software one node at a time
   * Cluster remains operational
* Possible to upgrade some nodes in parallel
   * Plan carefully!
   * Services on nodes being upgraded are unavailable


## Develop a Plan and Prepare for Upgrade
### High-Level Overview

{% include video id="199099214" provider="vimeo" %}

### Plan: Determine What to Include
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_122_plan_determine.png)

\* MEPs were introduced in MapR 5.2

### Plan: Develop a Test Plan
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_123_develop_test_plan.png){: .align-right}

* Run tests before and after each upgrade step
  * Core
  * Ecosystem components
  * Clients
* Test basic functionality
  * Verify cluster access and volumes
  * Use maprcli, hadoop fs, MCS
* Test jobs and queries
  * Based on the components you use

### Plan: Create an Upgrade Schedule
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_124_upgrade_schedule.png)

### Select an Upgrade Method
#### Weeks Ahead
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_125_plan1.png){: .align-right}

* Review release notes
  * Behaviour changes?
  * New features to enable?
* Prepare
  * Download the installer, packages, etc.
  * Verify node specifications
* Upgrade on a test cluster
  * Document surprises
  * Prepare configuration files for production cluster

#### Days Ahead
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_125_plan2.png){: .align-right}

* Remind cluster users of upcoming upgrade
* Verify cluster health
* Run tests and record results
* Back up critical data

#### Day of Upgrade
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_125_plan3.png){: .align-right}

* Verify cluster health and clear alarms
* Terminate jobs and clear the pipeline
* Stop cross-cluster operations
  * Volume mirroring
  * Table replication
* Upgrade, then verify and test

#### Post-Upgrade
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_125_plan4.png){: .align-right}

* Update applications to use new binaries
* Enable and test new features
* Put the cluster back into full production mode

## Summarise the Upgrade Process
### Included During an Upgrade of MapR Core
#### Included
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_131_included.png)

#### Not Included
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_131_not_included.png){: .align-right}
* Ecosystem components
  * Hive
  * Pig
  * Spark
  * Drill
  * etc.
* MapR Clients
  * Edge nodes that are not of the cluster
* Exception:  ecosystem components are upgraded if using the Installer

### Core Upgrades

#### New Files and Folders
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_132_new_files_folders.png){: .align-right}
During the upgrade process :

* Product binaries are replaced with new versions
* Additional folders may be created (if required for upgrade)
* `/opt/mapr/MapRBuildVersion` updated

#### Configuration Files
New default configuration files created when new features are added during upgrade.

![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_132_configuration_files.png)

Important!  Merge required changes into active configuration files.
{: .notice--info}

#### Hadoop Common
Following steps are performed when upgrade includes a new Hadoop Common.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_132_hadoop_common.png)


# Upgrade MapR Core

## Prepare to Upgrade
### High Level Overview
{% include video id="199716054" provider="vimeo" %}


## Upgrade MapR Core Software
### High-Level Overview
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210-221_overview.gif)

### Upgrade Options
{% include video id="201210361" provider="vimeo" %}

### Post Upgrade
1. Update Configuration Files
* Observe and merge changes or new configurations
* Manually update configuration files on all nodes
* Save time by doing this step during upgrade on test cluster
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_224_post_upgrade1.png)

2. Update License Files
* First upgrade to MapR 5.1 or later requires update to base license on all nodes during to the changes implemented in licensing.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_224_post_upgrade2.png)

### Verify and Validate
{% include video id="199719231" provider="vimeo" %}


# Upgrade Ecosystem Components and MapR Clients

## Upgrade Ecosystem Components
### Overview
* You don't need to upgrade all ecosystem components together during upgrade.
* Determining what ecosystems requires upgrade is part of pre-upgrade planning
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_311_upgrade_ecosystem1.png)

### MEPs
* MapR 5.2, introduces MapR Ecosystem Packs or MEPs.
* When you upgrade ecosystem components upgrade to versions that all include same MEPs, choose one of the MEPs available for a MapR version to ensure they work together well.
* As of MapR 5.2, must use ecosystem components that belong to the same MapR Ecosystem Pack (MEP)
* New MEPs released on a regular basis
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_311_upgrade_ecosystem2.png)

### Details
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_131_not_included.png){: .align-right}
* You may choose not to upgrade even if a new version is available for MEPs.
* Can wait unless they are incompatible with new version of MapR
* Follow pre  * and post-upgrade steps in the MapR documentation
* Always upgrade ecosystem components after an upgrade from MapR 3.x

### Minor and Major Upgrades
Ecosystem components may have
1. Minor upgrades   * package or patch update
2. Major upgrades   * take you to new version

#### Minor Upgrades
Minor upgrade may just require a package update
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_313_aa.png)

1. Verify repository is configured to get the latest packages
2. Determine package names
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_313_a2.png)

3. Upgrade
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_313_a3.png)

#### Major Upgrades
Major upgrade may require additional steps
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_313_bb.png)

There may be pre  * and post  * upgrade testing required depending on the component :
1. Perform pre-upgrade steps (Ex: _Hive_   * backup configuration)
2. Upgrade
3. Perform post-upgrade steps (Ex: _Hive_   * update the schema)

#### Post-Upgrade Testing
Once you complete the upgrade perform the same pre-upgrade tests and compare the results.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_313_c.png)

## Upgrade MapR Clients
### Types of Client Access
{% include video id="201341117" provider="vimeo" %}

### Upgrade or Migrate?
The Direct Access NFS functionality is provided in the mapr-nfs core package and does not need to be upgraded separately. So we need to upgrade/migrate the POSIX/MapR client
#### Option: Upgrade
If you choose to upgrade your NFS loopback POSIX client follow these steps :
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210-322_a.png)

Note! Support for NFS loopback client will be removed from a future version of MapR.
{: .notice--warning}

#### Recommendation: Migrate
* We recommend to migrate to FUSE client rather than upgrading NFS client.
* Platinum POSIX client requires a new license
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210-322_b.png)

#### Test
Finish up the process by testing the new client after upgrade or migrate.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_322_test-3c.png)


## Additional Upgrade Considerations
### Patching a Fresh Upgrade
After upgrading to new version of MapR check for minor patches that provide minor code release that are not included with base version upgrade.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_332_patching_a_fresh_upgrade.png)

Check for a patch at:
`http://package.mapr.com/patches/releases/<version>/<OS>`

### Considerations for Multiple Clusters

#### Mirroring Between Clusters
* Volumes must be mirrored to a cluster at the same, or higher, version
* Upgrade the destination cluster first
* Upgrade both clusters if mirroring both ways
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_333_mirroring-between-clusters.png)


#### Table Replication Between Clusters
* Clusters involved in table replication can be at different versions
![]({{ site.url }}{{ site.baseurl }}/images/mapr_upgrade/adm210_333_table-replication.png)

#### Enable New Features
* Review release notes for automatically enabled features
  * Anticipate behavior changes
* Recommended: wait to enable optional new features
  * Make sure the upgraded cluster is stable before implementing new features
