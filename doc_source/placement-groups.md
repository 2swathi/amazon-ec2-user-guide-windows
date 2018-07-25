# Placement Groups<a name="placement-groups"></a>

You can launch or start instances in a *placement group*, which determines how instances are placed on underlying hardware\. When you create a placement group, you specify one of the following strategies for the group:
+ *Cluster*—clusters instances into a low\-latency group in a single Availability Zone
+ *Spread*—spreads instances across underlying hardware

There is no charge for creating a placement group\.

**Topics**
+ [Cluster Placement Groups](#placement-groups-cluster)
+ [Spread Placement Groups](#placement-groups-spread)
+ [Placement Group Rules and Limitations](#concepts-placement-groups)
+ [Creating a Placement Group](#create-placement-group)
+ [Launching Instances in a Placement Group](#launch-instance-placement-group)
+ [Changing the Placement Group for an Instance](#change-instance-placement-group)
+ [Deleting a Placement Group](#delete-placement-group)

## Cluster Placement Groups<a name="placement-groups-cluster"></a>

A cluster placement group is a logical grouping of instances within a single Availability Zone\.

Cluster placement groups are recommended for applications that benefit from low network latency, high network throughput, or both, and if the majority of the network traffic is between the instances in the group\. To provide the lowest latency and the highest packet\-per\-second network performance for your placement group, choose an instance type that supports enhanced networking\. For more information, see [Enhanced Networking](enhanced-networking.md)\.

We recommend that you launch the number of instances that you need in the placement group in a single launch request and that you use the same instance type for all instances in the placement group\. If you try to add more instances to the placement group later, or if you try to launch more than one instance type in the placement group, you increase your chances of getting an insufficient capacity error\.

If you stop an instance in a placement group and then start it again, it still runs in the placement group\. However, the start fails if there isn't enough capacity for the instance\.

If you receive a capacity error when launching an instance in a placement group that already has running instances, stop and start all of the instances in the placement group, and try the launch again\. Restarting the instances may migrate them to hardware that has capacity for all the requested instances\.

## Spread Placement Groups<a name="placement-groups-spread"></a>

A spread placement group is a group of instances that are each placed on distinct underlying hardware\.

Spread placement groups are recommended for applications that have a small number of critical instances that should be kept separate from each other\. Launching instances in a spread placement group reduces the risk of simultaneous failures that might occur when instances share the same underlying hardware\. Spread placement groups provide access to distinct hardware, and are therefore suitable for mixing instance types or launching instances over time\.

A spread placement group can span multiple Availability Zones, and you can have a maximum of seven running instances per Availability Zone per group\.

If you start or launch an instance in a spread placement group and there is insufficient unique hardware to fulfill the request, the request fails\. Amazon EC2 makes more distinct hardware available over time, so you can try your request again later\.

## Placement Group Rules and Limitations<a name="concepts-placement-groups"></a>

Before you use placement groups, be aware of the following rules:
+ The name you specify for a placement group must be unique within your AWS account for the region\.
+ You can't merge placement groups\.
+ An instance can be launched in one placement group at a time; it cannot span multiple placement groups\.
+ Reserved Instances provide a capacity reservation for EC2 instances in a specific Availability Zone\. The capacity reservation can be used by instances in a placement group\. However, it is not possible to explicitly reserve capacity for a placement group\.
+ Instances with a tenancy of `host` cannot be launched in placement groups\.

The following rules apply to cluster placement groups:
+ The following are the only instance types that you can use when you launch an instance into a cluster placement group:
  + General purpose: M4, M5
  + Compute optimized: C3, C4, C5, C5d, `cc2.8xlarge`
  + Memory optimized: `cr1.8xlarge`, R3, R4, R5, R5d, X1, X1e, z1d
  + Storage optimized: D2, H1, `hs1.8xlarge`, I2, I3, `i3.metal`
  + Accelerated computing: F1, G2, G3, P2, P3
+ A cluster placement group can't span multiple Availability Zones\.
+ The maximum network throughput speed of traffic between two instances in a cluster placement group is limited by the slower of the two instances\. For applications with high\-throughput requirements, choose an instance type with 10–Gbps or 25–Gbps network connectivity\.
+ For instances that are enabled for enhanced networking, the following rules apply:
  + Traffic between instances within the same region that is addressed using private IPv4 or IPv6 addresses can use 5 Gbps for single\-flow traffic and up to 25 Gbps for multi\-flow traffic\. A flow represents a single, point\-to\-point network connection\.
  + Instances within a cluster placement group can use up to 10 Gbps for single\-flow traffic\.
  + Traffic to and from Amazon S3 buckets within the same region over the public IP address space or through a VPC endpoint can use all available instance aggregate bandwidth\.
+ You can launch multiple instance types into a cluster placement group\. However, this reduces the likelihood that the required capacity will be available for your launch to succeed\. We recommend using the same instance type for all instances in a cluster placement group\.
+ Network traffic to the internet and over an AWS Direct Connect connection to on\-premises resources is limited to 5 Gbps\.

The following rules apply to spread placement groups:
+ A spread placement group supports a maximum of seven running instances per Availability Zone\. For example, in a region with three Availability Zones, you can run a total of 21 instances in the group \(seven per zone\)\. If you try to start an eighth instance in the same zone and in the same spread placement group, the instance will not launch\. If you need to have more than seven instances in an AZ, then the recommendation is to use multiple spread placement groups\. This does not provide guarantees about the spread of instances between groups, but does ensure the spread for each group to limit impact from certain classes of failures\. 
+ Spread placement groups are not supported for Dedicated Instances or Dedicated Hosts\.

## Creating a Placement Group<a name="create-placement-group"></a>

You can create a placement group using the Amazon EC2 console or the command line\.

**To create a placement group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Placement Groups**, **Create Placement Group**\.

1. Specify a name for the group and choose the strategy\.

1. Choose **Create**\.

**To create a placement group using the command line**
+ [create\-placement\-group](http://docs.aws.amazon.com/cli/latest/reference/ec2/create-placement-group.html) \(AWS CLI\)
+ [New\-EC2PlacementGroup](http://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2PlacementGroup.html) \(AWS Tools for Windows PowerShell\)

## Launching Instances in a Placement Group<a name="launch-instance-placement-group"></a>

You can create an AMI specifically for the instances to be launched in a placement group\. To do this, launch an instance and install the required software and applications on the instance\. Then, create an AMI from the instance\. For more information, see [Creating a Custom Windows AMI](Creating_EBSbacked_WinAMI.md)\.<a name="launch_cluster_instance"></a>

**To launch instances into a placement group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Choose **Launch Instance**\. Complete the wizard as directed, taking care to do the following:
   + On the **Choose an Amazon Machine Image \(AMI\)** page, select an AMI\. To select an AMI you created, choose **My AMIs**\.
   + On the **Choose an Instance Type** page, select an instance type that can be launched into a placement group\.
   + On the **Configure Instance Details** page, enter the total number of instances that you need in this placement group, as you might not be able to add instances to the placement group later\.
   + On the **Configure Instance Details** page, select the placement group that you created from **Placement group**\. If you do not see the **Placement group** list on this page, verify that you have selected an instance type that can be launched into a placement group, as this option is not available otherwise\.

**To launch instances into a placement group using the command line**

1. Create an AMI for your instances using one of the following commands:
   + [create\-image](http://docs.aws.amazon.com/cli/latest/reference/ec2/create-image.html) \(AWS CLI\)
   + [New\-EC2Image](http://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2Image.html) \(AWS Tools for Windows PowerShell\)

1. Launch instances into your placement group using one of the following options:
   + `--placement` with [run\-instances](http://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html) \(AWS CLI\)
   + `-PlacementGroup` with [New\-EC2Instance](http://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2Instance.html) \(AWS Tools for Windows PowerShell\)

## Changing the Placement Group for an Instance<a name="change-instance-placement-group"></a>

You can move an existing instance to a placement group, move an instance from one placement group to another, or remove an instance from a placement group\. Before you begin, the instance must be in the `stopped` state\.

You can change the placement group for an instance using the command line or an AWS SDK\.

**To move an instance to a placement group using the command line**

1. Stop the instance using one of the following commands:
   + [stop\-instances](http://docs.aws.amazon.com/cli/latest/reference/ec2/stop-instances.html) \(AWS CLI\)
   + [Stop\-EC2Instance](http://docs.aws.amazon.com/powershell/latest/reference/items/Stop-EC2Instance.html) \(AWS Tools for Windows PowerShell\)

1. Use the [modify\-instance\-placement](http://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-placement.html) command \(AWS CLI\) and specify the name of the placement group to which to move the instance\.

   ```
   aws ec2 modify-instance-placement --instance-id i-0aa51192b00939a40 --group-name MySpreadGroup
   ```

   Alternatively, use the [Edit\-EC2InstancePlacement](http://docs.aws.amazon.com/powershell/latest/reference/items/Edit-EC2InstancePlacement.html) command \(AWS Tools for Windows PowerShell\)\.

1. Restart the instance using one of the following commands:
   + [start\-instances](http://docs.aws.amazon.com/cli/latest/reference/ec2/start-instances.html) \(AWS CLI\)
   + [Start\-EC2Instance](http://docs.aws.amazon.com/powershell/latest/reference/items/Start-EC2Instance.html) \(AWS Tools for Windows PowerShell\)

**To remove an instance from a placement group using the command line**

1. Stop the instance using one of the following commands:
   + [stop\-instances](http://docs.aws.amazon.com/cli/latest/reference/ec2/stop-instances.html) \(AWS CLI\)
   + [Stop\-EC2Instance](http://docs.aws.amazon.com/powershell/latest/reference/items/Stop-EC2Instance.html) \(AWS Tools for Windows PowerShell\)

1. Use the [modify\-instance\-placement](http://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-placement.html) command \(AWS CLI\) and specify an empty string for the group name\.

   ```
   aws ec2 modify-instance-placement --instance-id i-0aa51192b00939a40 --group-name ""
   ```

   Alternatively, use the [Edit\-EC2InstancePlacement](http://docs.aws.amazon.com/powershell/latest/reference/items/Edit-EC2InstancePlacement.html) command \(AWS Tools for Windows PowerShell\)\.

1. Restart the instance using one of the following commands:
   + [start\-instances](http://docs.aws.amazon.com/cli/latest/reference/ec2/start-instances.html) \(AWS CLI\)
   + [Start\-EC2Instance](http://docs.aws.amazon.com/powershell/latest/reference/items/Start-EC2Instance.html) \(AWS Tools for Windows PowerShell\)

## Deleting a Placement Group<a name="delete-placement-group"></a>

If you need to replace a placement group or no longer need one, you can delete it\. Before you can delete your placement group, you must terminate all instances that you launched into the placement group, or move them to another placement group\.

**To terminate or move instances and delete a placement group using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**\.

1. Select and terminate all instances in the placement group\. You can verify that the instance is in a placement group before you terminate it by checking the value of **Placement Group** in the details pane\.

   Alternatively, follow the steps in [Changing the Placement Group for an Instance](#change-instance-placement-group) to move the instances to a different placement group\.

1. In the navigation pane, choose **Placement Groups**\.

1. Select the placement group and choose **Delete Placement Group**\.

1. When prompted for confirmation, choose **Delete**\.

**To terminate instances and delete a placement group using the command line**

You can use one of the following sets of commands\. For more information about these command line interfaces, see [Accessing Amazon EC2](concepts.md#access-ec2)\.
+ [terminate\-instances](http://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html) and [delete\-placement\-group](http://docs.aws.amazon.com/cli/latest/reference/ec2/delete-placement-group.html) \(AWS CLI\)
+ [Remove\-EC2Instance](http://docs.aws.amazon.com/powershell/latest/reference/items/Remove-EC2Instance.html) and [Remove\-EC2PlacementGroup](http://docs.aws.amazon.com/powershell/latest/reference/items/Remove-EC2PlacementGroup.html) \(AWS Tools for Windows PowerShell\)