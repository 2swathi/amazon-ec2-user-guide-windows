# Amazon Data Lifecycle Manager<a name="snapshot-lifecycle"></a>

You can use Amazon Data Lifecycle Manager to automate the creation, retention, and deletion of EBS snapshots and EBS\-backed AMIs\. When you automate snapshot and AMI management, it helps you to:
+ Protect valuable data by enforcing a regular backup schedule\.
+ Create standardized AMIs that can be refreshed at regular intervals\.
+ Retain backups as required by auditors or internal compliance\.
+ Reduce storage costs by deleting outdated backups\.
+ Create disaster recovery backup policies that back up data to isolated accounts\.

When combined with the monitoring features of Amazon CloudWatch Events and AWS CloudTrail, Amazon Data Lifecycle Manager provides a complete backup solution for Amazon EC2 instances and individual EBS volumes at no additional cost\.

**Important**  
Amazon Data Lifecycle Manager cannot be used to manage snapshots or AMIs that are created by any other means\.  
Amazon Data Lifecycle Manager cannot be used to automate the creation, retention, and deletion of instance store\-backed AMIs\.

**Topics**
+ [How Amazon Data Lifecycle Manager works](#dlm-elements)
+ [Considerations for Amazon Data Lifecycle Manager](#dlm-considerations)
+ [Automate snapshot lifecycles](snapshot-ami-policy.md)
+ [Automate AMI lifecycles](ami-policy.md)
+ [Automate cross\-account snapshot copies](event-policy.md)
+ [View, modify, and delete lifecycle policies](view-modify-delete.md)
+ [AWS Identity and Access Management](dlm-prerequisites.md)
+ [Monitor the lifecycle of snapshots and AMIs](dlm-monitor-lifecycle.md)

## How Amazon Data Lifecycle Manager works<a name="dlm-elements"></a>

The following are the key elements of Amazon Data Lifecycle Manager\.

**Topics**
+ [Snapshots](#dlm-ebs-snapshots)
+ [EBS\-backed AMIs](#dlm-ebs-amis)
+ [Target resource tags](#dlm-tagging-volumes)
+ [Amazon Data Lifecycle Manager tags](#dlm-tagging-snapshots)
+ [Lifecycle policies](#dlm-lifecycle-policies)
+ [Policy schedules](#dlm-lifecycle-schedule)

### Snapshots<a name="dlm-ebs-snapshots"></a>

Snapshots are the primary means to back up data from your EBS volumes\. To save storage costs, successive snapshots are incremental, containing only the volume data that changed since the previous snapshot\. When you delete one snapshot in a series of snapshots for a volume, only the data that's unique to that snapshot is removed\. The rest of the captured history of the volume is preserved\.

For more information, see [Amazon EBS snapshots](EBSSnapshots.md)\.

### EBS\-backed AMIs<a name="dlm-ebs-amis"></a>

An Amazon Machine Image \(AMI\) provides the information that's required to launch an instance\. You can launch multiple instances from a single AMI when you need multiple instances with the same configuration\. Amazon Data Lifecycle Manager supports EBS\-backed AMIs only\. EBS\-backed AMIs include a snapshot for each EBS volume that's attached to the source instance\.

For more information, see [Amazon Machine Images \(AMI\)](AMIs.md)\.

### Target resource tags<a name="dlm-tagging-volumes"></a>

Amazon Data Lifecycle Manager uses resource tags to identify the resources to back up\. Tags are customizable metadata that you can assign to your AWS resources \(including Amazon EC2 instances, EBS volumes and snapshots\)\. An Amazon Data Lifecycle Manager policy \(described later\) targets an instance or volume for backup using a single tag\. Multiple tags can be assigned to an instance or volume if you want to run multiple policies on it\.

You can't use a '\\' or '=' character in a tag key\.

For more information, see [Tag your Amazon EC2 resources](Using_Tags.md)\.

### Amazon Data Lifecycle Manager tags<a name="dlm-tagging-snapshots"></a>

Amazon Data Lifecycle Manager applies the following tags to all snapshots and AMIs created by a policy, to distinguish them from snapshots and AMIs created by any other means:
+ `aws:dlm:lifecycle-policy-id`
+ `aws:dlm:lifecycle-schedule-name`
+ `aws:dlm:expirationTime` — For policies with age\-based retention schedules only\.
+ `dlm:managed`

You can also specify custom tags to be applied to snapshots and AMIs on creation\. You can't use a '\\' or '=' character in a tag key\.

The target tags that Amazon Data Lifecycle Manager uses to associate volumes with a snapshot policy can optionally be applied to snapshots created by the policy\. Similarly, the target tags that are used to associate instances with an AMI policy can optionally be applied to AMIs created by the policy\.

### Lifecycle policies<a name="dlm-lifecycle-policies"></a>

A lifecycle policy consists of these core settings:
+ **Policy type**—Defines the type of resources that the policy can manage\. Amazon Data Lifecycle Manager supports the following types of lifecycle policies:
  + Snapshot lifecycle policy—Used to automate the lifecycle of EBS snapshots\. These policies can target individual EBS volumes or all EBS volumes attached to an instance\.
  + EBS\-backed AMI lifecycle policy—Used to automate the lifecycle of EBS\-backed AMIs and their backing snapshots\. These policies can target instances only\.
  + Cross\-account copy event policy—Used to automate snapshot copies across accounts\. Use this policy type in conjunction with an EBS snapshot policy that shares snapshots across accounts\.
+ **Resource type**—Defines the type of resources that are targeted by the policy\. Snapshot lifecycle policies can target instances or volumes\. Use `VOLUME` to create snapshots of individual volumes, or use `INSTANCE` to create multi\-volume snapshots of all of the volumes that are attached to an instance\. For more information, see [Multi\-volume snapshots](ebs-creating-snapshot.md#ebs-create-snapshot-multi-volume)\. AMI lifecycle policies can target instances only\. One AMI is created that includes snapshots of all of the volumes that are attached to the target instance\. 
+ **Target tags**—Specifies the tags that must be assigned to an EBS volume or an Amazon EC2 instance for it to be targeted by the policy\.
+ **Schedules**—The start times and intervals for creating snapshots or AMIs\. The first snapshot or AMI creation operation starts within one hour after the specified start time\. Subsequent snapshot or AMI creation operations start within one hour of their scheduled time\. A policy can have up to four schedules: one mandatory schedule, and up to three optional schedules\. For more information, see [Policy schedules](#dlm-lifecycle-schedule)\. 
+ **Retention**—Specifies how snapshots or AMIs are to be retained\. You can retain snapshots or AMIs based either on their total count \(count\-based\), or their age \(age\-based\)\. For snapshot policies, when the retention threshold is reached, the oldest snapshot is deleted\. For AMI policies, when the retention threshold is reached, the oldest AMI is deregistered and its backing snapshots are deleted\. 

For example, you could create a policy with settings similar to the following:
+ Manages all EBS volumes that have a tag with a key of `account` and a value of `finance`\.
+ Creates snapshots every `24` hours at `0900 UTC`\.
+ Retains only the five most recent snapshots\.
+ Starts snapshot creation no later than `0959` UTC each day\.

### Policy schedules<a name="dlm-lifecycle-schedule"></a>

Policy schedules define when snapshots or AMIs are created by the policy\. Policies can have up to four schedules—one mandatory schedule, and up to three optional schedules\.

Adding multiple schedules to a single policy lets you create snapshots or AMIs at different frequencies using the same policy\. For example, you can create a single policy that creates daily, weekly, monthly, and yearly snapshots\. This eliminates the need to manage multiple policies\.

For each schedule, you can define the frequency, fast snapshot restore settings \(snapshot lifecycle policies only\), cross\-Region copy rules, and tags\. The tags that are assigned to a schedule are automatically assigned to the snapshots or AMIs that are created when the schedule is initiated\. In addition, Amazon Data Lifecycle Manager automatically assigns a system\-generated tag based on the schedule's frequency to each snapshot or AMI\.

Each schedule is initiated individually based on its frequency\. If multiple schedules are initiated at the same time, Amazon Data Lifecycle Manager creates only one snapshot or AMI and applies the retention settings of the schedule that has the highest retention period\. The tags of all of the initiated schedules are applied to the snapshot or AMI\.
+ \(Snapshot lifecycle policies only\) If more than one of the initiated schedules is enabled for fast snapshot restore, then the snapshot is enabled for fast snapshot restore in all of the Availability Zones specified across all of the initiated schedules\. The highest retention settings of the initiated schedules is used for each Availability Zone\.
+ If more than one of the initiated schedules is enabled for cross\-Region copy, the snapshot or AMI is copied to all Regions specified across all of the initiated schedules\. The highest retention period of the initiated schedules is applied\.

## Considerations for Amazon Data Lifecycle Manager<a name="dlm-considerations"></a>

Your AWS account has the following quotas related to Amazon Data Lifecycle Manager:
+ You can create up to 100 lifecycle policies per Region\.
+ You can add up to 45 tags per resource\.

The following considerations apply to lifecycle policies:
+ A policy does not begin creating snapshots or AMIs until you set its activation status to *enabled*\. You can configure a policy to be enabled upon creation\.
+ The first snapshot or AMI creation operation starts within one hour after the specified start time\. Subsequent snapshot or AMI creation operations start within one hour of their scheduled time\.
+ If you modify a policy by removing or changing its target tags, the EBS volumes or instances with those tags are no longer managed by the policy\.
+ If you modify a schedule name for a policy, the snapshots or AMIs created under the old schedule name are no longer affected by the policy\.
+ If you modify a time\-based retention schedule to use a new time interval, the new interval is used only for new snapshots or AMIs created after the change\. The new schedule does not affect the retention schedule of snapshots or AMIs created before the change\.
+ You cannot change the retention schedule of a policy from count\-based to time\-based after creation\. To make this change, you must create a new policy\.
+ If you disable a policy with an age\-based retention schedule, the snapshots or AMIs that are set to expire while the policy is disabled are retained indefinitely\. You must delete the snapshots or deregister the AMIs manually\. When you enable the policy again, Amazon Data Lifecycle Manager resumes deleting snapshots or deregistering AMIs as their retention periods expire\.
+ If you delete the resource to which a policy with count\-based retention applies, the policy no longer manages the previously created snapshots or AMIs\. You must manually delete the snapshots or deregister the AMIs if they are no longer needed\.
+ If you delete the resource to which a policy with age\-based retention applies, the policy continues to delete snapshots or deregister AMIs on the defined schedule, up to, but not including, the last snapshot or AMI\. You must manually delete the last snapshot or deregister the last AMI if it is no longer needed\.
+ You can create multiple policies to back up an EBS volume or an Amazon EC2 instance\. For example, if an EBS volume has two tags, where tag *A* is the target for policy *A* to create a snapshot every 12 hours, and tag *B* is the target for policy *B* to create a snapshot every 24 hours, Amazon Data Lifecycle Manager creates snapshots according to the schedules for both policies\. Alternatively, you can achieve the same result by creating a single policy that has multiple schedules\. For example, you can create a single policy that targets only tag *A*, and specify two schedules—one for every 12 hours and one for every 24 hours\.
+ If you create a policy that targets instances, and new volumes are attached to the instance after the policy has been created, the newly\-added volumes are included in the backup at the next policy run\. All volumes attached to the instance at the time of the policy run are included\.
+ For AMI lifecycle policies, when the AMI retention threshold is reached, the oldest AMI is deregistered and its backing snapshots are deleted\.
+ If a policy with a custom cron\-based schedule and age\-based or count\-based retention rule is configured to create only one snapshot or AMI, the policy will not automatically delete that snapshot or AMI when the retention threshold is reached\. You must manually delete the snapshot or deregister the AMI if it is no longer needed\.

The following considerations apply to snapshot lifecycle policies and [fast snapshot restore](ebs-fast-snapshot-restore.md):
+ A snapshot that is enabled for fast snapshot restore remains enabled even if you delete or disable the lifecycle policy, disable fast snapshot restore for the lifecycle policy, or disable fast snapshot restore for the Availability Zone\. You can disable fast snapshot restore for these snapshots manually\.
+ If you enable fast snapshot restore and you exceed the maximum number of snapshots that can be enabled for fast snapshot restore, Amazon Data Lifecycle Manager creates snapshots as scheduled but does not enable them for fast snapshot restore\. After a snapshot that is enabled for fast snapshot restore is deleted, the next snapshot that Amazon Data Lifecycle Manager creates is enabled for fast snapshot restore\.
+ When you enable fast snapshot restore for a snapshot, it takes 60 minutes per TiB to optimize the snapshot\. We recommend that you create a schedule that ensures that each snapshot is fully optimized before Amazon Data Lifecycle Manager creates the next snapshot\.
+ You are billed for each minute that fast snapshot restore is enabled for a snapshot in a particular Availability Zone\. Charges are pro\-rated with a minimum of one hour\. For more information, see [Pricing and Billing](ebs-fast-snapshot-restore.md#fsr-pricing)\.
**Note**  
Depending on the configuration of your lifecycle policies, you could have multiple snapshots enabled for fast snapshot restore simultaneously\.

The following considerations apply to sharing snapshots across accounts:
+ You can only share snapshots that are unencrypted or that are encrypted using a customer managed key\.
+ You can't share snapshots that are encrypted with the default EBS encryption KMS key\.
+ If you share encrypted snapshots, then you must also share the KMS key that was used to encrypt the source volume with the target accounts\. For more information, see [Allowing users in other accounts to use a KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying-external-accounts.html) in the *AWS Key Management Service Developer Guide*\.

The following considerations apply to cross\-account copy event policies:
+ You can only copy snapshots that are unencrypted or that are encrypted using a customer managed key\.
+ You can create a cross\-account copy event policy that copies snapshots that are shared outside of Amazon Data Lifecycle Manager\.
+ If you want to encrypt snapshots in the target account, then the IAM role selected for the cross\-account copy event policy must have permission to use the required KMS key\.

The following considerations apply to EBS\-backed AMI policies and AMI deprecation:
+ If you increase the AMI deprecation count for a schedule with count\-based retention, the change is applied to all AMIs \(existing and new\) created by the schedule\.
+ If you increase the AMI deprecation period for a schedule with age\-based retention, the change is applied to new AMIs only\. Existing AMIs are not affected\.
+ If you remove the AMI deprecation rule from a schedule, Amazon Data Lifecycle Manager will not cancel deprecation for AMIs that were previously deprecated by that schedule\.
+ If you decrease the AMI deprecation count or period for a schedule, Amazon Data Lifecycle Manager will not cancel deprecation for AMIs that were previously deprecated by that schedule\.
+ If you manually deprecate an AMI that was created by an AMI policy, Amazon Data Lifecycle Manager will not override the deprecation\.
+ If you manually cancel deprecation for an AMI that was previously deprecated by an AMI policy, Amazon Data Lifecycle Manager will not override the cancellation\.
+ If an AMI is created by multiple conflicting schedules, and one or more of those schedules do not have an AMI deprecation rule, Amazon Data Lifecycle Manager will not deprecate that AMI\.
+ If an AMI is created by multiple conflicting schedules, and all of those schedules have an AMI deprecation rule, Amazon Data Lifecycle Manager will use the deprecation rule with the latest deprecation date\.

The following considerations apply to snapshots policies and snapshot archiving:
+ If you manually archive a snapshot that was created by a policy, and that snapshot is in the archive tier when the policy’s retention threshold is reached, Amazon Data Lifecycle Manager will not delete the snapshot\. Amazon Data Lifecycle Manager does not manage snapshots while they are stored in the archive tier\. If you no longer need snapshots that are stored in the archive tier, you must manually delete them\.

The following considerations apply to snapshot policies and Recycle Bin:
+ If Amazon Data Lifecycle Manager deletes a snapshot and sends it to the Recycle Bin when the policy's retention threshold is reached, and you manually restore the snapshot from the Recycle Bin, you must manually delete that snapshot when it is no longer needed\. Amazon Data Lifecycle Manager will not automatically delete the snapshot\.
+ If you manually delete a snapshot that was created by a policy, and that snapshot is in the Recycle Bin when the policy’s retention threshold is reached, Amazon Data Lifecycle Manager will not delete the snapshot\. Amazon Data Lifecycle Manager does not manage the snapshots while they are stored in the Recycle Bin\.

  If the snapshot is restored from the Recycle Bin before the policy's retention threshold is reached, Amazon Data Lifecycle Manager will delete the snapshot when the policy's retention threshold is reached\.

  If the snapshot is restored from the Recycle Bin after the policy's retention threshold is reached, Amazon Data Lifecycle Manager will no longer delete the snapshot\. You must manually delete the snapshot when it is no longer needed\.