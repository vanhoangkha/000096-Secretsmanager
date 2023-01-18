---
title : "Preparation"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

**Prepare resources with AWS CloudFormation**

To conduct the infrastructure deployment, we will use the **AWS CloudFormation** service through the AWS Console or AWS CLI.

**Content**
- [CloudFormation Stack](#cloudformation-stack)
- [Template URL](#template-url)
- [Parameters](#parameters)
- [Stack Outputs](#stack-outputs)

#### CloudFormation Stack
{{% notice note %}}
The exercise will be set at **us-east-1 (North Virginia)**.
{{% /notice %}}

1. Sign in to AWS Console, access the [CloudFormation service](https://us-east-1.console.aws.amazon.com/cloudformation/home)
2. Proceed to create a new Stack by clicking the `Create Stack` button.
3. On the `Specify template` page, we have 2 ways:
   1. Enter the path of **Template URL** in the `Amazon S3 URL` box.
   2. Upload an existing Template using the `Upload a template file` button.
4. On the `Parameters` page, we will enter some required information.

![cloudformation-create-stack-parameters](/images/2-cloudformation-create-stack-parameters.png?featherlight=false&width=90pc)

5. On the `Specify Stack Details` page, select the `Next` button.
6. On the `Configure stack options` page, select the `Next` button.
7. On the `Capabilities` page, proceed to accept (Acknowledge) to allow the Template to create IAM roles, and finally select the `Create Stack` button.

![cloudformation-stack-status](/images/2-cloudformation-stack-status.png?featherlight=false&width=90pc)

{{% notice info %}}
The above process will take 5-10 minutes until we see the status of the Stack as `CREATE_COMPLETE`.
{{% /notice %}}

#### Template URL
| Ingredients | Value (Required) |
| ---------- | ------- |
| Stack Name | smdemo |
| Template URL | [RDSFargate.yml](https://s3.amazonaws.com/sa-security-specialist-workshops-us-east-1/secrets-manager-workshop/RDSFargate.yml) |

Or download the template below:

{{%attachments title="Template" pattern=".*(yml)"/%}}

#### Parameters
{{% notice info %}}
The values ​​below have been defined as default values ​​at Template.
{{% /notice %}}

| Ingredients | Value (Required) |
| ---------- | ------- |
| NamePrefix | smdemo |
| ProjectTag | smproj |
| DBName | smdemo |
| DBPort | 3306 |
| AmazonLinux2AmiId | /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 |

#### Stack Outputs
After the CloudFormation Stack has been prepared, we proceed to note all the values ​​in the *Outputs* section.

The following **Output Keys** will be used:
| Output Key | Definition |
| ---------- | ---------- |
| BastionIP | IP Address of *Bastion Host* |
| DBInstance | ID of RDS Instance |
| DBUser | Master User of RDS database |
| DBPassword | Master User's Password of RDS database |
| EC2UserPassword | Password of `ec2-user` |
| ECRRepository | ECR repository |
| ECSCluster | ECS cluster |

![cloudformation-stack-output](/images/2-cloudformation-stack-output.png?featherlight=false&width=90pc)

{{% notice tip %}}
We need to note and note the above values ​​because they will be used for the next sections.
{{% /notice %}}