---
title: Oracle solutions on Microsoft Azure | Microsoft Docs
description: Learn about supported configurations and limitations of Oracle solutions on Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: romitgirdhar
manager: jeconnoc
tags: azure-resource-management

ms.assetid: 5d71886b-463a-43ae-b61f-35c6fc9bae25
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/02/2018
ms.author: rogirdh
ms.custom: seodec18
---
# Oracle solutions and their deployment on Microsoft Azure
This article covers information required to successfully deploy various Oracle solutions on Microsoft Azure. These solutions are based on Virtual Machine images published by Oracle in the Azure Marketplace. To get a list of currently available images, run the following command:
```azurecli-interactive
az vm image list --publisher oracle -o table --all
```
As of 6-16-2017 the list of images are the following:
```bash
Offer                   Publisher    Sku                     Urn                                                          Version
----------------------  -----------  ----------------------  -----------------------------------------------------------  -------------
Oracle-Database-Ee      Oracle       12.1.0.2                Oracle:Oracle-Database-Ee:12.1.0.2:12.1.20170202             12.1.20170202
Oracle-Database-Se      Oracle       12.1.0.2                Oracle:Oracle-Database-Se:12.1.0.2:12.1.20170202             12.1.20170202
Oracle-Linux            Oracle       6.4                     Oracle:Oracle-Linux:6.4:6.4.20141206                         6.4.20141206
Oracle-Linux            Oracle       6.7                     Oracle:Oracle-Linux:6.7:6.7.20161007                         6.7.20161007
Oracle-Linux            Oracle       6.8                     Oracle:Oracle-Linux:6.8:6.8.20161020                         6.8.20161020
Oracle-Linux            Oracle       6.9                     Oracle:Oracle-Linux:6.9:6.9.20170406                         6.9.20170406
Oracle-Linux            Oracle       7.0                     Oracle:Oracle-Linux:7.0:7.0.20141217                         7.0.20141217
Oracle-Linux            Oracle       7.2                     Oracle:Oracle-Linux:7.2:7.2.20161020                         7.2.20161020
Oracle-Linux            Oracle       7.3                     Oracle:Oracle-Linux:7.3:7.3.20170320                         7.3.20170320
Oracle-WebLogic-Server  Oracle       Oracle-WebLogic-Server  Oracle:Oracle-WebLogic-Server:Oracle-WebLogic-Server:12.1.2  12.1.2
```

These images are considered "Bring Your Own License" and as such you will only be charged for compute, storage, and networking costs incurred by running a VM.  It is assumed you are properly licensed to use Oracle software and that you have a current support agreement in place with Oracle. Oracle has guaranteed license mobility from on-premises to Azure. See the published [Oracle and Microsoft](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html) note for details on license mobility. 

Individuals can also choose to base their solutions on a custom image they create from scratch in Azure or upload a custom image from their on premises environment.

## Support for JD Edwards
According to Oracle Support note [Doc ID 2178595.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=573435677515785&id=2178595.1&_afrWindowMode=0&_adf.ctrl-state=o852dw7d_4) , JD Edwards EnterpriseOne versions 9.2 and above are supported on **any public cloud offering** that meets their specific `Minimum Technical Requirements` (MTR).  You need to create custom images that meet their MTR specifications for OS and software application compatibility. 

## Oracle Database virtual machine images
Oracle supports running Oracle DB 12.1 Standard and Enterprise editions in Azure on virtual machine images based on Oracle Linux.  For the best performance for production workloads of Oracle DB on Azure, be sure to properly size the VM image and use Managed Disks that are backed by Premium Storage. For instructions on how to quickly get an Oracle DB up and running in Azure using the Oracle published VM image, [try the Oracle DB Quickstart walkthrough](oracle-database-quick-create.md).

### Attached disk configuration options

Attached disks rely on the Azure Blob storage service. Each standard disk is capable of a theoretical maximum of approximately 500 input/output operations per second (IOPS). Our premium disk offering is preferred for high-performance database workloads and can achieve up to 5000 IOps per disk. While you can use a single disk if that meets your performance needs - you can improve the effective IOPS performance if you use multiple attached disks, spread database data across them, and then use Oracle Automatic Storage Management (ASM). See [Oracle Automatic Storage overview](https://www.oracle.com/technetwork/database/index-100339.html) for more Oracle ASM specific information. For an example of how to install and configure Oracle ASM on a Linux Azure VM - you can try the [Installing and Configuring Oracle Automated Storage Management](configure-oracle-asm.md) tutorial.

## Oracle Real Application Cluster (Oracle RAC)
Oracle RAC is designed to mitigate the failure of a single node in an on-premises multi-node cluster configuration. It relies on two on-premises technologies which are not native to hyper-scale public cloud environments: network multi-cast and shared disk. If your database solution requires Oracle RAC in Azure, you need 3rd party software to enable these technologies. For more information on Oracle RAC, please see the [FlashGrid solution page](https://www.flashgrid.io/oracle-rac-in-azure/).

## High availability and disaster recovery considerations
When using Oracle Databases in Azure, you are responsible for implementing a high availability and disaster recovery solution to avoid any downtime. 

High availability and disaster recovery for Oracle Database Enterprise Edition (without relying on Oracle RAC) can be achieved on Azure using [Data Guard, Active Data Guard](https://www.oracle.com/technetwork/articles/oem/dataguardoverview-083155.html), or [Oracle Golden Gate](https://www.oracle.com/technetwork/middleware/goldengate), with two databases on two separate virtual machines. Both virtual machines should be in the same [virtual network](https://azure.microsoft.com/documentation/services/virtual-network/) to ensure they can access each other over the private persistent IP address.  Additionally, we recommend placing the virtual machines in the same availability set to allow Azure to place them into separate fault domains and upgrade domains.  Should you want to have geo-redundancy - you can have these two databases replicate between two different regions and connect the two instances with a VPN Gateway.

We have a tutorial "[Implement Oracle DataGuard on Azure](configure-oracle-dataguard.md)", which walks you through the basic setup procedure to trial this on Azure.  

With Oracle Data Guard, high availability can be achieved with a primary database in one virtual machine, a secondary (standby) database in another virtual machine, and one-way replication set up between them. The result is read access to the copy of the database. With Oracle GoldenGate, you can configure bi-directional replication between the two databases. To learn how to set up a high-availability solution for your databases using these tools, see [Active Data Guard](https://www.oracle.com/technetwork/database/features/availability/data-guard-documentation-152848.html) and [GoldenGate](https://docs.oracle.com/goldengate/1212/gg-winux/index.html) documentation at the Oracle website. If you need read-write access to the copy of the database, you can use [Oracle Active Data Guard](https://www.oracle.com/uk/products/database/options/active-data-guard/overview/index.html).

We have a tutorial "[Implement Oracle GoldenGate on Azure](configure-oracle-golden-gate.md)", which walks you through the basic setup procedure to trial this on Azure.

Despite having an HA and DR solution architected in Azure, you want to ensure you have a backup strategy in place to restore your database.  We have a tutorial [Backup and recover an Oracle Database](oracle-backup-recovery.md) which walks you through the basic procedure for establishing a consistent backup.

## Oracle WebLogic Server virtual machine images
* **Clustering is supported on Enterprise Edition only.** You are licensed to use WebLogic clustering only when using the Enterprise Edition of WebLogic Server. Do not use clustering with WebLogic Server Standard Edition.
* **UDP multicast is not supported.** Azure supports UDP unicasting, but not multicasting or broadcasting. WebLogic Server is able to rely on Azure UDP unicast capabilities. For best results relying on UDP unicast, we recommend that the WebLogic cluster size be kept static, or be kept with no more than 10 managed servers included in the cluster.
* **WebLogic Server expects public and private ports to be the same for T3 access (for example, when using Enterprise JavaBeans).** Consider a multi-tier scenario where a service layer (EJB) application is running on a WebLogic Server cluster consisting of two or more VMs, in a vNet named **SLWLS**. The client tier is in a different subnet in the same vNet, running a simple Java program trying to call EJB in the service layer. Because it is necessary to load balance the service layer, a public load-balanced endpoint needs to be created for the Virtual Machines in the WebLogic Server cluster. If the private port that you specify is different from the public port (for example, 7006:7008), an error such as the following occurs:

       [java] javax.naming.CommunicationException [Root exception is java.net.ConnectException: t3://example.cloudapp.net:7006:

       Bootstrap to: example.cloudapp.net/138.91.142.178:7006' over: 't3' got an error or timed out]

   This is because for any remote T3 access, WebLogic Server expects the load balancer port and the WebLogic managed server port to be the same. In the preceding case, the client is accessing port 7006 (the load balancer port) and the managed server is listening on 7008 (the private port). This restriction is applicable only for T3 access, not HTTP.

   To avoid this issue, use one of the following workarounds:

  * Use the same private and public port numbers for load balanced endpoints dedicated to T3 access.
  * Include the following JVM parameter when starting WebLogic Server:

         -Dweblogic.rjvm.enableprotocolswitch=true

For related information, see KB article **860340.1** at <https://support.oracle.com>.

* **Dynamic clustering and load balancing limitations.** Suppose you want to use a dynamic cluster in WebLogic Server and expose it through a single, public load-balanced endpoint in Azure. This can be done as long as you use a fixed port number for each of the managed servers (not dynamically assigned from a range) and do not start more managed servers than there are machines the administrator is tracking (that is, no more than one managed server per virtual machine). If your configuration results in more WebLogic servers being started than there are virtual machines (that is, where multiple WebLogic Server instances share the same virtual machine), then it is not possible for more than one of those instances of WebLogic servers to bind to a given port number – the others on that virtual machine fail.

   If you configure the admin server to automatically assign unique port numbers to its managed servers, then load balancing is not possible because Azure does not support mapping from a single public port to multiple private ports, as would be required for this configuration.
* **Multiple instances of Weblogic Server on a virtual machine.** Depending on your deployment’s requirements, you might consider the option of running multiple instances of WebLogic Server on the same virtual machine, if the virtual machine is large enough. For example, on a medium size virtual machine, which contains two cores, you could choose to run two instances of WebLogic Server. Note however that we still recommend that you avoid introducing single points of failure into your architecture, which would be the case if you used just one virtual machine that is running multiple instances of WebLogic Server. Using at least two virtual machines could be a better approach, and each of those virtual machines could then run multiple instances of WebLogic Server. Each of these instances of WebLogic Server could still be part of the same cluster. Note, however, it is currently not possible to use Azure to load-balance endpoints that are exposed by such WebLogic Server deployments within the same virtual machine, because Azure load balancer requires the load-balanced servers to be distributed among unique virtual machines.

## Oracle JDK virtual machine images
* **JDK 6 and 7 latest updates.** While we recommend using the latest public, supported version of Java (currently Java 8), Azure also makes JDK 6 and 7 images available. This is intended for legacy applications that are not yet ready to be upgraded to JDK 8. While updates to previous JDK images might no longer be available to the general public, given the Microsoft partnership with Oracle, the JDK 6 and 7 images provided by Azure are intended to contain a more recent non-public update that is normally offered by Oracle to only a select group of Oracle’s supported customers. New versions of the JDK images will be made available over time with updated releases of JDK 6 and 7.

   The JDK available in this JDK 6 and 7 images, and the virtual machines and images derived from them, can only be used within Azure.
* **64-bit JDK.** The Oracle WebLogic Server virtual machine images and the Oracle JDK virtual machine images provided by Azure contain the 64-bit versions of both Windows Server and the JDK.

## Next steps
You now have an overview of current Oracle Solutions on Microsoft Azure. Your next step is to deploy your first Oracle Database on Azure.
- Try the [Create an Oracle Database on Azure](oracle-database-quick-create.md) tutorial to get started.
