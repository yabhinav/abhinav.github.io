---
title: Install MapR Converged Data Platform.
excerpt: "Install the MapR Converged Data Platform and Add a MapR License"
tags: [MapR, Hadoop, Adminstration, Installation]
categories: [ MapR ]
comments: true
published: true
share: true
toc : true
toc_label : "MapR Cluster Installtion"
toc_icon : "flag-checkered"
---

# Pre-Install Checks
* Verify prerequisites on every node!
  * Correct any problems before you start installation.
* Nodes that are out of spec are a common cause of a failed installation
* Check the documentation for the full list of prerequisites

# Installation Overview
1. Download the `mapr-setup.sh` script to the node that will be part of the cluster
```shell
wget package.mapr.com/releases/installer/mapr-setup.sh
 ```
2. Run the script as root
``` shell
bash ./mapr-setup.sh
```
3. The script will perform the following tasks :
  * Verified dependencies on the install Nodes
  * Checks internet connectivity; if not available it uses archive files.
  * Establishes MapR administrator account (`mapr`)
  * Creates Repository with required package files; if you provided packages in archive files it will create a local repository.
4. Start the MapR Installer Web Interface on Installare node in webbrowser.
```
https://<external IP address>:9443
```
5. Login as MapR administrator (default is `mapr`)
6. Provide the details to installer as required and finish the installation.

# Prepare to Install
## Information to provide:
* Nodes to include in the cluster
* Services to install on the cluster
* Credentials for the MapR administrator account

![MapR Installer]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_213_preparetoinstall.png)

## Select Version and Services
### Version and Edition
Select the Community Free Edition or Enterprise Edition if you have the License

![Version & Edition]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_214_versionandedition.png)

### License Option
Get Trail License from you MapR Account or directly signup with the install later which will download the license.

![License Option]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_214_licenseoption.png)

### Select Services: Overview
Select Pre-Provisioned Templates and Expand `Show Advanced service options` to see the summary of components installed.

![Select Services]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_214_selectservicesoverview.png)

### Custom Services
Also you can choose to install custom services and versions. Warnings will be displayed when incompatible services are selected as shown below.

![Custom Services]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_214_customservices.png)

## Database Setup
Fields displayed will depend on services selected. If you choose to install Hue, Hive, Oozie or Metrics enter the database information as shown below:

1. `Database Type`
  * Use Embedded
  * Existing MySQL server
  * Install MySQL Server
2. `Username` and `password` for DB User
3. Name for each `DB Schema` like hue , oozie, hive etc..

You can also choose to have multiple database types but create unique users for each database.

![Database Setup]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_215_databasesetup.png)


# Set Up the Cluster

1. In the Setup Cluster screen provide the `mapr` user details created earlier.
2. In the `Cluster Name` filed enter a unique DNS name for the cluster like `mymapr.test.com`

![Cluster Setup]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_216_setupthecluster.png)


## Configure Nodes
On the Configuration page enter resolvable IPAddress/Hostnames/HostNameExpressions for the nodes that need to be included in the cluster.

> Note: If you are using cloud like AWS/Azure use internal resolvable privateIP address or FQDN.

![Specify Nodes]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_217_specifynodes.png)

## Configure Disks
In the Disks section enter a `,`  list of disks to use (Unused and Unmounted disks). By default disks are configured to be reformatted under ``Advanced Disk Options``

![Configure Disks]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_217_configuredisks.png)

## Configure Remote Authentication
Use Passwordless SSH between nodes to connect instead of `mapr` password.

![SSH Passwordless]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_217_configureremoteauthentication.png)


## Verify Nodes
### Verification Process
Installer will connect to all nodes and check if they statisfy minimum requirements. Notice the Following Indication on each node icon :
* White - Verification InProgress
* Green - Ready for installation
* Yellow - Warnings but can be installed
* Red - Node cannot be part of cluster

![Node Verification]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_218_verificationprocess.png)


### Verification Problems
If Node verification posts Warnings or Errors, Click on Node Icon and see the recommendation failed and correct them manually.
If the problem cannot be resolved , remove the node from the `Specify Nodes` step by reviewing it.You can add the node at later point of time by resolving the issue.

![Node Verification Problems]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_218_verificationproblems.png)


## Configure Service Layout
### Default Service Layout
After Verification as default service layout window is displayed as shown below:

![Default Service Layout]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_219_defaultservicelayout.png)


### View Node Layout
Click on `View Node Layout` to examine the which services are installed on which node.

![View Node Layout]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_219_viewnodelayout.png)

### Advanced Configuration
You can modify the default service layout as per needs by clicking on `Advanced Configuration` as shown below by drag&drop components and nodes.

![Advanced Configuration]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_219_advancedconfiguration.png)


# Install MapR
## Installation
Click `Next` to begin Installtion and Monitor the Progress of Installation.

![Install Cluster]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_2110_installmapr-installation.png)

## Completion
When finished installer will suggest whether a valid license to be applied or not in the later stages for all services to work.

![Install Cluster]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_2110_installmapr-completion.png)

## Service Links
Once Installation is completed a list of Service URLs like  CLDB, History Server , Hue and MapR Controller are provide. Login to those services and monitor status of components.

![Install Cluster]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_2110_installmapr-servicelinks.png)



# Add a MapR License
## Apply a License
Follow the steps below to install/update a new license :
1. Log into the MCS `https://<node IP address>:8443` as user `mapr`.

    ![login MCS]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_221_logintothemcs.png)

2. Click `Manage licenses` on Top Right Corner.

    ![Manage License]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_221_clickmanagelicense.png)

3. Upload or Copy-Paste the licenses and Click `Apply Licenses`.

    ![Apply License]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_221_applylicense.png)



## Restart Services
* After Installing License review the cluster dashboard to see some failed services because MapR tried to start these services without a valid licenses.
    ![Service Status]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_222_viewservicestatus.png)

* Click on `Manage Service`  next to the node with failed service where you can `Start` , `Stop` or `Restart` Services.

    ![Restart Services]({{ site.url }}{{ site.baseurl }}/images/mapr_install/adm200_222_restartservices.png)



This concludes the installation of MapR Enterprise Edition with a valid license. Read the Next article to manage MapR Configuration.
