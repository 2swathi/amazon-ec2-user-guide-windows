# Work with retention rules<a name="recycle-bin-working-with-rules"></a>

To enable and use Recycle Bin, you must create *retention rules* in the AWS Regions in which you want to protect snapshots\. Retention rules specify the following:
+ The snapshots that you want to retain in the Recycle Bin when they are deleted\.
+ The retention period for which to retain snapshots in the Recycle Bin after deletion\.

With Recycle Bin, you can create two types of retention rules:
+ **Tag\-level retention rules** — These retention rules use resource tags to identify the snapshots that are to be retained in the Recycle Bin\. For each retention rule, you specify one or more tag key and value pairs\. Snapshots that are tagged with at least one of the tag key and value pairs that are specified in the retention rule are automatically retained in the Recycle Bin upon deletion\. Use this type of retention rule if you want to protect specific snapshots in your account based on their tags\.
+ **Region\-level retention rules** — These retention rules do not have any resource tags specified\. They apply to all of the snapshots in the Region in which they are created, even if the snapshots are not tagged\. Use this type of retention rule if you want to protect all of your snapshots in a specific Region\.

After you create a retention rule, snapshots that match its criteria are automatically retained in the Recycle Bin for the specified period when they are deleted\.

**Topics**
+ [Create a retention rule](#recycle-bin-create-rule)
+ [View Recycle Bin retention rules](#recycle-bin-view-rule)
+ [Update retention rules](#recycle-bin-update-rule)
+ [Delete Recycle Bin retention rules](#recycle-bin-delete-rule)

## Create a retention rule<a name="recycle-bin-create-rule"></a>

To create a retention rule, you must specify:
+ An optional name for the retention rule\. The name can be up to 255 characters long\.
+ An optional description for the rule\. The description can be up to 255 characters long\.
+ Resource tags that identify the snapshots that are to be retained in the Recycle Bin\. You can specify up to 50 tags for each rule\. However, you can add the same tag key and value pair to up to 5 retention rules only\.

  To create a tag\-level retention rule, specify at least one tag key and value pair\. To create an Region\-level retention rule, do not specify any tag key and value pairs\.
+ The period for which the snapshots are to be retained in the Recycle Bin\. The period can be up to 1 year \(365 days\)\.
+ Optional retention rule tags to help identify and organize your retention rules\. You can assign up to 50 tags to each rule\.

Retention rules function only in the Regions in which they are created\. If you intend to use Recycle Bin in other Regions, you must create additional retention rules in those Regions\.

You can create a Recycle Bin retention rule using one of the following methods\.

------
#### [ Recycle Bin console ]

**To create a retention rule**

1. Open the Recycle Bin console at [https://console\.aws\.amazon\.com/rbin/home/](https://console.aws.amazon.com/rbin/home/)

1. In the navigation panel, choose **Retention rules** and then choose **Create retention rule**\.

1. In the **Rule details** section, do the following:

   1. \(*Optional*\) For **Retention rule name**, enter a descriptive name for the retention rule\.

   1. \(*Optional*\) For **Retention rule description**, enter a brief description for the retention rule\.

1. In the **Rule settings** section, do the following:

   1. For **Resource type**, select **EBS snapshots**\.

   1. Do one of the following:
      + To create a Region\-level retention rule that matches all deleted snapshots in the Region, select **Apply to all resources**\. The retention rule will retain all deleted snapshots in the Recycle Bin upon deletion, even if the snapshots do not have any tags\.
      + To create a tag\-level retention rule, for **Resource tags to match**, enter the tag key and value pairs to use to identify snapshots that are to be retained in the Recycle Bin\. Only snapshots that have at least one of the specified tag key and value pairs will be retained by the retention rule\.

   1. For **Retention period**, enter the number of days for which the retention rule is to retain snapshots in the Recycle Bin\.

1. \(*Optional*\) In the **Tags** section, do the following:

   1. To tag the rule with custom tags, choose **Add tag** and then enter the tag key and value pair\.

1. Choose **Create retention rule**\.

------
#### [ AWS CLI ]

**To create a retention rule**  
Use the [create\-rule](https://docs.aws.amazon.com/cli/latest/reference/rbin/create-rule.html) AWS CLI command\. For `--retention-period`, specify the number of days to retain deleted snapshots in the Recycle Bin\. For `--resource-type`, specify `EBS_SNAPSHOT`\. To create a tag\-level retention rule, for `--resource-tags`, specify the tags to use to identify the snapshots that are to be retained\. To create a Region\-level retention rule, omit `--resource-tags`\.

```
C:\> aws rbin create-rule \
--retention-period RetentionPeriodValue=number_of_days,RetentionPeriodUnit=DAYS \
--resource-type EBS_SNAPSHOT \
--description "rule_description" \
--resource-tags ResourceTagKey=tag_key,ResourceTagValue=tag_value
```

**Example 1**  
The following example command creates a Region\-level retention rule that retains all deleted snapshots for a period of `8` days\.

```
C:\> aws rbin create-rule \
--retention-period RetentionPeriodValue=8,RetentionPeriodUnit=DAYS \
--resource-type EBS_SNAPSHOT \
--description "Match all snapshots"
```

**Example 2**  
The following example command creates a tag\-level rule that retains deleted snapshots that are tagged with `purpose=production` for a period of `14` days\.

```
C:\> aws rbin create-rule \
--retention-period RetentionPeriodValue=14,RetentionPeriodUnit=DAYS \
--resource-type EBS_SNAPSHOT \
--description "Match snapshots with a specific tag" \
--resource-tags ResourceTagKey=purpose,ResourceTagValue=production
```

------

## View Recycle Bin retention rules<a name="recycle-bin-view-rule"></a>

You can view Recycle Bin retention rules using one of the following methods\.

------
#### [ Recycle Bin console ]

**To view retention rules**

1. Open the Recycle Bin console at [https://console\.aws\.amazon\.com/rbin/home/](https://console.aws.amazon.com/rbin/home/)

1. In the navigation panel, choose **Retention rules**\.

1. The grid lists all of the retention rules for the selected Region\. To view more information about a specific retention rule, select it in the grid\.

------
#### [ AWS CLI ]

**To view all of your retention rules**  
Use the [list\-rules](https://docs.aws.amazon.com/cli/latest/reference/rbin/list-rules.html) AWS CLI command, and for `--resource-type`, specify `EBS_SNAPSHOT`\.

```
C:\> aws rbin list-rules --resource-type EBS_SNAPSHOT
```

**To view information for a specific retention rule**  
Use the [get\-rule](https://docs.aws.amazon.com/cli/latest/reference/rbin/get-rule.html) AWS CLI command\.

```
C:\> aws rbin get-rule --identifier rule_ID
```

**Example**  
The following example command provides information about retention rule `pwxIkFcvge4`\.

```
C:\> aws rbin get-rule --identifier pwxIkFcvge4
```

------

## Update retention rules<a name="recycle-bin-update-rule"></a>

You can update a retention rule at any time after creation\. You can modify all of a retention rule's parameters, including its description, retention period, and resource tags\.

After you update a retention rule, the changes only apply to new snapshots that it retains\. The changes do not affect snapshots that it previously sent to the Recycle Bin\. For example, if you update a retention rule's retention period, only new snapshots that it retains from that point are retained for the new retention period\. Snapshots that it sent to the Recycle Bin before the update are still retained for the previous \(old\) retention period\.

You can update a retention rule using one of the following methods\.

------
#### [ Recycle Bin console ]

**To update a retention rule**

1. Open the Recycle Bin console at [https://console\.aws\.amazon\.com/rbin/home/](https://console.aws.amazon.com/rbin/home/)

1. In the navigation panel, choose **Retention rules**\.

1. In the grid, select the retention rule to update, and choose **Actions**, **Edit retention rule**\.

1. In the **Rule details** section, update **Retention rule name** and **Retention rule description** as needed\.

1. In the **Rule settings** section, update the **Resource tags to match** and **Retention period** as needed\.

1. In the **Tags** section, add or remove retention rule tags as needed\.

1. Choose **Save retention rule**\.

------
#### [ AWS CLI ]

**To update a retention rule**  
Use the [update\-rule](https://docs.aws.amazon.com/cli/latest/reference/rbin/update-rule.html) AWS CLI command\. For `--identifier`, specify the ID of the retention rule to update\.

```
C:\> aws rbin update-rule \
--identifier rule_ID \
--retention-period RetentionPeriodValue=number_of_days,RetentionPeriodUnit=DAYS \
--resource-type EBS_SNAPSHOT \
--description "rule_description"
```

**Example**  
The following example command updates retention rule `6lsJ2Fa9nh9` to retain all snapshots for `21` days and updates its description\.

```
C:\> aws rbin update-rule \
--identifier 6lsJ2Fa9nh9 \
--retention-period RetentionPeriodValue=21,RetentionPeriodUnit=DAYS \
--resource-type EBS_SNAPSHOT \
--description "Retain for three weeks"
```

------

## Delete Recycle Bin retention rules<a name="recycle-bin-delete-rule"></a>

You can delete a retention rule at any time\. When you delete a retention rule, it no longer retains new snapshots after they have been deleted\. Snapshots that were sent to the Recycle Bin before the retention rule was deleted continue to be retained in the Recycle Bin according to the retention period defined in the retention rule\. When the period expires, the snapshot is permanently deleted from the Recycle Bin\.

You can delete a retention rule using one of the following methods\.

------
#### [ Recycle Bin console ]

**To delete a retention rule**

1. Open the Recycle Bin console at [https://console\.aws\.amazon\.com/rbin/home/](https://console.aws.amazon.com/rbin/home/)

1. In the navigation panel, choose **Retention rules**\.

1. In the grid, select the retention rule to delete and choose **Actions**, **Delete retention rule**\.

1. When prompted, enter the confirmation message and choose **Delete retention rule**\.

------
#### [ AWS CLI ]

**To delete a retention rule**  
Use the [delete\-rule](https://docs.aws.amazon.com/cli/latest/reference/rbin/delete-rule.html) AWS CLI command\. For `--identifier`, specify the ID of the retention rule to delete\.

```
C:\> aws rbin delete-rule --identifier rule_ID
```

**Example**  
The following example command deletes retention rule `6lsJ2Fa9nh9`\.

```
C:\> aws rbin delete-rule --identifier 6lsJ2Fa9nh9
```

------