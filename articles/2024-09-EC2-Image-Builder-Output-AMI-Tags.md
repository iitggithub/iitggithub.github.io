## EC2 Image Builder - Adding Output AMI Tags to Shared AMI's

With EC2 Image Builder, you can specify AMI Tags in your Distribution configuration which are automatically added to your output AMIs and configure cross-account AMI distribution to deliver AMIs to specified target accounts, organizations, and OUs in the destination Region.

If you choose to to utilize AMI sharing in your Distribution configuration share output AMIs with specific target accounts, organizations, and OUs in the destination Region, the AMI Tags in your Distribution configuration are not shared along with the AMI. Image Builder uses the EC2 to facilitate AMI sharing which does not support sharing tags. For more information see [Share an AMI with specific AWS accounts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sharingamis-explicit.html#considerations-for-sharing-AMI-with-accounts) in the [Amazon Elastic Compute Cloud User Guide](https://docs.aws.amazon.com/ec2/).

This article will walk you through creating the necessarily IAM role in the target AWS accounts, as well as deploying a Cloudformation stack which utilises the Amazon Simple Notification Service (SNS) to trigger a Lambda function that adds AMI Tags to shared AMIs.

### Prerequisites

1. Intermediate understanding of IAM permissions for [EC2 Image Builder cross-account distribution](https://docs.aws.amazon.com/imagebuilder/latest/userguide/cross-account-dist.html).
2. Intermediate understanding of AWS CloudFormation, and EC2 Image Builder.
3. Basic understanding of network CIDR notation and addresses.
4. A pre-existing EC2 Image Builder Image Pipeline with AMI Sharing enabled, and Output AMI Distribution Tags configured.

### Deployment Walkthrough

#### Step 1: Create the EC2ImageBuilderDistributionCrossAccountRole IAM Role in each of the target accounts

The EC2ImageBuilderDistributionCrossAccountRole is required in each of the target accounts. The Lambda function will attempt to assume this role and use it to add the EC2 Image Builder distribution tags to the newly shared AMI.

To configure cross-account distribution permissions in AWS Identity and Access Management (IAM), follow these steps:

1. Open the AWS IAM console at <https://console.aws.amazon.com/iam/home#/roles>.
2. Choose **Create Role**.
3. For Trusted entity type, choose **Custom trust policy**.
4. Replace the example Custom trust policy with the trust policy below.

> **Note**
> Replace 111111111111 with the AWS account ID of the account that your Lambda function is in.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::111111111111:role/EC2ImageBuilderLambdaExecutionRole"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    }

5. To accept the trust policy, choose Next, and proceed with adding permissions to the IAM role.
6. In the Permissions policies list, select the **Ec2ImageBuilderCrossAccountDistributionAccess** AWS Managed IAM policy.
7. To accept the Permissions policies, choose Next, and proceed with naming the IAM role and reviewing its configuration.
8. For the Role name, enter the name **EC2ImageBuilderDistributionCrossAccountRole**.
9. Choose **Create role** to create the EC2ImageBuilderDistributionCrossAccountRole IAM role.

#### Step 2: Deploy the AWS CloudFormation stack in the EC2 Image Builder Source Account

> **Note**
> You need to deploy the AWS CloudFormation stack in the same AWS region as the EC2 Image Builder Pipeline. This is because the SNS Topic that's created is AWS Region-specific.

**Download the CloudFormation stack template**

You can download the stack template from [https://raw.githubusercontent.com/iitggithub/aws/master/EC2ImageBuilderAddTagsToSharedAMIs.json](https://raw.githubusercontent.com/iitggithub/aws/master/EC2ImageBuilderAddTagsToSharedAMIs.json).


**To create a stack on the CloudFormation console**

1. Open the AWS CloudFormation console at <https://console.aws.amazon.com/cloudformation>.
2. Create a new stack by using one of the following options:
* Choose **Create Stack**. This is the only option if you have a currently running stack.
* Choose **Create Stack** on the **Stacks** page. This option is visible only if you have no running stacks.
3. Choose **Upload a template file**
4. Choose **Choose file** and select the **EC2ImageBuilderAddTagsToSharedAMIs.json** file and then choose **Open**.

> **Note**
> When you upload a local template file, CloudFormation uploads it to an Amazon Simple Storage Service (Amazon S3) bucket in your AWS account. If you don't already have an S3 bucket that was created by CloudFormation, it creates a unique bucket for each Region in which you upload a template file. If you already have an S3 bucket that was created by AWS CloudFormation in your AWS account, CloudFormation adds the template to that bucket.
> Considerations to keep in mind about S3 buckets created by CloudFormation
> The buckets are accessible to anyone with Amazon S3 permissions in your AWS account.
> * CloudFormation creates the buckets with server-side encryption enabled by default, thereby encrypting all objects stored in the bucket.
> * You can directly manage encryption options for buckets that CloudFormation has created; for example, using the Amazon S3 console at <https://console.aws.amazon.com/s3/>, or the AWS CLI. For more information, see [Amazon S3 default encryption for S3 buckets](https://docs.aws.amazon.com/AmazonS3/latest/dev/bucket-encryption.html) in the [Amazon Simple Storage Service User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html).
> * You can use your own bucket and manage its permissions by manually uploading templates to Amazon S3. When you create or update a stack, specify the Amazon S3 URL of a template file.

5. To accept your settings, choose Next, and proceed with specifying the stack name and parameters.
6. For the Stack name, enter a name for your CloudFormation stack.
7. Under **VpcAvailabilityZones**, choose two Availability Zones.

> **Note**
> The CloudFormation stack creates a new VPC that by default occupies the 10.0.0.0/24 CIDR range. If you are already using this range for an existing VPC, you will need to update the VpcCidr, SubnetCidr1, SubnetCidr2, SubnetCidr3, and SubnetCidr4 parameters accordingly.

8. When you are satisfied with the parameter values, choose Next to proceed with setting options for your stack.
9. Choose Next.
10. Choose **I acknowledge that AWS CloudFormation might create IAM resources with customised names.**
11. Choose **Submit** to begin deploying the CloudFormation stack.

#### Step 3: Set the SNS Topic in the EC2 Image Builder Infrastructure Configuration

To update an infrastructure configuration resource from the Image Builder console, follow these steps:

**Choose an existing Image Builder infrastructure configuration**

1. Open the EC2 Image Builder console at <https://console.aws.amazon.com/imagebuilder/>.
2. To see a list of the infrastructure configuration resources under your account, choose **Infrastructure configuration** from the navigation pane.
3. To view details or edit an infrastructure configuration, choose the **Configuration name** link. This opens the detail view for the infrastructure configuration.

> **Note**
> You can also select the check box next to the **Configuration name**, then choose **View detail**.

4. From the upper right corner of the **Infrastructure details** panel, choose **Edit** .
5. In the  **AWS infrastructure ** section, under  **SNS Topic**, choose the **EC2ImageBuilderPipelineStatus** SNS Topic.
6. When you're ready to save updates you've made to your infrastructure configuration, choose **Save changes**.

#### Step 4: Execute an EC2 Image Builder Image Pipeline and verify functionality

1. Execute your Image Pipeline and wait for the pipeline to complete.
2. In the target account, open the AWS EC2 console at  <https://console.aws.amazon.com/ec2/home#Images:visibility=private>.
3. verifying that the newly created AMI has been shared with the target account
4. Choose the newly created AMI from the list of private AMIs.
5. Choose **Tags** and verify the Output AMI Distribution tag key/value pairs have been applied to the shared AMI in the target account.

### Frequently Asked Questions

* I've configured my distribution settings to copy the AMI to a target account and it's not working but AMI sharing is able to create tag key/value pairs in target accounts.

The EC2ImageBuilderDistributionCrossAccountRole IAM Role trust policy specified in this article is quite restrictive and only allows the Lambda function in the EC2 Image Builder source account to assume the EC2ImageBuilderDistributionCrossAccountRole IAM role in the target account. You can modify the trust policy and make it less restrictive to allow both the Lambda function, and EC2 Image Builder to assume the role. An example trust policy is shown below.

> **Note**
> Replace 111111111111 with the AWS account ID of the account that your Lambda function is in.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::111111111111:root"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    }

* I have EC2 Image Builder pipelines in other AWS regions. Do I need to deploy the CloudFormation stack in each region?

You will need to create an SNS Topic in the other AWS regions as SNS Topics are region-specific. Once you've created the SNS Topic, you can subscribe your existing Lambda function to it. For more information about creating SNS Topics, see [Creating an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-create-topic.html).

### Conclusion

In this article, we deployed an AWS CloudFormation stack that uses AWS Lambda to add EC2 Image Builder output AMI distribution tag key/value pairs to shared AMIs. We also created an IAM role in target accounts which the Lambda function can assume.
