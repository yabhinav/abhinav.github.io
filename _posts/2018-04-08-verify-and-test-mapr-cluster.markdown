---
layout: single
title: "Verify and Test MapR Cluster"
date: "2018-04-08 12:19:27 +0100"
excerpt: "Verify Cluster Status, Run Post-Install Benchmark Tests and Explore the Cluster Structure"
tags: [MapR, Hadoop, Adminstration]
categories: [ MapR ]
comments: true
published: true
share: true
toc : true
toc_label : "Verify Installtion"
toc_icon : "flag-checkered"
---
# Intro
After installing a MapR Cluster, this post helps in :
1. Verifying Cluster Status
2. Run Post-Install Benchmark Tests
3. Explore the Cluster Structure

# Verify Cluster Status

## The MapR Control System (MCS)

### Connect
Open the MapR control system on `https://<node ip address>:8443` on any of the control nodes.

![MCS Connect]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_311_maprcontrolsystemconnect.png)

### Dashboard
The dashboard view opens by default and there is a navigation pane on the left gives access to all MapR features. A Cluster Heatmap in centre gives color coded overview of the health of each node

![MCS Dashboard]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_311_maprcontrolsystemdashboard.png)


### Heatmap
The Zoom Slider on the heatmap increases the size of icon for each node to see more details. Warnings will appear inside relevant node icon. Any alarms detected will be in the Alarms pane below the heatmap.

![MCS Heatmap]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_311_maprcontrolsystemheatmap.png)

### Services Pane
`Services pane` lists the services on the cluster. It will also display any failed or stopped services.

![MCS Services]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_311_maprcontrolsystemservicespane.png)


## Using the Command-Line Interface (CLI)
When the cluster is up, hadoop commands can be run on the cluster. Alternatively we can also use `maprcli` to interact with the cluster.

1. List Cluster Contents
![List Cluster Contents]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_312_cli_listclustercontents.png)
2. List Nodes
![List Nodes]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_312_cli_listnodes.png)
3. List Services
![List Services]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_312_cli_listservices.png)
4. Show Processes
![Show Processes]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_312_cli_showprocesses.png)

Visit [MapR Documentation](http://doc.mapr.com/display/MapR/Home) to see full list of `maprcli` commands.

# Run Post-Install Benchmark Tests
## Post-Installation Testing and Benchmarks

* Since its not feasible to test performance on each individual job; benchmarks are used as a proxy for workload.
* Use benchmarks to establish a baseline and test ongoing performance. The goal is to have performance in optimal range not stress to limits or underperform.
* Single benchmark tests won't give whole picture; comparing different benchmarks will indicate expected performance.

## Benchmark Types
There are two types of bench benchmarks.
1. Synthetic Benchmarks
  * Industry standard tests designed to see the max capabilities of a component.
  * Exercise the system to see what it can do
  * If you see a problem, you know it’s not the code but the hardware.
2. Application Benchmarks
  * Provide performance information on certain jobs to simulate the type of jobs you run on the system.
  * Use the same data set each time
  * Problems could be system components, or code  

## Verification Through the MCS
1. Cluster Utilization
MCA can be used to monitor jobs and cluster performance. Monitor the heatmaps while the benchmarks are running to see how the cluster is performing like `Cluster Utilization` and `Yarn` specifications on the dashboard provides overall and realtime information on running Yarn jobs.
![Cluster Utilization]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_323_verificationthroughmcs_clusterutilization.png)

2. Node Performance
Double click on a node to expand its properties and view its performance. Test and make changes iteratively to get best balance.
![Node Performance]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_323_verificationthroughmcs_nodeperformance.png)


## Synthetic Benchmark Tests
1. `DFSIO`
![DFSIO]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_324_syntheticbenchmarktests_dfsio.png){: .align-right}
  * `DFSIO` can be used to test reads and writes for the cluster
  * Runs a MapReduce job
  * Provides an estimate of I/O performance
  * Reasonable starting performance: `DFSIO` results at least _50%_ of maximum from `disk-test.sh` run as pre-installation checks.
  * IOZone is a destructive test and will ruin hadoop software if run post installation.

2. `RWSpeedTest`
  * Post-install test made available by MapR
  * Provides aggregate read/write speeds for the node
  * Unlike DFSIO, RWSpeedTest is not a MapReduce jobs
  ```bash
   /root/post-install/runRWSpeedTest.sh
   ```

3. `TeraGen` and `TeraSort`
  * Comprehensive synthetic benchmark designed to measure upper limits of performance
  * TeraGen generates data (run only once)
  ```bash
  hadoop jar /opt/mapr/hadoop/hadoop-2.7.0/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0-mapr-1506.jar teragen 5000000 /data/teragen-data
  ```
  *  TeraSort sorts it ( run several times while tuning performance)
  ```bash
  hadoop jar /opt/mapr/hadoop/hadoop-2.7.0/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0-mapr-1506.jar terasort /data/teragen-data /data/teragen-output
  ```




# Cascading and Remote Mirrors
## Explore the Cluster Structure

1. Mounting the Cluster File System
​During installation, the cluster file system is automatically mounted to
`/mapr/<cluster name>` in localFS.
![]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_331_mountingclusterfilesystem.png)

2. Cluster File System(Hadoop) Versus Local File System(Linux)
![]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_331_clusterfilesystemvslocalfilesystem2.png)

3. Since MapR mounts the HadoopFS to localFS it is possible to see the cluster filesystem with `ls` command.
![View Cluster]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_331_viewthecluster.png)

## MapR System Volumes

{% include video id="159874879" provider="vimeo" %}

## Local Files and Directories
MapR also creates files and directories during installation. Some of them are listed here :

![MapR local files]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_333_localfilesanddirectories.png)


![]({{ site.url }}{{ site.baseurl }}/images/mapr_install/.png)
