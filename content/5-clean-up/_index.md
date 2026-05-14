---
title : "Clean up resources"
date: 2024-01-01
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

**Resource Cleanup**

To clean up the AWS resources created in the practice, we will follow the steps below in turn.

**Content**
- [Run shell script ```cleanup.sh```](#run-shell-script-cleanupsh)
- [Delete Secret at Secret Manager](#delete-secret-at-secret-manager)
- [Delete CloudFormation Stack](#delete-cloudformation-stack)

#### Run shell script ```cleanup.sh```
1. Access the service [Systems Manager](https://us-east-1.console.aws.amazon.com/systems-manager)
2. In the left-hand navigation bar, select **Session Manager**.
3. Select the EC2 instance corresponding to the **BastionIP** value.
4. Press the `Start Session` button.
5. In the new window bar, we will proceed to run the necessary commands.

{{% notice note %}}
Before running the scripts, we need to make sure the account used is `ec2-user`
{{% /notice %}}.

Proceed to switch to `ec2-user` account with the following command:

```bash
sudo su - ec2-user
```

![shell-cleanup](/images/5-shell-cleanup.png)

Perform ECS Cluster and Amazon ECR cleanup with the following shell script:

```bash
./cleanup.sh
```

#### Delete Secret at Secret Manager
1. Access the service [Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager).
2. Delete the *Secret* created in the previous section.

#### Delete CloudFormation Stack
1. Access the service [CloudFormation](https://us-east-1.console.aws.amazon.com/cloudformation/home)
2. Search for a Stack named with the following format `-RDSMySQLRotationSingleUser-`. This is the Stack used for **Key Rotation**
3. Proceed to delete CloudFormation Stack above.
4. After the process is complete, we will proceed to delete the Stack built in the preparation environment.
5. Search for the Stack named with the following format `smdemo`.
6. Proceed to delete CloudFormation Stack above.

![cloudformation-stacks](/images/5-cloudformation-stacks.png)