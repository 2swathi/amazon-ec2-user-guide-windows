# Recover snapshots from the Recycle Bin<a name="recycle-bin-working-with-snaps"></a>

Recycle Bin for Amazon EBS snapshots is a snapshot recovery feature that enables you to restore accidentally deleted snapshots\. When using Recycle Bin, if your snapshots are deleted, they are retained in the Recycle Bin for a time period that you specify\.

You can restore a snapshot from the Recycle Bin at any time before its retention period expires\. After you restore a snapshot from the Recycle Bin, the snapshot is removed from the Recycle Bin and you can use it in the same way you use any other snapshot in your account\. If the retention period expires and the snapshot is not restored, the snapshot is permanently deleted from the Recycle Bin and is no longer available for recovery\.

For more information, see [Recycle Bin for Amazon EBS snapshots](recycle-bin.md)\.

**Topics**
+ [View snapshots in the Recycle Bin](#recycle-bin-view-snaps)
+ [Restore snapshots from the Recycle Bin](#recycle-bin-restore-snaps)

## View snapshots in the Recycle Bin<a name="recycle-bin-view-snaps"></a>

While a snapshot is in the Recycle Bin, you can view limited information about it, including:
+ The ID of the snapshot\.
+ The snapshot description\.
+ The ID of the volume from which the snapshot was created\.
+ The date and time when the snapshot was deleted and it entered Recycle Bin\.
+ The date and time when the retention period expires\. The snapshot will be permanently deleted from the Recycle Bin at this time\.

You can view the snapshots in the Recycle Bin using one of the following methods\.

------
#### [ Recycle Bin console ]

**To view snapshots in the Recycle Bin using the console**

1. Open the Recycle Bin console at [https://console\.aws\.amazon\.com//rbin/home/](https://console.aws.amazon.com//rbin/home/)

1. In the navigation panel, choose **Recycle Bin**\.

1. The grid lists all of the snapshots that are currently in the Recycle Bin\. To view the details for a specific snapshot, select it in the grid and choose **Actions**, **View details**\.

------
#### [ AWS CLI ]

**To view snapshots in the Recycle Bin using the AWS CLI**  
Use the [ list\-snapshots\-in\-recycle\-bin](https://docs.aws.amazon.com/cli/latest/reference/ec2/list-snapshots-in-recycle-bin.html) AWS CLI command\. Include the `--snapshot-id` option to view a specific snapshot\. Or omit the `--snapshot-id` option to view all snapshots in the Recycle Bin\.

```
C:\> aws ec2 list-snapshots-in-recycle-bin --snapshot-id snapshot_id
```

For example, the following command provides information about snapshot `snap-01234567890abcdef` in the Recycle Bin\.

```
C:\> aws ec2 list-snapshots-in-recycle-bin --snapshot-id snap-01234567890abcdef
```

Example output:

```
{
    "SnapshotRecycleBinInfo": [
        {
            "Description": "Monthly data backup snapshot",
            "RecycleBinEnterTime": "2021-12-01T13:00:00.000Z",
            "RecycleBinExitTime": "2021-12-15T13:00:00.000Z",
            "VolumeId": "vol-abcdef09876543210",
            "SnapshotId": "snap-01234567890abcdef"
        }
    ]
}
```

------

## Restore snapshots from the Recycle Bin<a name="recycle-bin-restore-snaps"></a>

You can't use a snapshot in any way while it is in the Recycle Bin\. To use the snapshot, you must first restore it\. When you restore a snapshot from the Recycle Bin, the snapshot is immediately available for use, and it is removed from the Recycle Bin\. You can use a restored snapshot in the same way that you use any other snapshot in your account\.



You can restore a snapshot from the Recycle Bin using one of the following methods\.

------
#### [ Recycle Bin console ]

**To restore a snapshot from the Recycle Bin using the console**

1. Open the Recycle Bin console at [https://console\.aws\.amazon\.com//rbin/home/](https://console.aws.amazon.com//rbin/home/)

1. In the navigation panel, choose **Recycle Bin**\.

1. The grid lists all of the snapshots that are currently in the Recycle Bin\. Select the snapshot to restore and and choose **Recover**\.

1. When prompted, choose **Recover**\.

------
#### [ AWS CLI ]

**To restore a deleted snapshot from the Recycle Bin using the AWS CLI**  
Use the [ restore\-snapshot\-from\-recycle\-bin](https://docs.aws.amazon.com/cli/latest/reference/ec2/restore-snapshot-from-recycle-bin.html) AWS CLI command\. For `--snapshot-id`, specify the ID of the snapshot to restore\.

```
C:\> aws ec2 restore-snapshot-from-recycle-bin --snapshot-id snapshot_id
```

For example, the following command restores snapshot `snap-01234567890abcdef` from the Recycle Bin\.

```
C:\> aws restore-snapshot-from-recycle-bin --snapshot-id snap-01234567890abcdef
```

The command returns no output on success\.

------