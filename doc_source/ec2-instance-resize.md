# Changing the Instance Type<a name="ec2-instance-resize"></a>

As your needs change, you might find that your instance is over\-utilized \(the instance type is too small\) or under\-utilized \(the instance type is too large\)\. If this is the case, you can change the size of your instance\. For example, if your `t2.micro` instance is too small for its workload, you can change it to another instance type that is appropriate for the workload\.

You might also want to migrate from a previous generation instance type to a current generation instance type to take advantage of some features; for example, support for IPv6\.

You can change the size of the instance simply by changing its instance type, which is known as *resizing* it\.

When you resize an instance, you must select an instance type that is compatible with the configuration of the instance\. If the instance type that you want is not compatible with the instance configuration you have, then you must migrate your application to a new instance with the instance type that you need\.

**Important**  
When you resize an instance, the resized instance usually has the same number of instance store volumes that you specified when you launched the original instance\. If you want to add instance store volumes, you must migrate your application to a new instance with the instance type and instance store volumes that you need\. An exception to this rule is when you resize to a storage\-optimized instance type that by default contains a higher number of volumes\. For more information about instance store volumes, see [Amazon EC2 Instance Store](InstanceStorage.md)\.

**Topics**
+ [Compatibility for Resizing Instances](#resize-limitations)
+ [Resizing an Amazon EBS–backed Instance](#resize-ebs-backed-instance)
+ [Migrating to a New Instance Configuration](#migrate-instance-configuration)

## Compatibility for Resizing Instances<a name="resize-limitations"></a>

You can resize an instance only if its current instance type and the new instance type that you want are compatible in the following ways:
+ **Network**: Some instance types are not supported in EC2\-Classic and must be launched in a VPC\. Therefore, you can't resize an instance in EC2\-Classic to a instance type that is available only in a VPC unless you have a nondefault VPC\. For more information, see [Instance Types Available Only in a VPC](using-vpc.md#vpc-only-instance-types)\. To check if your instance is in a VPC, check the **VPC ID** value on the details pane of the **Instances** screen in the Amazon EC2 console\.
+ **Platform**: All Amazon EC2 instance types support 64\-bit AMIs, but only the following instance types support 32\-bit AMIs: `t2.nano`, `t2.micro`, `t2.small`, `t2.medium`, `c3.large`, `t1.micro`, `m1.small`, `m1.medium`, and `c1.medium`\. If you are resizing a 32\-bit instance, you are limited to these instance types\. To check the platform of your instance, go to the **Instances** screen in the Amazon EC2 console and choose **Show/Hide Columns**, **Architecture**\.
+ **Enhanced networking**: Instance types that support [enhanced networking](enhanced-networking.md) require the necessary drivers installed\. For example, the C5, C5d, M5, M5d, R5, R5d, T3, and z1d instance types require EBS\-backed AMIs with the Elastic Network Adapter \(ENA\) drivers installed\. To resize an existing instance to an instance type that supports enhanced networking, you must first install the [ENA drivers](enhanced-networking-ena.md) or [ixgbevf drivers](sriov-networking.md) on your instance, as appropriate\.
+ **NVMe**: Instance types that support NVMe, such as C5, C5d, M5, M5d, R5, R5d, T3, and z1d, expose EBS volumes as NVMe block devices\. To resize an existing instance to an instance type that supports NVMe, you must first install the [NVMe drivers](nvme-ebs-volumes.md) on your instance\. Also, the device names for devices that you specify in the block device mapping are renamed using NVMe device names \(`/dev/nvme[0-26]n1`\)\.
+ **AMI**: For information about the AMIs required by instance types that support enhanced networking and NVMe, see the Release Notes in the following documentation:
  + [General Purpose Instances](general-purpose-instances.md)
  + [Compute Optimized Instances](compute-optimized-instances.md)
  + [Memory Optimized Instances](memory-optimized-instances.md)
  + [Storage Optimized Instances](storage-optimized-instances.md)

## Resizing an Amazon EBS–backed Instance<a name="resize-ebs-backed-instance"></a>

You must stop your Amazon EBS–backed instance before you can change its instance type\. When you stop and start an instance, be aware of the following:
+ We move the instance to new hardware; however, the instance ID does not change\.
+ If your instance is running in a VPC and has a public IPv4 address, we release the address and give it a new public IPv4 address\. The instance retains its private IPv4 addresses, any Elastic IP addresses, and any IPv6 addresses\.
+ If your instance is running in EC2\-Classic, we give it new public and private IP addresses, and disassociate any Elastic IP address that's associated with the instance\. Therefore, to ensure that your users can continue to use the applications that you're hosting on your instance uninterrupted, you must re\-associate any Elastic IP address after you restart your instance\.
+ If your instance is in an Auto Scaling group, the Amazon EC2 Auto Scaling service marks the stopped instance as unhealthy, and may terminate it and launch a replacement instance\. To prevent this, you can suspend the scaling processes for the group while you're resizing your instance\. For more information, see [Suspending and Resuming Scaling Processes](https://docs.aws.amazon.com/autoscaling/latest/userguide/as-suspend-resume-processes.html) in the *Amazon EC2 Auto Scaling User Guide*\.
+ Ensure that you plan for downtime while your instance is stopped\. Stopping and resizing an instance may take a few minutes, and restarting your instance may take a variable amount of time depending on your application's startup scripts\.

For more information, see [Stop and Start Your Instance](Stop_Start.md)\.

Use the following procedure to resize an Amazon EBS–backed instance using the AWS Management Console\.

**To resize an Amazon EBS–backed instance**

1. \(Optional\) If the new instance type requires drivers that are not installed on the existing instance, you must connect to your instance and install the drivers first\. For more information, see [Compatibility for Resizing Instances](#resize-limitations)\.
**Note**  
The AWS PV driver package should be updated before changing instance families\. For more information, see [Upgrading PV Drivers on Your Windows Instances](Upgrading_PV_drivers.md)\.

1. Open the Amazon EC2 console\.

1. \[Windows Server 2016\] Connect to your Windows instance and run the following EC2Launch PowerShell script to configure the instance after it is resized\.

   ```
   PS C:\> C:\ProgramData\Amazon\EC2-Windows\Launch\Scripts\InitializeInstance.ps1 -Schedule
   ```

1. In the navigation pane, choose **Instances**, and select the instance\.

1. \[EC2\-Classic\] If the instance has an associated Elastic IP address, write down the Elastic IP address and the instance ID shown in the details pane\.

1. Choose **Actions**, select **Instance State**, and then choose **Stop**\.

1. In the confirmation dialog box, choose **Yes, Stop**\. It can take a few minutes for the instance to stop\.

   \[EC2\-Classic\] When the instance state becomes `stopped`, the **Elastic IP**, **Public DNS \(IPv4\)**, **Private DNS**, and **Private IPs** fields in the details pane are blank to indicate that the old values are no longer associated with the instance\.

1. With the instance still selected, choose **Actions**, select **Instance Settings**, and then choose **Change Instance Type**\. Note that this action is disabled if the instance state is not `stopped`\.

1. In the **Change Instance Type** dialog box, do the following:

   1. From **Instance Type**, select the instance type that you want\. If the instance type that you want does not appear in the list, then it is not compatible with the configuration of your instance \(for example, because of virtualization type\)\.

   1. \(Optional\) If the instance type that you selected supports EBS–optimization, select **EBS\-optimized** to enable EBS–optimization or deselect **EBS\-optimized** to disable EBS–optimization\. Note that if the instance type that you selected is EBS–optimized by default, **EBS\-optimized** is selected and you can't deselect it\.

   1. Choose **Apply** to accept the new settings\.

1. To restart the stopped instance, select the instance, choose **Actions**, select **Instance State**, and then choose **Start**\.

1. In the confirmation dialog box, choose **Yes, Start**\. It can take a few minutes for the instance to enter the `running` state\.

1. \[EC2\-Classic\] When the instance state is `running`, the **Public DNS \(IPv4\)**, **Private DNS**, and **Private IPs** fields in the details pane contain the new values that we assigned to the instance\. If your instance had an associated Elastic IP address, you must reassociate it as follows:

   1. In the navigation pane, choose **Elastic IPs**\.

   1. Select the Elastic IP address that you wrote down before you stopped the instance\.

   1. Choose **Actions** and then choose **Associate address**\.

   1. From **Instance**, select the instance ID that you wrote down before you stopped the instance, and then choose **Associate**\.

## Migrating to a New Instance Configuration<a name="migrate-instance-configuration"></a>

If the current configuration of your instance is incompatible with the new instance type that you want, then you can't resize the instance to that instance type\. Instead, you can migrate your application to a new instance with a configuration that is compatible with the new instance type that you want\.

**To migrate your application to a compatible instance**

1. Back up any data on your instance store volumes that you need to keep to persistent storage\. To migrate data on your EBS volumes that you need to keep, create a snapshot of the volumes \(see [Creating an Amazon EBS Snapshot](ebs-creating-snapshot.md)\) or detach the volume from the instance so that you can attach it to the new instance later \(see [Detaching an Amazon EBS Volume from an Instance](ebs-detaching-volume.md)\)\.

1. Launch a new instance, selecting the following:
   + \[EC2\-VPC\] If you are using an Elastic IP address, select the VPC that the original instance is currently running in\.
   + Any EBS volumes that you detached from the original instance and want to attach to the new instance, or new EBS volumes based on the snapshots that you created\.
   + If you want to allow the same traffic to reach the new instance, select the security group that is associated with the original instance\.

1. Install your application and any required software on the instance\.

1. Restore any data that you backed up from the instance store volumes of the original instance\.

1. If you are using an Elastic IP address, assign it to the newly launched instance as follows:

   1. In the navigation pane, choose **Elastic IPs**\.

   1. Select the Elastic IP address that is associated with the original instance, choose **Actions**, and then choose **Disassociate address**\. When prompted for confirmation, choose **Disassociate address**\.

   1. With the Elastic IP address still selected, choose **Actions**, and then choose **Associate address**\.

   1. From **Instance**, select the new instance, and then choose **Associate**\.

1. \(Optional\) You can terminate the original instance if it's no longer needed\. Select the instance and verify that you are about to terminate the original instance, not the new instance \(for example, check the name or launch time\)\. Choose **Actions**, select **Instance State**, and then choose **Terminate**\.

For information about migrating an application from an instance in EC2\-Classic to an instance in a VPC, see [Migrating from a Windows Instance in EC2\-Classic to a Windows Instance in a VPC](vpc-migrate.md)\.