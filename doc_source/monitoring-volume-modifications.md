# Monitoring the Progress of Volume Modifications<a name="monitoring-volume-modifications"></a>

When you modify an EBS volume, it goes through a sequence of states\. The volume enters the `modifying` state, the `optimizing` state, and finally the `completed` state\. At this point, the volume is ready to be further modified\. Rarely, a transient AWS fault can result in the `failed` state\. If this occurs, retry the volume modification\.

While the volume is in the `optimizing` state, your volume performance is in between the source and target configuration specifications\. Transitional volume performance will be no less than the source volume performance\. If you are downgrading IOPS, transitional volume performance is no less than the target volume performance\.

Volume modification changes take effect as follows:
+ Size changes usually take a few seconds to complete and take effect after a volume is in the `Optimizing` state\.
+ Performance \(IOPS\) changes can take from a few minutes to a few hours to complete and are dependent on the configuration change being made\.
+ It may take up to 24 hours for a new configuration to take effect, and in some cases more, such as when the volume has not been fully initialized\. Typically, a fully used 1\-TiB volume takes about 6 hours to migrate to a new performance configuration\. 

Use one of the following methods to monitor the progress of a volume modification\.

**Topics**
+ [Monitoring the Progress of a Volume Modification \(Console\)](#console_monitoring)
+ [Monitoring the Progress of a Volume Modification \(AWS CLI\)](#api_cli_monitoring)
+ [Monitoring the Progress of a Volume Modification \(CloudWatch Events\)](#cwe_monitoring)

## Monitoring the Progress of a Volume Modification \(Console\)<a name="console_monitoring"></a>

Use the following procedure to view the progress of one or more volume modifications\.

**To monitor progress of a modification using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Volumes**\.

1. Select the volume\. The volume status is displayed in the **State** column and in the **State** field of the details pane\. In this example, the modification state is **completed**\.

1. Open the information icon next to the **State** field to display before and after information about the most recent modification action, as shown in this example\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/monitor_modifications.png)

## Monitoring the Progress of a Volume Modification \(AWS CLI\)<a name="api_cli_monitoring"></a>

Use the [describe\-volumes\-modifications](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-volumes-modifications.html) command to view the progress of one or more volume modifications\. The following example describes the volume modifications for two volumes\.

```
aws ec2 describe-volumes-modifications --volume-id vol-11111111111111111 vol-22222222222222222
```

In the following example output, the volume modifications are still in the `modifying` state\.

```
{
    "VolumesModifications": [
        {
            "TargetSize": 200,
            "TargetVolumeType": "io1",
            "ModificationState": "modifying",
            "VolumeId": "vol-11111111111111111",
            "TargetIops": 10000,
            "StartTime": "2017-01-19T22:21:02.959Z",
            "Progress": 0,
            "OriginalVolumeType": "gp2",
            "OriginalIops": 300,
            "OriginalSize": 100
        },
        {
            "TargetSize": 2000,
            "TargetVolumeType": "sc1",
            "ModificationState": "modifying",
            "VolumeId": "vol-22222222222222222",
            "StartTime": "2017-01-19T22:23:22.158Z",
            "Progress": 0,
            "OriginalVolumeType": "gp2",
            "OriginalIops": 300,
            "OriginalSize": 1000
        }
    ]
}
```

The next example describes all volumes with a modification state of either `optimizing` or `completed`, and then filters and formats the results to show only modifications that were initiated on or after February 1, 2017:

```
aws ec2 describe-volumes-modifications --filters Name=modification-state,Values="optimizing","completed" --query "VolumesModifications[?StartTime>='2017-02-01'].{ID:VolumeId,STATE:ModificationState}"
```

The following is example output with information about two volumes:

```
[
    {
        "STATE": "optimizing",
        "ID": "vol-06397e7a0eEXAMPLE"
    },
    {
        "STATE": "completed",
        "ID": "vol-ba74e18c2aEXAMPLE"
    }
]
```

## Monitoring the Progress of a Volume Modification \(CloudWatch Events\)<a name="cwe_monitoring"></a>

With CloudWatch Events, you can create a notification rule for volume modification events\. You can use your rule to generate a notification message using [Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/) or to invoke a [Lambda function](https://docs.aws.amazon.com/lambda/latest/dg/) in response to matching events\.

**To monitor progress of a modification using CloudWatch Events**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. Choose **Events**, **Create rule**\.

1. For **Build event pattern to match events by service**, choose **Custom event pattern**\.

1. For **Build custom event pattern**, replace the contents with the following and choose **Save**\.

   ```
   {
     "source": [
       "aws.ec2"
     ],
     "detail-type": [
       "EBS Volume Notification"
     ],
     "detail": {
       "event": [
         "modifyVolume"
       ]
     }
   }
   ```

   The following is an example of a typical event:

   ```
   Body:
   {
      "version": "0",
      "id": "1ea2ace2-7790-46ed-99ab-d07a8bd68685",
      "detail-type": "EBS Volume Notification",
      "source": "aws.ec2",
      "account": "065441870323",
      "time": "2017-01-12T21:09:07Z",
      "region": "us-east-1",
      "resources": [
         "arn:aws:ec2:us-east-1:065441870323:volume/vol-03a55cf56513fa1b6"
      ],
      "detail": {
         "result": "optimizing",
         "cause": "",
         "event": "modifyVolume",
         "request-id": "auto-58c08bad-d90b-11e6-a309-b51ed35473f8"
      }
   }
   ```