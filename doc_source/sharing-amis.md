# Shared AMIs<a name="sharing-amis"></a>

*A shared AMI* is an AMI that a developer created and made available for other developers to use\. One of the easiest ways to get started with Amazon EC2 is to use a shared AMI that has the components you need and then add custom content\. You can also create your own AMIs and share them with others\. 

You use a shared AMI at your own risk\. Amazon can't vouch for the integrity or security of AMIs shared by other Amazon EC2 users\. Therefore, you should treat shared AMIs as you would any foreign code that you might consider deploying in your own data center and perform the appropriate due diligence\. We recommend that you get an AMI from a trusted source\.

Amazon's public images have an aliased owner, which appears as `amazon` in the account field\. This enables you to find AMIs from Amazon easily\. Other users can't alias their AMIs\.

 For information about creating an AMI, see [Creating an Amazon EBS\-Backed Windows AMI](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/Creating_EBSbacked_WinAMI.html) \. For more information about building, delivering, and maintaining your applications on the AWS Marketplace, see the [AWS Marketplace User Guide](https://docs.aws.amazon.com/marketplace/latest/controlling-access/what-is-marketplace.html) and [AWS Marketplace Seller Guide](http://awsmp-loadforms.s3.amazonaws.com/AWS_Marketplace_-_Seller_Guide.pdf)\.

**Topics**
+ [Finding Shared AMIs](usingsharedamis-finding.md)
+ [Making an AMI Public](sharingamis-intro.md)
+ [Sharing an AMI with Specific AWS Accounts](sharingamis-explicit.md)
+ [Using Bookmarks](using-bookmarks.md)
+ [Guidelines for Shared Windows AMIs](windows-amis-guidelines.md)