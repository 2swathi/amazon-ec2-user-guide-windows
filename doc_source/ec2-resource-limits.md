# Amazon EC2 Service Limits<a name="ec2-resource-limits"></a>

Amazon EC2 provides different *resources* that you can use\. These resources include images, instances, volumes, and snapshots\. When you create your AWS account, we set default limits on these resources on a per\-Region basis\. For example, there is a limit on the number of instances that you can launch in a Region\. Therefore, when you launch an instance in the US West \(Oregon\) Region, the request must not cause your usage to exceed your current instance limit in that Region\.

The Amazon EC2 console provides limit information for the resources managed by the Amazon EC2 and Amazon VPC consoles\. You can request an increase for many of these limits\. Use the limit information that we provide to manage your AWS infrastructure\. Plan to request any limit increases in advance of the time that you'll need them\.

For more information about the limits for other services, see [AWS Service Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html) in the *Amazon Web Services General Reference*\.

## Viewing Your Current Limits<a name="view-limits"></a>

Use the **EC2 Service Limits** page in the Amazon EC2 console to view the current limits for resources provided by Amazon EC2 and Amazon VPC, on a per\-Region basis\.

**To view your current limits**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. From the navigation bar, select a Region\.  
![\[Select a Region\]](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/images/EC2_select_region.png)

1. From the navigation pane, choose **Limits**\.

1. Locate the resource in the list\. The **Current Limit** column displays the current maximum for that resource for your account\.

## Requesting a Limit Increase<a name="request-increase"></a>

Use the **Limits** page in the Amazon EC2 console to request an increase in the limits for resources provided by Amazon EC2 or Amazon VPC, on a per\-Region basis\.

**To request a limit increase**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. From the navigation bar, select a Region\.

1. From the navigation pane, choose **Limits**\.

1. Locate the resource in the list\. Choose **Request limit increase**\.

1. Complete the required fields on the limit increase form\. We'll respond to you using the contact method that you specified\.

## Limits on Email Sent Using Port 25<a name="port-25-throttle"></a>

Amazon EC2 throttles traffic on port 25 of all instances by default\. You can request that this throttle be removed\. For more information, see [How do I remove the throttle on port 25 from my EC2 instance?](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-port-25-throttle/) in the AWS Knowledge Center\.