# Change the instance type of an Amazon EBS–backed instance<a name="resize-ebs-backed-instance"></a>

## Considerations<a name="resize-ebs-backed-instance-considerations"></a>

You must stop your Amazon EBS–backed instance before you can change its instance type\. When you stop and start an instance, be aware of the following:
+ We move the instance to new hardware; however, the instance ID does not change\.
+ If your instance has a public IPv4 address, we release the address and give it a new public IPv4 address\. The instance retains its private IPv4 addresses, any Elastic IP addresses, and any IPv6 addresses\.
+ When you resize an instance, the resized instance usually has the same number of instance store volumes that you specified when you launched the original instance\. With instance types that support NVMe instance store volumes \(which are available by default\), the resized instance might have additional instance store volumes, depending on the AMI\. Otherwise, you can migrate your application to an instance with a new instance type manually, specifying the number of instance store volumes that you need when you launch the new instance\.
+ If your instance is in an Auto Scaling group, the Amazon EC2 Auto Scaling service marks the stopped instance as unhealthy, and may terminate it and launch a replacement instance\. To prevent this, you can suspend the scaling processes for the group while you're resizing your instance\. For more information, see [Suspending and Resuming Scaling Processes](https://docs.aws.amazon.com/autoscaling/latest/userguide/as-suspend-resume-processes.html) in the *Amazon EC2 Auto Scaling User Guide*\.
+ If your instance is in a [cluster placement group](placement-groups.md#placement-groups-cluster) and, after changing the instance type, the instance start fails, try the following: stop all the instances in the cluster placement group, change the instance type for the affected instance, and then restart all the instances in the cluster placement group\. 
+ Ensure that you plan for downtime while your instance is stopped\. Stopping and resizing an instance may take a few minutes, and restarting your instance may take a variable amount of time depending on your application's startup scripts\.

For more information, see [Stop and start your instance](Stop_Start.md)\.

## Change the instance type<a name="resize-ebs-backed-instance-considerations-how-to"></a>

Use the following procedure to change the instance type of an Amazon EBS–backed instance using the AWS Management Console\.

------
#### [ New console ]

**To change the instance type of an Amazon EBS–backed instance**

1. \(Optional\) If the new instance type requires drivers that are not installed on the existing instance, you must connect to your instance and install the drivers first\. For more information, see [Compatibility for changing the instance type](resize-limitations.md)\.
**Note**  
The AWS PV driver package should be updated before changing instance families\. For more information, see [Upgrade PV drivers on Windows instances](Upgrading_PV_drivers.md)\.

1. \(Optional\) If you configured your Windows instance to use [static IP addressing](config-windows-multiple-ip.md#step1) and you resize the instance from a type that doesn't support enhanced networking to an instance type that does support enhanced networking, you might get a warning about a potential IP address conflict when you reconfigure static IP addressing\. To prevent this, enable DHCP on the network interface for your instance before you change the instance type\. From your instance, open the **Network and Sharing Center**, go to **Internet Protocol Version 4 \(TCP/IPv4\) Properties** for the network interface, and choose **Obtain an IP address automatically**\. Change the instance type and reconfigure static IP addressing on the network interface\.

1. Open the Amazon EC2 console\.

1. \[Windows Server 2016 and later\] Connect to your Windows instance and run the following EC2Launch PowerShell script to configure the instance after it is resized\.

   ```
   PS C:\> C:\ProgramData\Amazon\EC2-Windows\Launch\Scripts\InitializeInstance.ps1 -Schedule
   ```

1. In the navigation pane, choose **Instances**\.

1. Select the instance and choose **Actions**, **Instance state**, **Stop instance**\.

1. In the confirmation dialog box, choose **Stop**\. It can take a few minutes for the instance to stop\.

1. With the instance still selected, choose **Actions**, **Instance settings**, **Change instance type**\. This action is grayed out if the instance state is not `stopped`\.

1. In the **Change instance type** dialog box, do the following:

   1. From **Instance type**, select the instance type that you want\. If the instance type that you want does not appear in the list, then it is not compatible with the configuration of your instance \(for example, because of virtualization type\)\. For more information, see [Compatibility for changing the instance type](resize-limitations.md)\.

   1. \(Optional\) If the instance type that you selected supports EBS–optimization, select **EBS\-optimized** to enable EBS–optimization or deselect **EBS\-optimized** to disable EBS–optimization\. If the instance type that you selected is EBS–optimized by default, **EBS\-optimized** is selected and you can't deselect it\.

   1. Choose **Apply** to accept the new settings\.

1. To restart the stopped instance, select the instance and choose **Instance state**, **Start instance**\. It can take a few minutes for the instance to enter the `running` state\.

------
#### [ Old console ]

**To change the instance type of an Amazon EBS–backed instance**

1. \(Optional\) If the new instance type requires drivers that are not installed on the existing instance, you must connect to your instance and install the drivers first\. For more information, see [Compatibility for changing the instance type](resize-limitations.md)\.
**Note**  
The AWS PV driver package should be updated before changing instance families\. For more information, see [Upgrade PV drivers on Windows instances](Upgrading_PV_drivers.md)\.

1. \(Optional\) If you configured your Windows instance to use [static IP addressing](config-windows-multiple-ip.md#step1) and you resize the instance from a type that doesn't support enhanced networking to an instance type that does support enhanced networking, you might get a warning about a potential IP address conflict when you reconfigure static IP addressing\. To prevent this, enable DHCP on the network interface for your instance before you change the instance type\. From your instance, open the **Network and Sharing Center**, go to **Internet Protocol Version 4 \(TCP/IPv4\) Properties** for the network interface, and choose **Obtain an IP address automatically**\. Change the instance type and reconfigure static IP addressing on the network interface\.

1. Open the Amazon EC2 console\.

1. \[Windows Server 2016 and later\] Connect to your Windows instance and run the following EC2Launch PowerShell script to configure the instance after it is resized\.

   ```
   PS C:\> C:\ProgramData\Amazon\EC2-Windows\Launch\Scripts\InitializeInstance.ps1 -Schedule
   ```

1. In the navigation pane, choose **Instances**\.

1. Select the instance and choose **Actions**, **Instance State**, **Stop**\.

1. In the confirmation dialog box, choose **Yes, Stop**\. It can take a few minutes for the instance to stop\.

1. With the instance still selected, choose **Actions**, **Instance Settings**, **Change Instance Type**\. This action is grayed out if the instance state is not `stopped`\.

1. In the **Change Instance Type** dialog box, do the following:

   1. From **Instance Type**, select the instance type that you want\. If the instance type that you want does not appear in the list, then it is not compatible with the configuration of your instance \(for example, because of virtualization type\)\. For more information, see [Compatibility for changing the instance type](resize-limitations.md)\.

   1. \(Optional\) If the instance type that you selected supports EBS–optimization, select **EBS\-optimized** to enable EBS–optimization or deselect **EBS\-optimized** to disable EBS–optimization\. If the instance type that you selected is EBS–optimized by default, **EBS\-optimized** is selected and you can't deselect it\.

   1. Choose **Apply** to accept the new settings\.

1. To restart the stopped instance, select the instance and choose **Actions**, **Instance State**, **Start**\.

1. In the confirmation dialog box, choose **Yes, Start**\. It can take a few minutes for the instance to enter the `running` state\.

------