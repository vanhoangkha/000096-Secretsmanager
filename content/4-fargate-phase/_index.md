---
title : "Using on Fargate"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

**Use Secrets Manager to interact with Fargate**

{{% notice note %}}
You need to complete **RDS Phase** before you can continue below.
{{% /notice %}}

#### Overview
In this section, we will learn how to use the **Secrets Manager** service with **AWS Fargate**.

**AWS Fargate** is a `compute engine` used for **Amazon ECS (Elastic Container Service)** service that allows us to instantiate Containers without having to manage to compute or must configure the VMs. This is a step forward to help us reduce the load on deciding which *EC2 Instance Type* is suitable to run our application as well as making decisions about *Scaling*. At that time, we will focus more on designing and building the current application.

The CloudFormation Template used for this exercise prepared us with the shell scripts needed to build **Docker Image** and **Amazon ECR (Elastic Container Registry)**. The exercise will focus on **Secrets Manager** rather than on **Docker**, **Amazon ECS**, or **Amazon ECR**.

Regarding the architectural model, the prepared AWS resources will be linked as follows:
![fargate-architecture](/images/4-fargate-architecture.png?width=40pc)

**Contents**
- [Overview](#overview)
- [Prepare Docker Images for Amazon ECR](#prepare-docker-images-for-amazon-ecr)
- [Configure AWS Fargate](#configure-aws-fargate)
- [Access to Fargate Containers](#access-to-fargate-containers)
- [Access to RDS Database](#access-to-rds-database)
- [Conclusion and a better understanding of the process](#conclusion-and-a-better-understanding-of-the-process)

#### Prepare Docker Images for Amazon ECR

---
**Access to Bastion Host with Session Manager**

1. Access the service [Systems Manager](https://us-east-1.console.aws.amazon.com/systems-manager)
2. In the left-hand navigation bar, select **Session Manager**.

![ssm-session-manager](/images/3-ssm-session-manager.png)

3. Press the `Start Session` button.
4. Select the EC2 instance corresponding to the **BastionIP** value and press the `Start Session` button.

![ssm-session-manager-start-session](/images/3-ssm-session-manager-start-session.png)

5. In the new window bar, we will proceed to run the necessary commands.

{{% notice note %}}
Before running the scripts, we need to make sure the account used is `ec2-user`.
{{% /notice %}}

Proceed to switch to `ec2-user` account with the following command:

```bash
sudo su - ec2-user
```

![ssm-session-manager-new-session-bastion](/images/3-ssm-session-manager-new-session-bastion.png)

---

**Build Docker Image**

We proceed to build **Docker Image** using the shell script `dockerbuild.sh`. This code will proceed to create Docker Image based on `Dockerfile` prepared (by CloudFormation).

```bash
./dockerbuild.sh
```

![shell-docker-build](/images/4-shell-docker-build.png)

{{% notice note %}}
Building the Docker Image will take a few minutes to complete.
{{% /notice %}}

After the process is complete, we check the Docker Image named with `-ecrre-` format with the following command:

```bash
docker images
```

![shell-docker-images](/images/4-shell-docker-images.png)

Once confirmed, we proceed to push this Docker Image to Amazon ECR using the shell script `dockertagandpush.sh`.

```bash
./dockertagandpush.sh
```

![shell-docker-push](/images/4-shell-docker-push.png)

{{% notice note %}}
Pushing the Docker Image to Amazon ECR will take a few minutes to complete.
{{% /notice %}}

1. Go to [Amazon ECR](https://console.aws.amazon.com/ecr/repositories?region=us-east-1)
2. Confirm that the latest Image has been uploaded.

![ecr-repository](/images/4-ecr-repository.png)

Once the process is complete, we proceed to configure **AWS Fargate** based on **Amazon ECR**.

#### Configure AWS Fargate
To configure **AWS Fargate**, we need to configure **Task Definition** first.

1. Access the service [Amazon ECS](https://us-east-1.console.aws.amazon.com/ecs).
2. In the left hand navigation bar, select **Clusters**.

![ecs-cluster](/images/4-ecs-cluster.png)

3. Click on the Cluster named with the following format `-ECSCluster-`.
4. In the left hand navigation bar, select **Task Definitions**, select the Task named with the following format `-TaskDefinition-`.
5. Proceed to create a new revision, click the `Create new revision` button.

![ecs-task-definition](/images/4-ecs-task-definition.png)

6. Click the `Configure via JSON` button.

![ecs-task-definition-configure-json](/images/4-ecs-task-definition-configure-json.png)

7. Scroll to the **secrets** section, replacing `SECRETNAME` with the *Secret* we created in the previous section.

![ecs-task-definition-replace-secret](/images/4-ecs-task-definition-replace-secret.png)

8. Press the `Save` button.
9. Click the `Create` button to proceed with initialization.

![ecs-task-definition-new-revision](/images/4-ecs-task-definition-new-revision.png)

10. Press the `Actions` button, select `Run Task`.

![ecs-task-definition-run-task](/images/4-ecs-task-definition-run-task.png)

11. Proceed to select the following values:
   ![ecs-task-definition-run-task-config](/images/4-ecs-task-definition-run-task-config.png)
   1. `Launch Type`: **Fargate**
   2. `Cluster VPC`: Select a prepared VPC with a CIDR of `10.200.0.0/16`.
   3. `Subnets`: Select all.
   4. `Security Group`: Select a name with the following format `-BastionSG-`.
12. Press the `Run Task` button.
13. Wait for the status from **PROVISIONING**, switch to **PENDING** and finally **RUNNING**.

![ecs-cluster-tasks](/images/4-ecs-cluster-tasks.png)

After the Tasks are in **RUNNING** state, we proceed to access the **Fargate Container**. First of all, we need to make a note of the IP address of a Fargate Container.

![ecs-cluster-tasks-detailed-networking](/images/4-ecs-cluster-tasks-detailed-networking.png)

#### Access to Fargate Containers
From the Bastion Host, proceed to access the Fargate Container. When asked for a password, we use the value **EC2UserPassword**.

```bash
ssh <FARGATE_TASK_PRIVATE_IP>
```

![ssm-session-manager-remote-fargate](/images/4-ssm-session-manager-remote-fargate.png)

#### Access to RDS Database
In this part, we will use the new way through the shell script `mysql.newway.sh`.

1. We execute `mysql.newway.sh` to access the RDS Database.
   ```bash
   ./mysql.newway.sh smdemo
   ```
2. Then execute SQL statements to check the existing data.
   ```sql
   use smdemo;
   show tables;
   select * from bookinfo;
   quit;
   ```
3. The result will be similar to the following.

![shell-mysql-new-fargate](/images/4-shell-mysql-new-fargate.png)

#### Conclusion and a better understanding of the process
After successfully accessing RDS Database from Fargate Container, we need to understand that there are many ways to access **RDS Database**, however, which option is secure and suitable for our application. we are the important thing.

For access from Bastion Host, we see the benefit of using shell script to access **RDS Database**, if we use the old `hard-coded password` method then after perform the **Secret Rotation** process, this method will not work.

**For access from Fargate Container:**

1. First, we use **Secrets Manager ARN** as `Environment Variable` in **Task Definition**, then Fargate Containers can easily know and make access to *Secret * created earlier.
2. On the **Docker Image** side, **Dockerfile** is used to enable the shell script `startprocesses.sh`. This code will pass the `Environment Variable` parameters inside the Fargate Container.
3. Fargate Containers can then use the value obtained from the `Environment Variable` parameter `TASKDEF_SECRET` to perform access to **RDS Database**.

4. The following is the code snippet for `startprocesses.sh`.

```sh
touch /etc/profile.d/ecs.sh
chmod 644 /etc/profile.d/ecs.sh

env | \
  grep "^TASKDEF_" | \
  awk -F= '{printf "export %s=%c%s%c\n", $1, 39, $2, 39 }' \
  >> /etc/profile.d/ecs.sh
```

5. From the Task Definition, we can know the `Envionment Variables' defined by the Fargate Container.

![ecs-task-definitions-container-environment-variable](/images/4-ecs-task-definitions-container-environment-variable.png)

6. From the Fargate Container, we can confirm the values ​​of `Environment Variable` with the following command:

```bash
env|grep TASKDEF_SECRET
```

![shell-grep-environment-variable](/images/4-shell-grep-environment-variable.png)

7. The result will be in a format similar to this:
```bash
TASKDEF_SECRET={
   "username": "<DBUser>",
   "password": "<DBPassword>",
   "engine": "mysql",
   "host": "<RDS_ENDPOINT>",
   "port": 3306,
   "dbname": "<RDS_DATABASE>",
   "dbInstanceIdentifier": "<RDS_ID>"
}
```