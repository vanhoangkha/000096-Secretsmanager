---
title : "Sử dụng trên RDS"
date: 2024-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

**Sử dụng Secrets Manager tương tác với RDS**

#### Tổng quan
Ở phần này, chúng ta sẽ tìm hiểu làm sao sử dụng dịch vụ **AWS Secrets Manager** nhằm thực hiện thay đổi password định kỳ cho RDS Database nội bộ.

![rds-phase-architecture](/images/3-rds-phase-architecture.png?featherlight=false&width=60pc)

**Nội dung**
- [Tổng quan](#tổng-quan)
- [Bảo mật RDS Credential với Secret Manager](#bảo-mật-rds-credential-với-secret-manager)
- [Truy cập vào RDS Database](#truy-cập-vào-rds-database)
- [Secret Rotation](#secret-rotation)
- [Truy cập RDS Database sau khi hoàn tất Secret Rotation](#truy-cập-rds-database-sau-khi-hoàn-tất-secret-rotation)

#### Bảo mật RDS Credential với Secret Manager
Chúng ta sẽ tiến hành lưu trữ các giá trị **RDS Credential** thông qua dịch vụ AWS Secret Manager.

1. Truy cập vào dịch vụ [Secrets Manager](https://us-east-1.console.aws.amazon.com/secretsmanager).

![secrets-manager-console](/images/2/0001.png?featherlight=false&width=90pc)

2. Chọn `Store a new secret`.

![secrets-manager-console](/images/2/0002.png?featherlight=false&width=90pc)

3. Ở phần *Select secret type*, chọn nút `Credentials for RDS database`.
   1. Ở ô `Username`: nhập giá trị của **DBUser**.
   2. Ở ô `Password`: nhập giá trị của **DBPassword**.
   3. Ở ô `Select the encryption key`, chọn `DefaultEncryptionKey`.
4. Ở phần *Select which RDS database this secret will access*, chọn RDS instance tương ứng với giá trị của **DBInstance**. Sau đó, chọn nút `Next`.
5. Ở phần *Secret name and description* nhập tên và miêu tả. Sau đó, chọn nút `Next`.

![secrets-manager-console](/images/2/0003.png?featherlight=false&width=90pc)

6. Ở phần *Configure automatic rotation*, chọn nút `Disable automatic rotation`. Sau đó, chọn nút `Next`.

![secrets-manager-console](/images/2/0004.png?featherlight=false&width=90pc)

7. Ở phần *Review*, chọn nút `Store`.

![secrets-manager-console](/images/2/0005.png?featherlight=false&width=90pc)

{{% notice note %}}
Với việc **AWS SDK** hỗ trợ nhiều ngôn ngữ, chúng ta có thể dễ dàng tích hợp Secrets Manager với ứng dụng hiện tại, ví dụ như [Java](https://docs.aws.amazon.com/code-samples/latest/catalog/code-catalog-javav2-example_code-secretsmanager.html) hay [Python](https://docs.aws.amazon.com/code-samples/latest/catalog/code-catalog-python-example_code-secretsmanager.html).
{{% /notice %}}

![secrets-manager-console](/images/2/0006.png?featherlight=false&width=90pc)

#### Truy cập vào RDS Database
Bây giờ, chúng ta sẽ sử dụng **Bastion Host**, là nơi được phép truy cập vào RDS Database để thực thi các *shell scripts* đã được chuẩn bị.

---
**Truy cập vào Bastion Host với Session Manager**

1. Truy cập vào dịch vụ [Systems Manager](https://us-east-1.console.aws.amazon.com/systems-manager)
2. Tại thanh điều hướng bên tay trái, chọn **Session Manager**.

![secrets-manager-console](/images/2/0007.png?featherlight=false&width=90pc)

3. Nhấn nút `Start Session`.
4. Chọn EC2 instance tương ứng với giá trị **BastionIP** và nhấn nút `Start Session`.

![secrets-manager-console](/images/2/0008.png?featherlight=false&width=90pc)

5. Ở thanh cửa sổ mới, chúng ta sẽ tiến hành chạy các lệnh cần thiết.

{{% notice info %}}
Trước khi chạy các scripts, chúng ta cần đảm bảo tài khoản được sử dụng là `ec2-user`.
{{% /notice %}}

Tiến hành chuyển qua tài khoản `ec2-user` bằng câu lệnh sau:

```
sudo su - ec2-user
```

![secrets-manager-console](/images/2/0009.png?featherlight=false&width=90pc)

---
**Thực thi các Shell Scripts**

1. Tìm kiếm các *shell scripts* được chuẩn bị sẵn.
2. Sẽ có 2 *shell scripts* được chuẩn bị sẵn.
   1. `mysql.oldway.sh`: đoạn mã này dùng để truy cập vào RDS database thông qua cách cũ - `hard-coded password`.
   2. `mysql.newway.sh`: đoạn mã này dùng để truy cập vào RDS database thôgn qua cách mới - `Secrets Manager`.

![secrets-manager-console](/images/2/00010.png?featherlight=false&width=90pc)

Đối với `mysql.oldway.sh`, nội dung bên trong đoạn mã sẽ tương tự như sau, trong đó các giá trị sau sẽ được điền sẵn.
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

mysql \
-pPASSWORD \
-u USER \
-P 3306 \
-h ENDPOINT
```

1. Chúng ta tiến hành thực thi `mysql.oldway.sh` để truy cập vào RDS Database.
   ```bash
   ./mysql.oldway.sh
   ```


![secrets-manager-console](/images/2/00011.png?featherlight=false&width=90pc)

2. Sau đó, thực thi các câu lệnh SQL để kiểm tra dữ liệu hiện có.
   ```
   use smdemo;
   show tables;
   select * from bookinfo;
   quit;
   ```

3. Kết quả sẽ tương tự như sau.


![secrets-manager-console](/images/2/00012.png?featherlight=false&width=90pc)

Đối với `mysql.newway.sh`, nội dung bên trong đoạn mã sẽ tương tự như sau.
- Hàm *getsecretvalue()* được dùng để lấy RDS Credential được lưu trữ tại dịch vụ **Secrets Manager** ở bước trước, sau đó gán cho argument `$1`.
- Lần lượt gán các giá trị liên quan từ kết quả trả về của hàm *getsecretvalue()*.
- Thực hiện kết nối tới RDS database thông qua lệnh `mysql`.

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

mysql \
-p$password \
-u $user \
-P $port \
-h $endpoint
```

{{% notice info %}}
Giá trị trả về của hàm *getsecretvalue()* là một **JSON String**.
{{% /notice %}}

Kết quả sẽ tương tự như sau:
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

1. Chúng ta tiến hành thực thi `mysql.newway.sh` để truy cập vào RDS Database.
   ```
   ./mysql.newway.sh <SECRET_NAME>
   ```
2. Sau đó, thực thi các câu lệnh SQL để kiểm tra dữ liệu hiện có.
   ```
   use smdemo;
   show tables;
   select * from bookinfo;
   quit;
   ```
3. Kết quả sẽ tương tự như sau.

![secrets-manager-console](/images/2/00015.png?featherlight=false&width=90pc)

#### Secret Rotation
Sau khi đã truy cập vào RDS Database thông qua cả 2 cách, chúng ta sẽ tiến hành quá trình **Secret Rotation**. Đây là một quá trình định kỳ nhằm tuân thủ các quy tắc bảo mật.

1. Truy cập vào dịch vụ [Secrets Manager](https://console.aws.amazon.com/secretsmanager).
2. Chọn *Secret* mà chúng ta đã tạo ở bước trước.
3. Nhấn nút `Edit rotation`.

![secrets-manager-console](/images/2/00016.png?featherlight=false&width=90pc)

4. Chọn `Enable automatic rotation`.

![secrets-manager-console](/images/2/00017.png?featherlight=false&width=90pc)

5. Ở mục *Select rotation interval*, chọn giá trị là **30 days**.
6. Ở mục *New AWS Lambda function name*, nhập giá trị là **smdemo**.
7. Ở mục *Select which secret will be used to perform the rotation*, chọn `Use this secret`.
8. Nhấn nút `Save`.
9. Sẽ có một tin nhắn tương tự như sau khi quá trình **Secret Rotation** bắt đầu.

![secrets-manager-console](/images/2/00018.png?featherlight=false&width=90pc)

10.  Khi quá trình **Secret Rotation** hoàn tất, tin nhắn như sau sẽ xuất hiện.

![secrets-manager-console](/images/2/00019.png?featherlight=false&width=90pc)

11.  Để lấy được giá trị mới của *Secret*, nhấn nút `Retrieve secret value`.

{{% notice note %}}
Secrets Manager sẽ sử dụng [AWS Severless Application Repository](https://aws.amazon.com/serverless/serverlessrepo/) để cài đặt Lambda Function.
{{% /notice %}}

#### Truy cập RDS Database sau khi hoàn tất Secret Rotation
Sau khi đã hoàn tất quá trình **Key Rotation**, thông qua 2 cách truy cập cũ và mới, chúng ta lần lượt truy cập vào RDS Database và quan sát kết quả.

Đối với `mysql.oldway.sh`, kết quả sẽ như sau:

![secrets-manager-console](/images/2/00020.png?featherlight=false&width=90pc)

Đối với `mysql.newway.sh`, kết quả sẽ như sau:

![secrets-manager-console](/images/2/00021.png?featherlight=false&width=90pc)

Đối với cách cũ, bởi vì chúng ta đã sử dụng `hard-coded password` nên khi password bị thay đổi, lỗi sẽ xuất hiện và ngăn cản tác vụ truy cập. Đối với cách mới, thông qua dịch vụ **Secrets Manager**, chúng ta có thể dễ dàng truy cập vào RDS Database trước và sau quá trình **Key Rotation**.