# Hibernation prerequisites<a name="hibernating-prerequisites"></a>

**Topics**
+ [Supported Windows AMIs](#hibernation-prereqs-supported-amis)
+ [Supported instance families](#hibernation-prereqs-supported-instance-families)
+ [Instance size](#hibernation-prereqs-instance-size)
+ [Instance RAM size](#instance-ram-size)
+ [Root volume type](#hibernation-prereqs-root-volume-type)
+ [EBS root volume size](#hibernation-prereqs-ebs-root-volume-size)
+ [Supported EBS volume types](#hibernation-prereqs-ebs-volume-types)
+ [EBS root volume encryption](#hibernation-prereqs-ebs-root-volume-encryption)
+ [Enable hibernation at launch](#hibernation-prereqs-enable-at-launch)
+ [Purchasing options](#hibernation-prereqs-purchasing-options)

## Supported Windows AMIs<a name="hibernation-prereqs-supported-amis"></a>

Must be an HVM AMI that supports hibernation:
+ Windows Server 2012 AMI released 2019\.09\.11 or later
+ Windows Server 2012 R2 AMI released 2019\.09\.11 or later
+ Windows Server 2016 AMI released 2019\.09\.11 or later
+ Windows Server 2019 AMI released 2019\.09\.11 or later

For information about the supported Linux AMIs, see [Supported Linux AMIs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hibernating-prerequisites.html#hibernation-prereqs-supported-amis) in the *Amazon EC2 User Guide for Linux Instances*\.

## Supported instance families<a name="hibernation-prereqs-supported-instance-families"></a>
+ Xen: C3, C4, I3, M3, M4, R3, R4, T2
+ Nitro: C5, C5d, M5, M5a, M5ad, M5d, R5, R5a, R5ad, R5d, T3\*, T3a\*

  \* For hibernation, we recommend that you use a T3 or T3a instance with at least 1 GB of RAM\.

**To see the available instance types that support hibernation in a specific Region**  
The available instance types vary by Region\. To see the available instance types that support hibernation in a Region, use the [describe\-instance\-types](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instance-types.html) command with the `--region` parameter\. Include the `--filters` parameter to see only the instance types that support hibernation\.

```
C:\> aws ec2 describe-instance-types \
--region us-east-2 \
--filters Name=hibernation-supported,Values=true \
--query "InstanceTypes[*].[InstanceType]" \
--output table
```

Example output

```
-----------------------
|DescribeInstanceTypes|
+---------------------+
|  r5a.xlarge         |
|  c4.4xlarge         |
|  m5ad.large         |
|  c5.4xlarge         |
|  m4.4xlarge         |
|  t3.2xlarge         |
...
```

## Instance size<a name="hibernation-prereqs-instance-size"></a>

Not supported for bare metal instances\.

## Instance RAM size<a name="instance-ram-size"></a>

Can be up to 16 GB\.

## Root volume type<a name="hibernation-prereqs-root-volume-type"></a>

Must be an EBS volume, not an instance store volume\.

## EBS root volume size<a name="hibernation-prereqs-ebs-root-volume-size"></a>

Must be large enough to store the RAM contents and accommodate your expected usage, for example, OS or applications\. If you enable hibernation, space is allocated on the root volume at launch to store the RAM\.

## Supported EBS volume types<a name="hibernation-prereqs-ebs-volume-types"></a>
+ General Purpose SSD \(`gp2` and `gp3`\)
+ Provisioned IOPS SSD \(`io1` and `io2`\)

If you choose a Provisioned IOPS SSD volume type, you must provision the EBS volume with the appropriate IOPS to achieve optimum performance for hibernation\. For more information, see [Amazon EBS volume types](ebs-volume-types.md)\.

## EBS root volume encryption<a name="hibernation-prereqs-ebs-root-volume-encryption"></a>

To use hibernation, the root volume must be encrypted to ensure the protection of sensitive content that is in memory at the time of hibernation\. When RAM data is moved to the EBS root volume, it is always encrypted\. Encryption of the root volume is enforced at instance launch\.

Use one of the following three options to ensure that the root volume is an encrypted EBS volume:
+ **EBS encryption by default** – You can enable EBS encryption by default to ensure that all new EBS volumes created in your AWS account are encrypted\. This way, you can enable hibernation for your instances without specifying encryption intent at instance launch\. For more information, see [Encryption by default](EBSEncryption.md#encryption-by-default)\.
+ **EBS "single\-step" encryption** – You can launch encrypted EBS\-backed EC2 instances from an unencrypted AMI and also enable hibernation at the same time\. For more information, see [Use encryption with EBS\-backed AMIs](AMIEncryption.md)\.
+ **Encrypted AMI** – You can enable EBS encryption by using an encrypted AMI to launch your instance\. If your AMI does not have an encrypted root snapshot, you can copy it to a new AMI and request encryption\. For more information, see [Encrypt an unencrypted image during copy](AMIEncryption.md#copy-unencrypted-to-encrypted) and [Copy an AMI](CopyingAMIs.md#ami-copy-steps)\.

## Enable hibernation at launch<a name="hibernation-prereqs-enable-at-launch"></a>

You cannot enable hibernation on an existing instance \(running or stopped\)\. For more information, see [Enable hibernation for an instance](enabling-hibernation.md)\.

## Purchasing options<a name="hibernation-prereqs-purchasing-options"></a>

This feature is available for On\-Demand Instances and Reserved Instances\. It is not available for Spot Instances\. For information about hibernating a Spot Instance, see [Hibernate interrupted Spot Instances](hibernate-spot-instances.md)\.