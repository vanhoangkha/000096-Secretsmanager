---
title : "Using on RDS"
date: 2024-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

**Using Secrets Manager to interact with RDS**

#### Overview
In this section, we will learn how to use the **AWS Secrets Manager** service to perform periodic password changes for the internal RDS Database.

![rds-phase-architecture](/images/3-rds-phase-architecture.png?featherlight=false&width=60pc)

**Content**
- [Overview](#overview)
- [Secure RDS Credential with Secret Manager](#secure-rds-credential-with-secret-manager)
- [Access to RDS Database](#access-to-rds-database)
- [Secret Rotation](#secret-rotation)
- [Access RDS Database after completing Secret Rotation](#access-rds-database-after-completing-secret-rotation)

#### Secure RDS Credential with Secret Manager
We will proceed to store **RDS Credential** values ​​through the AWS Secret Manager service.

1. Access the service [Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager).

![secrets-manager-console](/images/2/0001.png?featherlight=false&width=90pc)

2. Select `Store a new secret`.

![secrets-manager-console](/images/2/0002.png?featherlight=false&width=90pc)

3. In the *Select secret type* section, select the `Credentials for RDS database` button.
   1. In the `Username` box: enter the value of **DBUser**.
   2. In the `Password` box: enter the value of **DBPassword**.
   3. In the `Select the encryption key` box, select `DefaultEncryptionKey`.
4. In the *Select which RDS database this secret will access* section, select the RDS instance corresponding to the value of **DBInstance**. Then, select the `Next` button.
5. In the *Secret name and description* field enter a name and description. Then, select the `Next` button.

![secrets-manager-console](/images/2/0003.png?featherlight=false&width=90pc)

6. In the *Configure automatic rotation* section, select the `Disable automatic rotation` button. Then, select the `Next` button.

![secrets-manager-console](/images/2/0004.png?featherlight=false&width=90pc)

7. In the *Review* section, select the `Store` button.

![secrets-manager-console](/images/2/0005.png?featherlight=false&width=90pc)

{{% notice note %}}
With **AWS SDK** supporting multiple languages, we can easily integrate Secrets Manager with existing application, for example Java(https://docs.aws.amazon.com/code) -samples/latest/catalog/code-catalog-javav2-example_code-secretsmanager.html) or [Python](https://docs.aws.amazon.com/code-samples/latest/catalog/code-catalog-python- example_code-secretsmanager.html).
{{% /notice %}}

![secrets-manager-console](/images/2/0006.png?featherlight=false&width=90pc)

#### Access to RDS Database
Now, we will use **Bastion Host**, which is allowed access to the RDS Database to execute the prepared *shell scripts*.

---
**Access to Bastion Host with Session Manager**

1. Access the service [Systems Manager](https://us-east-1.console.aws.amazon.com/systems-manager)
2. In the left-hand navigation bar, select **Session Manager**.

![secrets-manager-console](/images/2/0007.png?featherlight=false&width=90pc)

3. Press the `Start Session` button.
4. Select the EC2 instance corresponding to the **BastionIP** value and press the `Start Session` button.

![secrets-manager-console](/images/2/0008.png?featherlight=false&width=90pc)

5. In the new window bar, we will proceed to run the necessary commands.

{{% notice info %}}
Before running the scripts, we need to make sure the account used is `ec2-user`.
{{% /notice %}}

Proceed to switch to the `ec2-user` account with the following command:

```
sudo su - ec2-user
```

![secrets-manager-console](/images/2/0009.png?featherlight=false&width=90pc)

---

**Execute Shell Scripts**

1. Search for prepared *shell scripts*.
2. There will be 2 *shell scripts* prepared.
   1. `mysql.oldway.sh`: this code is used to access the RDS database through the old way - `hard-coded password`.
   2. `mysql.newway.sh`: this code is used to access the RDS database through the new way - `Secrets Manager`.

![secrets-manager-console](/images/2/00010.png?featherlight=false&width=90pc)

For `mysql.oldway.sh`, the content inside the code will be similar to the following, where the following values ​​will be pre-filled.
- PASSWORD
- USER
- ENDPOINT

```sh
#/bin/bash

# mysql.oldway.sh

# This is the old way of accessing a database, with a hard-coded password.
# This script will only work right after the CloudFormation template runs.
# After you store and rotate the secret, you will need to use the
# mysql.newway.sh script.

mysql\
-pPASSWORD\
-u USER \
-P 3306\
-h ENDPOINT
```

1. We execute `mysql.oldway.sh` to access the RDS Database.
   ```bash
   ./mysql.oldway.sh
   ```


![secrets-manager-console](/images/2/00011.png?featherlight=false&width=90pc)

2. Then execute SQL statements to check the existing data.
   ```
   use smdemo;
   show tables;
   select * from bookinfo;
   quit;
   ```

3. The result will be similar to the following.


![secrets-manager-console](/images/2/00012.png?featherlight=false&width=90pc)

For `mysql.newway.sh`, the content inside the code will be similar to the following.
- The *getsecretvalue()* function is used to get the RDS Credential stored at the **Secrets Manager** service in the previous step, and then assign it to the argument `$1`.
- In turn assign the relevant values ​​from the return results of the *getsecretvalue()* function.
- Make connection to RDS database via `mysql` command.

```sh
getsecretvalue() {
  aws secretsmanager get-secret-value --secret-id $1 | \
    jq .SecretString | \
    jq fromjson
}

secret=`getsecretvalue $1`
user=$(echo $secret | jq -r .username)
password=$(echo $secret | jq -r .password)
endpoint=$(echo $secret | jq -r .host)
port=$(echo $secret | jq -r .port)

mysql\
-p$password \
-u $user \
-P $port \
-h $endpoint
```

{{% notice info %}}
The return value of the *getsecretvalue()* function is a **JSON String**.
{{% /notice %}}

The result will be similar to this:
```json
{
  "engine": "mysql",
  "username": "myuser",
  "password": "mypassword",
  "host": "my-database-endpoint.us-east-1.rds.amazonaws.com",
  "dbname": "myDatabase",
  "port": "3306"
}
```

1. We execute `mysql.newway.sh` to access the RDS Database.
   ```
   ./mysql.newway.sh <SECRET_NAME>
   ```
2. Then execute SQL statements to check the existing data.
   ```
   use smdemo;
   show tables;
   select * from bookinfo;
   quit;
   ```
3. The result will be similar to the following.

![secrets-manager-console](/images/2/00015.png?featherlight=false&width=90pc)

#### Secret Rotation
After accessing the RDS Database through both ways, we will proceed with the **Secret Rotation** process. This is a periodic process to comply with security rules.

1. Access the service [Secrets Manager](https://console.aws.amazon.com/secretsmanager).
2. Select the *Secret* that we created in the previous step.
3. Click the `Edit rotation` button.

![secrets-manager-console](/images/2/00016.png?featherlight=false&width=90pc)

4. Select `Enable automatic rotation`.

![secrets-manager-console](/images/2/00017.png?featherlight=false&width=90pc)

5. In the *Select rotation interval* field, set the value to **30 days**.
6. In the *New AWS Lambda function name* field, enter the value **smdemo**.
7. Under *Select which secret will be used to perform the rotation*, select `Use this secret`.
8. Press the `Save` button.
9. There will be a message similar to this after **Secret Rotation** starts.

![secrets-manager-console](/images/2/00018.png?featherlight=false&width=90pc)

10. When **Secret Rotation** is complete, the following message will appear.

![secrets-manager-console](/images/2/00019.png?featherlight=false&width=90pc)

11. To get the new value of *Secret*, press the `Retrieve secret value` button.

{{% notice note %}}
Secrets Manager will use [AWS Severless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/) to install Lambda Function.
{{% /notice %}}

#### Access RDS Database after completing Secret Rotation
After completing the **Key Rotation** process, through the old and new access methods, we in turn access the RDS Database and observe the results.

For `mysql.oldway.sh`, the output will be as follows:

![secrets-manager-console](/images/2/00020.png?featherlight=false&width=90pc)

For `mysql.newway.sh`, the output will be as follows:

![secrets-manager-console](/images/2/00021.png?featherlight=false&width=90pc)

For the old way, because we used `hard-coded password`, when the password is changed, an error will appear and prevent access. For the new way, through **Secrets Manager** service, we can easily access RDS Database before and after **Key Rotation** process.